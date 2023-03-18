---
layout: post
title: "Flipping coins like a pro"
caraer-img: "img/coins.jpeg"
# All dates must be YYYY-MM-DD format!
author: "Adrià"
mathjax: true
date: 2021-04-20
catalog: true
tags:
  - Probability
---

Today I'm going to comment on a simple question I came across the other day on Twitter:
“How many times do you have to flip a coin to get 3 heads in a row?”.

Seems simple right? My first (and wrong) approach was the following: 
With 3 tosses, you have 8 possible outcomes (2^3), and how many of those outcomes contain 3 heads? Only one. So there is no doubt right: 8 throws are needed.

After answering the question inside my head, I went to the Internet and asked Mr. savior ~~Google~~ DuckDuckGo if I was right.

```
Me: How many times do you have to flip a coin to get 3 heads in a row?
DuckDuckGo: 14.
Me: Fuck.
```

## Starting at the bottom
Imagine you have two friends, Sara and Javi. Sara is counting how many times she gets Head and Tails (OX) and Javi is counting Tails and Tails (XX). On average, who is going to get their pattern more times?
If we were to use the incorrect argument above, you could say the same thing: Both are looking for a pattern of 2 throws, ergo 4 possibilities (1/4).

But that is not true: If you are Sara, you have more options to be inside this desired pattern:

 a) You start getting Tails (which makes you start over)
 <br>
 b) You start getting Heads (you are inside the pattern)
  - In the following toss you can get Tail (and you have completed), or you can get Head (and you are still inside the pattern and can start from b)

 However, Javi can get a Head at any time which is not in his desired pattern (Cross Cross) and “have to start over”. The correct answer is that Sara will get the pattern more times because on average she only requires 4 tosses to get OX, while to get XX he needs 6 tosses. 

## The joy of flipping a coin
Now that we know that my mistake was not taking into account the times we have to start from the bottom, we can restate our calculations to get the correct answer.

The probability of getting only one side is $frac{1}{2}$ so we need to flip the coin 2 times. This can be calculated as follows:


<div align="center">
Twice heads = probability of tails * (Twice heads + 1) + probability of heads and tails * (Twice heads + 2) + probability Twice heads * 2
<br><br>
$X_2 = \frac{1}{2} * (X_2 +1) + \frac{1}{4}*(X_2 + 2)$ + $\frac{1}{4}*2$
<br>
$X_2 = 6$
</div>

A general formula for $n$ would be as follows:

<div align="center">
$X_n=(1-p)(X_n +1) + p*(1-p)(X_n +2) + ... + p^{n-1}(1-p)(X_n+n)+ p^n(n)$
<br><br>
$X_n=\frac{(p^{-n}-1)}{1-p}$
</div>

Using the formula we can calculate that it takes 14 pitches to get 3 heads in a row, then 30 pitches to get 4 heads in a row,.... (it grows exponentially).

