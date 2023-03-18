---
layout: post
title: "Sharing private keys on public media: Diffie-Hellman"
header-img: "img/diffie-helman.jpg"
author: "Adrià"
mathjax: true
date: 2021-02-27
catalog: true
tags:
  - Cryptography
---

Traditionally, secure encrypted communication required keys to be exchanged over some secure physical medium, such as paper key lists and a trusted messenger. However, advances in cryptography have allowed keys to be shared on public media, such as a web page. Today's post is about one of the most common examples of this technology: **Diffie-Hellman**.

Dillie Hellman's method allows two parties who do not know each other previously to establish a shared secret key over an insecure channel.  The explanation of the algorithm is usually initially explained with colors (ref. Wikipedia) as it is very easy to understand:

<div align="center">
<img class="ui image" width="40%" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/46/Diffie-Hellman_Key_Exchange.svg/800px-Diffie-Hellman_Key_Exchange.svg.png">
</div>
<br>

however, we will do it with numbers as it is much closer to what is actually done.

In this example we will use the multiplicative group of integers modulo $p$ where $p$ is a prime number and g is one of the primitive roots of modulo $p$.

## What the f*** is a primitive root?

If you are like me and got a silly face when you read primitive root of $p$, don't worry, it is actually a pretty simple concept to understand. 

An integer $g$ is a primitive root of modulo $n$ if every number coprime to $n$ (numbers that shares no prime factor) is congruent to a power of $g$ $mod (n)$ (congruent means that they have the same remainder when divided by $n$). For example, 3 is a primitive root of 7 ($g$ = 3 and $n$ = 7) because 3 is congruent to $3^1 mod(7)$, $3^2 mod(7)$, $3^3 mod(7)$, $3^4 mod(7)$, $3^5 mod(7)$, $3^6 mod(7)$ and $3^7 mod(7)$:


- $3^0 mod(7) \equiv 1$
- $3^1 mod(7) \equiv 3$
- $3^2 mod(7) \equiv 2$
- $3^3 mod(7) \equiv 6$ 
- $3^4 mod(7) \equiv 4$ 
- $3^5 mod(7) \equiv 5$ 
- $3^6 mod(7) \equiv 1$ 
- $3^7 mod(7) \equiv 1$ (which is equals to $3^0 mod(7)$)



As you can see, within the possible results: $1$,$2$,$3$,$4$,$5$,$6$; we can find all coprime numbers to $7$ (numbers that do not share any prime factor). 

However, it is not a primitive root of $11$:


- $3^0 mod(11) \equiv 1$ 
- $3^1 mod(11) \equiv 3$
- $3^2 mod(11) \equiv 9$ 
- $3^3 mod(11) \equiv 5$ 
- $3^4 mod(11) \equiv 4$ 
- $3^5 mod(11) \equiv 1$ 
- $3^6 mod(11) \equiv 3$
- $3^7 mod(11) \equiv 9$
- and so on...

With modulo 11, you only have the values 1,3,4,5 and 9 (the sequence repeats after the $3^5$) so you can never get other coprimes of 11 such as 2 or 4. 

## Continuation

These values $p$ and $g$ are chosen in a way that the resulting secret can be any number from $1$ to $p-1$. As we did with RSA, let's see an example:

Let's imagine that Laia and Paul want to decide on a shared secret key in order to communicate. 

1. Laia and Paul publicly agree to use modulus p=23 and base g=5.
2. Laia secretly chooses the integer a=4 and sends Paul $A=g^a mod p= 5^4 mod(23) = 4$.
3. Paul secretly chooses the integer b=3 and sends to Laia $B=g^b mod p=5^3 mod(23) = 10$$ 4.
4. Laia computes $s = B ^a mod(p)= 10^4(mod 23) = 18$$ 5.
5. Paul computes $s= A^b mod(p) = 4^3 mod(23) = 18$$ 

The two members have reached the same number $s$ (the secret key) because:

$A^b mod(p) = g^{ab} mod(p) = g^{ba} mod (p) = B^a mod(p)$

The beauty of this whole framework is that neither a nor b has been compromised and that computing $a$ (or $b$) given only $g$, $p$ and $g^a mod p$ is extremely expensive. In this example, since there are only 23 possible outcomes it would be fairly easy to figure out the key, however, if you use numbers for $p$ of about $600$ digits, there is no way to solve it efficiently (a problem known as discrete logarithm). Once both Alice and Paul have calculated the secret key, they can use it as the encryption key. 



