
layout: post
title: "How to Beat Your Kids at Their Own Game"
header-img: "img/toy.jpg"
author: "Adrià"
# All dates must be YYYY-MM-DD format!
date: 2020-05-18
mathjax: true
catalog: true
tags:
  - Mathematics
  - Probability
---

Today I wanted to talk about the scientific notes with (in my opinion) the funniest name ever: <b>How Often Should You Beat Your Kids?</b> by *Don Zagier*. Do not worry, it is not what it seems. Even though the paper states facts like the following:

> A result is proved which shows, roughly speaking, that one should beat one's kids every day except Sunday.

it is a joyful text with very cool mathematical tricks.

However, the paper itself is a continuation of another note: <b>How to Beat Your Kids at Their Own Game</b> by *K. Levasseur* (another beaufitul name for a research text). So I guess that first I have to explain this one... So, no rushes... I'll catch up with Zagier's work another day. Jyi, this is their final conclusion to the quesion <b>How Often Should You Beat Your Kids?</b>:



In <b>How to Beat Your Kids at Their Own Game</b>, the author proposes the following card game:

1. It starts with a deck consisting of $n$ red cards and $n$ black cards.
2. The cards are turned up one at a time.
3. The player has to predict the color of the card which is about to appear.
4. The game ends when the deck is empty. Your score is the number of correct guesses and wins who gets the best score.

Your opponent (the kid) is supposed to guess "Red" or "Black" randomly with equal probability, while you play in an optimal strategy (guessing randomly) whenever equal number of cards of both colors remain in the deck and otherwise predicting which is currently in the majority. The kid would have an average score of exactly $n$. Meanwhile, your average score will be $n + \frac{\sqrt(\pi n - 1)}{2} + O(n^{\frac{1}{2}})$ (do not worry we will go over the details of why this is the average score).

## How to compute average score using card counting?

Each game of "Red or black" can be viewed as decreasing path through lattice points starting at $(n,n)$ and ending at (0,0). At each point in the path, the first coordinate represents the number of red cards left in the deck and the second coordinate the number of black cards. Each step of the path corresponds to the removal of one card from the deck. Thus, exactly one of the coordinates of the point decreases just one step. A <b>game path</b> is any of those paths.

Notice that our expected score grows proportionally with the number of times that cards counting gives no information (when we are on the diagonal red=black). Suppose that we started at $(n, n)$ and that the first card in the deck is black. We can assume that this first card contributes $\frac{1}{2}$ to our expected score (because we have the same probability to get a black or a red card).

<div align="center">
<img src="https://imgur.com/p3KFOyV.png" width="80%" alt="Example of a game path">
</div>


Now assume that our game path meets the diagonal at (n-k, n-k). In the course of traversing this path we would continue to guess that the next card is red since red > black in that portion of the path. We already know that during that path, in the deck there was a black card less, therefore, we will be correct exactly $k$ times and incorrect $k-1$ times during this part of the game. In the path from $(n,n)$ to $(n-k, n-k)$, our expected score is then $k + 0.5$ of a possible $2k$. A child's strategy would yield an average of $k$. Clearly, the more we visit the diagonal, the more frequently we will gain this extra $0.5$ in our expected score.

Example: Imagine a game path randomly chosen (it can be seen in the figure) from a deck of $52$ cards ($26$ black cards and $26$ red cards). The game path visits the diagonal 5 times (without counting $(0,0)$). If we assume that at each of those times, our guess is made at random, we would have an expected score of $28.5$ ($26 + (0.5)\times 5$).


Let $P$ be the set of all game paths. Since a game path is determined by the $n$ positions of the red cards in a $2n$ card deck, there are $C(2n,n)$ (being $C$ the combinatoria formula $C(n,x)=\frac{n !}{x !(n-x) !})$ different game paths. Let $p$ be any game path; and let $v(p)$ be the number of diagonal visits by $p$. Given $v(p)$, the expected score is $n + 0.5(v(p)-1)$. We consider visiting $(0,0)$ a diagonal visit; hence the $-1$ in this formula. Since we assume a deck of cards randomly shuffled, each of the $C(2n, n)$ paths is equally likely: 
<div align="center">
$\begin{aligned}
S(n) &=\sum_{p \in P}(n+0.5(v(p)-1)) / C(2 n, n) \end{aligned}$ <br>
$\begin{aligned}
&=n+0.5\left(\left(\sum_{p \in P} v(p) / C(2 n, n)\right)-1\right)
\end{aligned}$
</div>
<br>
This reduces our problem to the calculation of the average number of visits to the diagonal; in other words, to the calculation of the total number of diagonal visits in all game paths. To calculate the total number of times that we visit the diagonal by all game paths, $V(n)$, we define first $\chi(p, m)$ to be $1$ if $p$ visits $(m,m)$ and $0$ otherwise.

