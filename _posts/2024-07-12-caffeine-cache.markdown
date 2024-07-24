The other day, while perusing a tech post, I stumbled upon S3 FIFO: https://s3fifo.com—a method claiming to outperform LRU (Least Recently Used) in terms of cache miss ratio. Intriguingly, notable companies like RedPanda, Rising Wave, and Cloudflare have already implemented it in various capacities. This piqued my interest. At Datadog, we rely heavily on LRU caches, so I knew I had to put S3 FIFO to the test.

However, diving into a new caching approach without a deep understanding of our current system seemed premature. In my team we extensively use [Caffeine](https://github.com/ben-manes/caffeine) and let's be sincere, I do not know it's internals and I have never actually checked if there were knobs and parameters to fine tune.

 This post chronicles my journey of delving into the intricacies of cache systems. I will explore Caffeine’s inner workings, dissect its code, and run simulations with real data. 

Join me as we unravel the complexities of modern caching strategies, evaluate their performance, and seek to optimize our systems. Whether you're a seasoned engineer or just curious about advanced caching mechanisms, this exploration promises insights and practical takeaways. Let's dive in.

---------------------
# Overview of Caffeine's Architecture
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

3. **Overflow Handling**

This one was kinda surprising to see as it handles a potential overflow when dealing with `System.nanoTime()`. 

```java
if ((previousTimeNanos < 0) && (currentTimeNanos > 0)) {
    previousTimeNanos += Long.MAX_VALUE;
    currentTimeNanos += Long.MAX_VALUE;
}
```

TODO MAKE SURE WE UNDERSTAND THIS!!!!!!!!!!!!!!

Here's what's happening:

1. The condition `(previousTimeNanos < 0) && (currentTimeNanos > 0)` detects if a wraparound has occurred. This happens when the previous time was close to `Long.MAX_VALUE` (thus negative when interpreted as a signed long) and the current time has wrapped around to a small positive number. If a wraparound is detected, both times are increased by `Long.MAX_VALUE`.

In order to understand when this could happen, you need to know that `System.nanoTime()` doesn't start at `0` when your system boots up. Instead, it starts at an arbitrary point, which could be any value, positive or negative. Depending on the initial value when your JVM starts, an overflow could occur much sooner.

Let's see an example. Imagine the system has been running for a very long time, and `System.nanoTime()` is approaching its maximum value 
```
previousTimeNanos = 9223372036854775807 (Long.MAX_VALUE)
```

A little time passes, and we call `advance()` again. Now:
```
currentTimeNanos = -9223372036854775808 (Long.MIN_VALUE)
```
If we didn't handle the overflow, subtracting `previousTimeNanos` from `currentTimeNanos` would give us a huge negative number, incorrectly suggesting that time had gone backwards!

This is where the overflow handling comes in. The code effectively shifts both times into the positive range, maintaining their relative difference. The condition `(previousTimeNanos < 0) && (currentTimeNanos > 0)` detects exactly this situation. It's true when `previousTimeNanos` has become negative (due to being interpreted as a signed long) but `currentTimeNanos` hasn't yet. When this condition is true, we add `Long.MAX_VALUE` to both times:
```
previousTimeNanos = 9223372036854775807 + 9223372036854775807 = -2
currentTimeNanos = -9223372036854775808 + 9223372036854775807 = -1
```
Now, when we calculate the time difference, we get:
```
currentTimeNanos - previousTimeNanos = -1 - (-2) = 1
```
This approach works correctly as long as the time between calls to `advance()` is less than about 292 years which is the total range that can be represented by a long value in nanoseconds (which is a safe assumption for cache operations).


## Read and Write Buffers
Caffeine uses read and write buffers to batch operations and minimize lock contention. This approach allows for efficient reordering of entries without locking on every operation because instead of locking for each operation, operations are buffered and applied in bulk. 

## Eviction Policy: Window TinyLFU
Caffeine uses the Window TinyLFU policy, which combines a recency-biased admission window with a frequency-biased main space, to optimize hit rates.

## Adaptivity
Caffeine dynamically adjusts the size of its admission window and main space based on workload characteristics, using a hill-climbing algorithm to optimize performance.



