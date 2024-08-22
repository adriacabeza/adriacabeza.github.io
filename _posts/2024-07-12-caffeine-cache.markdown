The other day, while perusing a tech post, I stumbled upon S3 FIFO: https://s3fifo.com—a method claiming to outperform LRU (Least Recently Used) in terms of cache miss ratio. Intriguingly, notable companies like RedPanda, Rising Wave, and Cloudflare have already implemented it in various capacities. This piqued my interest. At Datadog, we rely heavily on LRU caches, so I knew I had to put S3 FIFO to the test.

However, diving into a new caching approach without a deep understanding of our current system seemed premature. In my team we extensively use [Caffeine](https://github.com/ben-manes/caffeine) and let's be sincere, I do not know it's internals and I have never actually checked if there were knobs and parameters to fine tune.

 This post chronicles my journey of delving into the intricacies of cache systems. I will explore Caffeine’s inner workings, dissect its code, and run simulations with real data. 

Join me as we unravel the complexities of modern caching strategies, evaluate their performance, and seek to optimize our systems. Whether you're a seasoned engineer or just curious about advanced caching mechanisms, this exploration promises insights and practical takeaways. Let's dive in.

---------------------
# Overview of Caffeine's implementation 
Caffeine is a high performance, near optimal caching library. It provides awesome features like automatic loading of entries, size-based eviction, statistics, time-based expiration and it is used in a lot of impactful projects like Kafka, Solr, Cassandra, HBase or Neo4j. 

Caffeine's architecture is designed for high performance, leveraging various data structures and algorithms to optimize cache operations. The diagram below gives a high-level overview:

## Order queues
We have two main queues in the cache that ensure a great performance. They are both based on the [AbstractLinkedDeque.java](https://github.com/ben-manes/caffeine/blob/master/caffeine/src/main/java/com/github/benmanes/caffeine/cache/AbstractLinkedDeque.java#L32) which provides an optimised double linked list. These are some of the interesting aspects of the implementation: 

1. **No sentinel nodes**

The class uses a double-linked list without sentinel nodes (dummy nodes at the start and the end of the list). As we can see in a comment in the code:
```
The first and last elements are manipulated instead of a slightly more convenient sentinel element to avoid the insertion of null checks with NullPointerException throws in the byte code. 
```
this is done to reduce null checks. However, both elements are declared as `@Nullable`:
```java
@Nullable E first;
@Nullable E last;
```
so how does it exactly reduce null checks? In a sentinel-based implementation, you always have non-null head and tail nodes. This means you can always safely access `head.next` and `tail.prev` without null checks. However, in this implementation without sentinels, first and last can be null. Shouldn't this require more null checks? The key is in how the JVM handles null checks. When you access a field or method on a potentially null object, the JVM automatically inserts null checks in the bytecode. If the object is null, it throws a NullPointerException. By carefully structuring the code to handle the null cases explicitly, this implementation avoids these automatic null checks and potential NullPointerExceptions in critical paths.

For example, consider the `linkFirst` method:

```java
void linkFirst(final E e) {
  final E f = first;
  first = e;

  if (f == null) {
    last = e;
  } else {
    setPrevious(f, e);
    setNext(e, f);
  }
  modCount++;
}
```
This method handles the case where the list is empty (f == null) separately from the case where it's not. By doing so, it avoids the need for the JVM to insert automatic null checks when accessing fields or methods of f.
In a sentinel-based implementation, you might have code like this:
```java
void linkFirst(final E e) {
  Node newNode = new Node(e);
  newNode.next = head.next;
  newNode.prev = head;
  head.next.prev = newNode;  // Potential automatic null check
  head.next = newNode;       // Potential automatic null check
}
```
Here, the JVM might insert automatic null checks for head.next, even though we know it's never null.

2. **Structural modification tracking**

The class maintains an integer `modCount` to track structural modifications, which is used to detect concurrent modifications during iteration. It is incremented every time an element is added or removed and its primary purpose is to support fail-fast behaviours in iterators:
-  When an iterator is created, it captures the current modCount:
```java
AbstractLinkedIterator(@Nullable E start) {
  expectedModCount = modCount;
  cursor = start;
}
```
- Every time the iterator perform an operation, it checks if the modCount has changed. If it has changed, it means the list was modified outside of the iterator so it throws an exception: 
```java
void checkForComodification() {
  if (modCount != expectedModCount) {
    throw new ConcurrentModificationException();
  }
}
```
The modCount is incremented in methods that structurally modify the list, such as `linkFirst()`, `linkLast()`, `unlinkFirst()`, `unlinkLast()`, and `unlink()`.


### Access Order Queue: AccessOrderDeque.java
The Access Order Queue keeps track of all the entries that are in the hash table based on how recently they were accessed. It is a doubly linked list that maintains the order of cache entries based on access frequency. When an entry is accessed, it is moved to the tail of the list, ensuring that the least recently used (LRU) entry is always at the head. This is vital for implementing the Least Recently Used (LRU) eviction policy. For example, if we have a cache with entries [A <-> B <-> C] where A is the least recently accessed and C is the most recently accessed. If we access B, it will move it to the end of the queue keeping it like this: [A <-> C <-> B]

### Write Order Queue: WriteOrderDeque.java
Similar to the Access Order Queue, the Write Order Queue orders entries based on their creation or update time. It keeps track of the entries based on their write times. This is particularly useful when we want to expire entries after a certain duration since their last write (expireAfterWrite).

For example, consider a scenario where [A <-> B <-> C] are written in that order. If we update A, it will move it to the end of the queue: [B <-> C <-> A]


## Hierarchical TimerWheel
A timer wheel is data structure used to manage time-based events efficiently. The basic idea is that it stores timer events in buckets on a circular buffer, each bucket representing a specific time span (like seconds or minutes). 
In the case of Caffeine, the entries are added to these buckets based on their expiration times, allowing efficient addition, removal and expiration in O(1) time. Given that the circular buffer size is limited, we would have problems when an event needs to be scheduled for a moment in future larger than the size of the ring. That is why we use a hierarchical timer wheel which simply layers multiple timer wheels with different resolutions. If you want to know more about it, it is beautifully explained in this [blogpost](https://www.snellman.net/blog/archive/2016-07-27-ratas-hierarchical-timer-wheel/).   

Let's take a brief look at the code to make sure we understand how it works:

1. **Hierarchical Structure: Buckets and spans**

Each element in the BUCKETS array represents the number of buckets in a timer wheel level, while SPANS defines the duration each bucket covers. As mentioned earlier, the hierarchical structure allows events to cascade from broader to finer time spans. These are the values that were chosen for the Caffeine implementation:

```java
static final int[] BUCKETS = { 64, 64, 32, 4, 1 };
static final long[] SPANS = {
    ceilingPowerOfTwo(TimeUnit.SECONDS.toNanos(1)), // 1.07s
    ceilingPowerOfTwo(TimeUnit.MINUTES.toNanos(1)), // 1.14m
    ceilingPowerOfTwo(TimeUnit.HOURS.toNanos(1)),   // 1.22h
    ceilingPowerOfTwo(TimeUnit.DAYS.toNanos(1)),    // 1.63d
    BUCKETS[3] * ceilingPowerOfTwo(TimeUnit.DAYS.toNanos(1)), // 6.5d
    BUCKETS[3] * ceilingPowerOfTwo(TimeUnit.DAYS.toNanos(1)), // 6.5d
};
```

2. **Clever Bit Manipulation**
The implementation uses bit manipulation techniques to efficiently calculate bucket indices:
```java
long ticks = (time >>> SHIFT[i]);
int index = (int) (ticks & (wheel[i].length - 1));
```
This avoids expensive modulo operations. Here is a breakdown of how it works:

```java
static final long[] SPANS = {
    ceilingPowerOfTwo(TimeUnit.SECONDS.toNanos(1)), // 1.07s
    ceilingPowerOfTwo(TimeUnit.MINUTES.toNanos(1)), // 1.14m
    // ...
};

static final long[] SHIFT = {
    Long.numberOfTrailingZeros(SPANS[0]),
    Long.numberOfTrailingZeros(SPANS[1]),
    // ...
};
```
Each SPAN value is rounded up to the nearest power of 2 and the SHIFT array stores the number of trailing zeros for each SPAN value, which is equivalent to $log_2{span[i]}$. It represents the duration of one tick for that wheel. Then, in order to calculate the bucket index we can do some bit manipulation for a very quick calculation of which bucket an event belogs in:

```java
long ticks = (time >>> SHIFT[i]);
int index = (int) (ticks & (wheel[i].length - 1));
```

- `time >> SHIFT[i]`: this right shifts the appropiate amount for the current wheel, its equivalent to dividing by `SPANS[i]`
- `wheel[i].length - 1`: this is a bitmask. Since wheel[i].length is always a power of 2, this creates a mask of all 1s in the lower bits.
- `ticks & (wheel[i].length - 1)`: this performs a bitwise AND with the mask, which is equivalent to `ticks % wheel[i].length` but again, much faster.

Let's look at an example. Suppose we have:
```
time = 1,500,000,000 nanoseconds (1.5 seconds)
SPANS[i] = 1,073,741,824 (2^30, about 1.07 seconds)
SHIFT[i] = 30
```

In binary:
```
time = 1011001010000000000000000000000
```
Now, when we do `time >>> SHIFT[i]`, we're shifting right by `30` bits:
```
1011001010000000000000000000000 >>> 30 = 1
```
This result, `1`, means means that 1 full tick of this wheel has elapsed. For the next part, let's assume that `wheel[i].length` is `64`. In binary:
```
ticks = 00000000000000000000000000000001
wheel[i].length - 1 = 00000000000000000000000000111111
```

we perform the bitwise AND:
```
  00000000000000000000000000000001
& 00000000000000000000000000111111
  --------------------------------
  00000000000000000000000000000001
```
and the result is 1, so our index is 1. What this means in practice is that the time (1.5 seconds) has causes the wheel to tick once and this tick places the event in the second bucket of the wheel. The beauty of this approach is that it wraps around automatically. If we had $64$ ticks, the index would be $0$ again, as $64 & 63 = 0$. Suppose the next time value is 3.000.000.000 nanoseconds (3 seconds):
```
3,000,000,000 >>> 30 = 2 (ticks)
2 & 63 = 2 (index)
```
So this event would go into the third bucket (index 2) of the wheel.


## Read and Write Buffers
Caffeine uses read and write buffers to batch operations and minimize lock contention. This approach allows for efficient reordering of entries without locking on every operation because instead of locking for each operation, operations are buffered and applied in bulk. 

## Eviction Policy: Window TinyLFU
Caching is all about maximizing the hit ratio - that is, ensuring the most frequently used data is retained in the cache. The eviction policy is the algorithm that decides which entries to keep and which to discard when the cache is full.

The traditional Least Recently Used (LRU) policy is a good starting point, as it's simple and performs well in many workloads. But modern caches can do better by considering both recency and frequency of access.

Recency captures the likelihood that a recently accessed item will be accessed again soon. Frequency captures the likelihood that an item accessed frequently will continue to be accessed frequently.

Caffeine uses a policy called Window TinyLFU to combine these two signals. It works like this:

1. **Admission Window**: When a new entry is added, it goes through an "admission window" before being fully admitted to the cache. This gives the entry a chance to build up its popularity before being included.
2. **Frequency Sketch**: Caffeine uses a compact data structure called a CountMinSketch to track the frequency of access for cache entries. This allows it to efficiently estimate the access frequency of the items.
3. **Eviction**: When the cache is full and a new entry needs to be added, Caffeine checks the frequency sketch. It will only admit the new entry if its estimated frequency is higher than the entry that would need to be evicted to make room.
4. **Aging**: To keep the cache history fresh, Caffeine periodically "ages" the frequency sketch by halving all the counters. This ensures the cache adapts to changing access patterns over time.
5. **Segmented LRU**: For long-term retention, Caffeine uses a Segmented LRU policy. Entries start in a "probationary" segment, and on subsequent access are promoted to a "protected" segment. When the protected segment is full, entries are evicted back to the probationary segment, where they may eventually be discarded.

### Frequency Sketch

As mentioned, the FrequencySketch class is a key component in the cache's eviction policy as it provides an efficient way to estimate the popularity (frequency of access) of cache entries.  The implementation can be found in the `FrequencySketch.java` file and is implemented as a 4-bit CountMinSketch. 

**1. Data Structure**
The sketch itself is represented as a single-dimensional array of 64 bit long values (`table`). Each long value holds 16 4 bit counters, corresponding to 16 different hash buckets. This layout is chosen to improve efficiency as it keeps the counters for a single entry within a single cache line. 

Note that the length of the `table` array is set to the closest power of two greater than or equal to the maximum size of the cahce, to enable efficient bit masking operations. 


**2. Hashing**
The sketch uses two hashing functions `spread()` and `rehash()` to apply supplemental hash functions to the normal element's hash code. 

```java
...
    int blockHash = spread(e.hashCode());
    int counterHash = rehash(blockHash);
...

 /** Applies a supplemental hash function to defend against a poor quality hash. */
  static int spread(int x) {
    x ^= x >>> 17;
    x *= 0xed5ad4bb;
    x ^= x >>> 11;
    x *= 0xac4c1b51;
    x ^= x >>> 15;
    return x;
  }

  /** Applies another round of hashing for additional randomization. */
  static int rehash(int x) {
    x *= 0x31848bab;
    x ^= x >>> 14;
    return x;
  }
```

3. **Frequency Retrieval**

The frequency retrieval happens in the method `frequency()` where it takes the minimum of the 4 relevant counters as a good approximation:
```java
  @NonNegative
  public int frequency(E e) {
    if (isNotInitialized()) {
      return 0;
    }

    int[] count = new int[4];
    int blockHash = spread(e.hashCode());
    int counterHash = rehash(blockHash);
    int block = (blockHash & blockMask) << 3;
    for (int i = 0; i < 4; i++) {
      int h = counterHash >>> (i << 3);
      int index = (h >>> 1) & 15;
      int offset = h & 1;
      count[i] = (int) ((table[block + offset + (i << 1)] >>> (index << 2)) & 0xfL);
    }
    return Math.min(Math.min(count[0], count[1]), Math.min(count[2], count[3]));
  }
```

Note that there is some clever bit manipulation happening here. The same happens with the method `increment` which increments the popularity of an element. Here it is a breakdown of the key steps: 


1. `blockHash = spread(e.hashCode())`: This spreads the hash code of the input element e to get a better distribution of the hash values.
2. `counterHash = rehash(blockHash)`: This further rehashes the blockHash to get a different hash value, which will be used to index into the 16 different hash buckets.
3. `int block = (blockHash & blockMask) << 3`: 
To understand this part, we first need to check `blockMask` and how it is created. It is calculated as `(table.length >> 3 ) - 1`. The reason why we right-shift the table length by 3 bits is because it is equivalent to divide by `8`. Given that each block in the `table` array contains `16` counters, and each counter is `4` bits wide, the total size of each block is `16*4=64 bits (8 bytes)`. This means that by right-shifting by 3, we are effectively getting the number of blocks in the table array.  We then substract 1 to have all the possible values. For example, if the table length is 256, `table.length >> 3` would give us `32`; we substract one so it gives us `31`, in binary `11111`. 

Thus, by masking the `blockHash` with the `blockMask`, we ensure that the resulting blocking index is always within the range of the `table` array. 

4. Then for each iteration (0 to 3), we compute the 4 counter indices: 
  - `int h = counterHash >>> (i << 3)`: This extracts a 8-bit value from the counterHash by right-shifting it by `i * 8 bits`. This gives us the hash value for the current 4-bit counter.
  - `int index = (h >>> 1) & 15`: We first perform a logical right-shift of `h` by 1 (aka divide by 2) to take the least significant bit. The reason why we do this is to use it later for the offset calculation. We then mask it with `15` (`1111` in binary) to get the 4 least significant bits. This gives us the counter index within the block (as there are `16` counters). 
  - `int offset = h & 1`: This line calculates the offset within the `64-bit` block which is either 0 or 1. It does it by taking the least significant bit of the 8-bit hash value.
  - `count[i] = (int) ((table[block + offset + (i << 1)] >>> (index << 2)) & 0xfL)`: This retrieves the 4-bit counter value from the table by:

Computing the index in the table array where the 4 counters for the given element are located: `block + offset + (i << 1)`:
  - `block` is the starting index of the block in the `table` array
  - `offset` is either 0 or 1, depending on which of the two 8-byte segments within the block we are accessing.
  - `(i << 1)` is the offset within the 16-byte segment that contains the 4 counters. Recall that the counters are stored in 16 bytes segment. Given that the variable `i` can be 0,1,2 or 3. Shifting `i` left by 1 is equivalent to multiply it by 2 which gives us the offset in (bytes) of the counter. For example, for `i=2` that would give us `4` which is the offset of the third counter. 



  Then it applies a bitmask to the extracted counter value to ensure that it is a 4-bit unsigned integer `& 0xfL` (`1111` in binary). 
  

5. Finally, the method returns the minimum value among the 4 frequency counts stored in the count array.

4. **Aging**
Periodically, when the number of observed events reaches a certain threshold (`sampleSize`) the `reset()` method is called. This method halves the value of all counters and substract the number of odd counters. 


## Expiration Policy

Expiration is often implemented as variable per entry and expired entries are evicted lazily due to a capacity constraint. This pollutes the cache with dead items, so sometimes a scavenger thread is used to periodically sweep the cache and reclaim free space. This strategy tends to work better than ordering entries by their expiration time on a O(log n) priority queue due to hiding the cost from the user isntead of incurring a penalty on every read or write operation. 



## Concurrency
The traditional solution to access a cache is to guard it with a single lock. This might then be improved through lock stripping by splitting the cache into many smaller independent regions. Unfortunately that tends to have a limited benefit due to hot entries causing some locks to be more contented than others. When contention becomes a bottleneck, the next classic step has been to update only per entry metadata and use either a random sampling or a FIFO based eviction policy. Those techniques can have great read performance, poor write performance and difficulty in choosing a good victim. 

An alternative is to borrow an idea from database theory where writes are scaled by using a commit log. Instead of mutating the data structures immediately, the updates are written to a log and replayes in asynchronous batches. This same idea can be applied to a cache by performing the hash table operation, recording the operation to a buffer and scheduling the replay activity against the policy when necessary. The policy is still gaurded by a lock, or a try lock, but shifts contention onto appending to the log buffers instead. 

In Caffeine, separate buffers are used for cache reads and writes. An access is recorded into a striped ring bugger where the stripe is chosen by a thread specific hash and the number of stripes grows when contention is detected. When a ring buffer is full an asynchronous frain is scheduled and subsqeuent additions to that buffer are discarded  until space becomes available. 


## Adaptative Cache Policy
Caffeine dynamically adjusts the size of its admission window and main space based on workload characteristics, using a hill-climbing algorithm to optimize performance.

Hill Climbing is a simple optimization technicque for searching a local maximum of a function. In our context, we first change the configuration in a certain direction e.g. enlarge the window cache size. Then we compare the hit ratio obtained under the new configuration to the previously recorded hit ratio. If the hit ratio has improved we make an additional step in the same direction. Otherwise, we flip diraction and make a step backward. 

The difficulty in realizing this method is determining how large each steps should be and how frequently to take such a step. Measuring the hit ratio over a short duration if a noisy process. With frequent steps, it is difficult to distinguish between a change in the hit ratio that was caused by the new config and noise. 

In Caffeine, they ended up choosing to do infrequent and relatively large changes. Steps of 5% of the window cache size or +-1  to the increment size:

- 21 possible configurations when adapting the window size (0%, 1%, 5%, 10%, etc)
- 15 possible configurations when adapting the sketch parameters (maximual value of counters sketch is 15). 

Decision internval of once every 10 times the cache size (empirically decided).