<div align="center">
$ V(n) =\sum_{p \in P}\left[\sum_{m=0}^{n} \chi(p, m)\right]=\sum_{m=0}^{n}\left[\sum_{p \in P} \chi(p, m)\right]$ <br>
$=\sum_{m=0}^{n}$ the number of paths that visit $(m, m)$
</div>
<br>

To carry on with the proof, we do the following reasoning: If $p$ visits $(m,m)$, then the first $2(n-m)$ cards in the deck are $n-m$ red cards and $n-m$ black cards. Those cards can be rearranged in $C(2(n-m),n-m)$ ways. Also, the other $2m$ cards remaining can be rearranged in $C(2m,m)$ ways. So the total number of game paths that visit $(m,m)$ is 

<div align="center">
$C(2m,m) \cdot C(2(n-m), n-m)$.
</div>

<br>
Therefore
<br>
<div align="center">
$V(n) = \sum_{m=0}^{n}$ the number of paths that visit $(m, m) = \sum_{m=0}^{n} C(2 m, m) C(2(n-m), n-m)$
</div>

<br>
<div align="center">
<img src="/img/beat_1.png" width="80%" alt="Visual Proof">
</div>


Once all the information at this point is clear, we can start with the sequences and generating functions to continue understanding which is the total number of times we visit the diagonal for all game paths.

We need to know that if A is a sequence of numbers, the generating function of A is $G(A;z) = A(0)+A(1)z+A(2)z^2+A(3)z^3+...=\sum_{n=0}^\infty A(n)z^n$

In our case, the A function that we will use is $A(n)=ar^n$. Hence, its associated generating function is $G(A;z) = a/(1-rz)$ because of the properties of geometric series.

#### Reminder of geometric series

For $r\neq 1$, the sum of the first n terms corresponds to 

<div align="center">$a+ar+ar^2+...+ar^{n-1} = \sum_{k=0}^{n-1}ar^k = a\left( \frac{1-r^n}{1-r}\right)$</div>

In the last two steps, the procedure followed is the next one:

<div align="center">
$s =a+a r+a r^{2}+a r^{3}+\cdots+a r^{n-1}$ <br>
$r s =a r+a r^{2}+a r^{3}+\cdots+a r^{n-1}+a r^{n}$ <br>
$s-r s =a-a r^{n}$ <br>
$s(1-r) =a\left(1-r^{n}\right)$ <br> 
$s =a\left(\frac{1-r^{n}}{1-r}\right) \quad(\text { if } r \neq 1)$ <br>
</div>

The case that involves us is when n goes to infinity and r must follow $\mid r \mid<1$ for the serie to converge. Once we establish this conditions, the result of the sum is

<div align="center">$\sum_{k=0}^{\infty}ar^k = \frac{a}{1-r}$</div>

because $1-r^n=1$ when $\mid r \mid<1$ and $n\rightarrow \infty$.


Following the theory of generating functions, we will also need that if A and B are sequences of numbers, the product of $A\cdot B$ corresponds to 

<div align="center">$(A\cdot B)(n) = \sum_{m=0}^n A(m)B(n-m)$</div>

And following this, we can also assume that

<div align="center">$G(A\cdot B;z) = G(A;z)G(B;z)$</div>

## Taylor series magics

Now that all the theory is done we can follow with this statement: Let V be the total number of diagional visits by all game paths in P. Then

<div align="center">$G(V;z) = 1/(1-4z) \Rightarrow V(n) = 4^n$</div>

To understand it, we need to know that the Taylor series expansion of $1/\sqrt{1-4z}$  is  $\sum_{n=0}^{\infty}C(2n,n)z^n$ (proof for the reader :P).

Let $D(n) = C(2n,n)$ and using the theory above, $V(n) = (D\cdot D)(n)$ so $V = D\cdot D$. And then,

<div align="center">$G(V;z) = G(D\cdot D; z) = G(D;z)^2 = 1/(1-4z)$</div>

In this case, the only sequence with this generating function is the geometric sequence $4^n$ therefore, $V(n) = 4^n$.

## Wrapping up



### Sources

- [How to Beat Your Kids at Their Own Game](https://www.jstor.org/stable/2689550)
- [How Often Should You Beat Your Kids?](https://people.mpim-bonn.mpg.de/zagier/files/math-mag/63-2/fulltext.pdf)
- [Fermat library](https://fermatslibrary.com/s/how-often-should-you-beat-your-kids)