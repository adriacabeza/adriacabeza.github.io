---
layout: post
title: "0x5f3759df: The mathematical hack of Quake III Arena"
header-img: "img/games.jpg"
author: "Adrià"
# All dates must be YYYY-MM-DD format!
mathjax: true
date: 2020-08-02
catalog: true
tags:
  - Hack
  - Videogames
---

In this post we are going to explain a super smart hack that calculates the inverse square root: $frac{1}{sqrt(x)}$.

<br> It is especially known for its first use in the first-person shooter Quake III Arena and is very useful in computer graphics for calculating incidence and reflection angles for lighting and shading. It is unclear who invented this method, but some sources credit legendary game programmer John Carmack. For this explanation I'm going to assume that you already know how [floating point number representation](https://www.geeksforgeeks.org/introduction-of-floating-point-representation/) works. 
## The basics: Newton-Raphson Method

The code uses two iterations of this method so I must explain it as well. The Newton-Raphson method is one of the most famous methods for finding roots of real-valued functions. The most basic version starts with the function *f*, its derivative *f'*, and an initial guess *x_o* for a root *f*. We then take an initial guess and iterate it with the following formula:

<div align="center">
$x_{n+1}=x_n - \frac{f(x_n)}{f'(x_n)}$
</div>

The idea is to start with an initial guess that is reasonably close to the true root and then approximate the new point by calculating the intersection of the tangent line of the function with the x-axis. The x-intercept is usually a better approximation to the root of the original function than the first guess. Thus, the method can be iterated. 

<div align="center">
<img class="ui image" alt="formula"  src="https://upload.wikimedia.org/wikipedia/commons/e/e0/NewtonIteration_Ani.gif">
Source: wikipedia
</div>
<br>

Pretty intuitive, isn't it? Moreover, the formula can be easily derived starting from the equation of the tangent line to the curve $y=f(x)$ at $x=x_n$:

<div align="center">
$y=f'(x_n)\cdot(x-x_n) %2B f(x_n)$
</div>


We take the following approximation of $x_{n+1}$ as the x-axis intercept:
<div align="center">
 $0=f'(x_n)\cdot(x_{n+1}-x_{n})+f(x_n)$
</div>

and solving this for $x_{n+1}$ we obtain the formula of the method:

<div align="center">
$x_{n+1}=x_n - \frac{f(x_n)}{f'(x_n)}$
</div>



## The actual code
Below we can see the C code with the original comments of Quake III Arena.
```c
float Q_rsqrt( float number )
{
	long i;
	float x2, y;
	const float threehalfs = 1.5F;

	x2 = number * 0.5F;
	y  = number;
	i  = * ( long * ) &y;                       // evil floating point bit level hacking
	i  = 0x5f3759df - ( i >> 1 );               // what the fuck? 
	y  = * ( float * ) &i;
	y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
//	y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed

	return y;
}
```
It looks pretty weird, I know. Don't go yet! We're not going to explain every part to make it more understandable.

The algorithm accepts a 32-bit floating point number as input and stores a halved value for later use.
```c
x2 = number * 0.5F;
```
It then treats the bits representing the floating-point number as a 32-bit integer.
```c
i  = * ( long * ) &y;
```
It performs a logical right shift of one bit and the result is subtracted from the number 0x5F3759DF. This constant is a floating point representation of an approximation of $\sqrt(2^{127})$. As Chris Lomot explains in his [paper](http://www.lomont.org/papers/2003/InvSqrt.pdf), the magic hexadecimal number, 0x5f3759df, was chosen in such a way that it minimizes the error between the two possible cases: the number having an even exponent or an odd exponent. Thus, whether the bit-shifted number has an even or odd exponent, subtracting the number from the magic number yields an exact approximation of the inverse square root of the original number (once the result, i , has been reinterpreted as a float). The complete proof can be found in his article.
<br>
<br>

The magic value 0x5f3759df is best understood as a floating point number, in binary it is *0.10111110.011011101100111011110111*. The exponent is *10111110*, which is *190* in decimal, representing $2^{(190-127)}$ which is $2^{63}$. The mantissa (after adding the hidden or implicit *1* bit) is *1.011011101100111011110111*, which is 1.4324301481246948482421875 in decimal. Therefore, the magic constant 0x5f3759df is $1.432430148481246948482421875 \times 2^{63}$, which is equivalent to the integer *13211836172961054720*, or about $1.3211...\times 10^{19}$. This is very close to the square root of $2^{127}$, which is approximately $1.3043...\times 10^{19}$.

The reason it is significant is that the exponents in the 32-bit IEEE representation are "excess-127". This, combined with the fact that the floating-point representation "exponent.mantissa" crudely approximates a fixed-point representation of the logarithm of the number (with an added shift), means that you can approximate multiplication and division by simply adding and subtracting the integer form of the floating-point numbers, and take a square root by dividing by two (which is just a shift to the right). This only works when the sign is 0 (i.e., for positive floating point values).

```c
i  = 0x5f3759df - ( i >> 1 );
```
The result is the first approximation of the inverse square root of the input.

Then, again treating the bits as a floating point number, run an iteration of Newton's method, obtaining a more accurate approximation. You can try to obtain the same expression by calculating Newton Raphson's formula of $f(y)=frac{1}{y^2} -x$.


```c
y  = * ( float * ) &i;
y  = y * ( threehalfs - ( x2 * y * y ) );
```

My conclusion in this case is that sometimes code can be strangely beautiful and that the author (or should I say wizard?) probably looks like this:
<div align="center">
<img class="ui image" alt="Big brain meme"  src="https://i.kym-cdn.com/photos/images/newsfeed/001/218/305/85e.jpg">
</div>

### Fuentes
- [Lomont.org](http://www.lomont.org/papers/2003/InvSqrt.pdf)
- [Mrob.com](https://mrob.com/pub/math/numbers-16.html#le009_16)
- [FastInverseSqrt Paper](http://web.archive.org/web/20030426190503/http://www.magic-software.com/Documentation/FastInverseSqrt.pdf)
- [Wikipedia](https://en.wikipedia.org/wiki/Fast_inverse_square_root)