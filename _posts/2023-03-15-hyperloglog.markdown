---
layout: post
title: "Hyperloglog: smart counting"
date: 2023-03-15
tags:
  - Algorithm
  - Probability 
---


If I told you to count how many different different users have visited a high traffic website, how would you do it? It's not trivial, is it? Probably, the first idea that would come to every programmer's mind would be to use some kind of set with the user id in it. That's not bad at all: each access to the hash set is constant. However, having to keep each element in memory doesn't seem very optimal, we'd be wasting a lot of memory… What if we were counting something with a very high cardinality (i.e. IPs)? That could explode easily in memory. 

We can do better.


This is where today's algorithm comes in:

## Hyperloglog, the algorithm.

Hyperloglog is one of the most widely used algorithms for estimating cardinalities on large datasets. The algorithm is capable of estimating cardinalities of $>10^9$ with an accuracy of 2% using only 1.5 KB of memory. 

To see how it works, let's look at this example: 
```
list_of_words = ["soydev", "gigachad", "doge", "stonks"]
```

To calculate the number of elements, we only have to convert each element into a random integer and count the maximum number of leading zeros we find in its binary form:

```
h("soydev") = 10000100
h("gigachad") = 01001001
h("doge") = 01110001
h("stonks") = 11101010
```

If the maximum number of leading zeros observed is $n$, an estimate of the number of distinct elements in the set is $2^n$. So in this case $n=1$, the length of the list would be $2^{n+1}=2^2=4$ (soydev, gigachad, doge and stonks).

### Why does it work?

Imagine a string of length $m$ consisting of {$0$,$1$} with equal probability. What is the probability that a string starts with a $0$? And what about $2$ zeros? $3$? $k$?

The probability is $\frac{1}{2}$, $\frac{1}{4}$, $\frac{1}{8}$, $\frac{1}{2^k}$. This means that if you have found a string starting with $k$ zeros, you have observed (according to the laws of mathematics) approximately about $2^k$ elements. 

For example:

Let's imagine a hash function of $4$ bits (which transforms whatever we want to count) that leads to $16$ possible values with equal probability ($\frac{1}{16}$ probability each): 

```
- 0000
- 0001
- 0011, 0010
- 0100, 0111, 0110, 0101
- 1111, 1110, 1001, 1010, 1100, 1011, 1000
```

It is obvious that half of the binary sequences start with $1$, and each additional 0 reduces the probability by half:

```
- 1/4 starts with 01
- 1/8 starts with 001
- 1/16 starts with 0001
- ...
```

So seeing 001 has $\frac{1}{8}$ probability, which means we had to look at approximately 8 objects until we saw it. This allows us to estimate the number of unique items seen (with a maximum of $k$) with a single number of size $log(k)$ bits (constant memory).
All we need is a good hashing function that evenly distributes the elements between $0$ and $2^{k-1}$.

## Reduce outliers

This idea is not exactly HyperLogLog, it is called **Probabilistic counting** and has been around for many years (1977). HyperLogLog is just a further step of this idea, but with some improvements.

The main problem of the previous method described above is how fragile it is: a random occurrence of a large number with prefix $0$ can ruin everything. The first idea that people came up with was to use different hash functions, count their zeros and average them. This is very clever because it allows us to have a much more robust method. However, the computational effort increases a lot because we need to compute severalhashes for each number (and hashing is somewhat expensive).

To solve this, we have the **LogLog** method. This method uses multiple buckets to decrease the variance: the first bits of the hash are used to determine the bucket, and the remaining bits are used to count the 0s. Note how the buckets cleverly simulate the idea of having different hash functions without any heavy computation. Each bucket records its own count, and then we take the average of all buckets to get the cardinality.

HyperLogLog is just an enhancement of the LogLog method that reduces the error of the estimator. The added improvements are as follows:

- It was observed that outliers decrease the accuracy of the estimator. Therefore, before averaging, the largest values are discarded. Specifically, throwing away 30% of the largest values (this is called SuperLogLog).
- Instead of using the geometric mean, it uses the harmonic mean. 

> As a fun fact, Facebook uses this HyperLogLog to count the number of different people visiting Facebook in a week using a single machine with less than 1MB of memory.


### Sources
- [Approximate counting algorithm](https://en.wikipedia.org/wiki/Approximate_counting_algorithm)
- [Facebook Engineering](https://engineering.fb.com/2018/12/13/data-infrastructure/hyperloglog/)
- [This amazing talk by Tyler Treat](https://speakerdeck.com/tylertreat/probabilistic-algorithms-for-fun-and-pseudorandom-profit)
- [LogLog Paper](https://www.ic.unicamp.br/~celio/peer2peer/math/bitmap-algorithms/durand03loglog.pdf)
