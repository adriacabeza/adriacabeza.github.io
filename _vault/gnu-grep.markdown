---
layout: post
title: "String search algorithm: The secret of GNU grep"
header-img: "img/grep.jpg"
author: "Adrià"
catalog: true
date: 2020-06-22
tags:
  - Algorithm
  - Search algorithm
---

 The other day, while browsing hackernews, I came across a post from the FreeBSD operating system mailing list: ["Why GNU grep is so fast?"](https://lists.freebsd.org/pipermail/freebsd-current/2010-August/019310.html). In the post, the creator of GNU grep explains how grep works and how they made it so fast. This post is about the algorithm he used: the Boyer-Moore algorithm, a really smart string search algorithm.

## The longer the better

The Booyer-Moore algorithm is an algorithm that is the standard reference in the string search literature. It was developed by Robert S. Boyer and J. Strother Moore in 1977 and its main idea is to pre-process the string to be searched in order to skip sections of the text afterwards. For example, let's imagine that we search for the word *"adria "* in a text. Instead of starting at the beginning and checking if the first character is *a*, and the next *d*, and so on; this algorithm starts with the last character (the fifth character). If the fifth character is not inside the word *"adria "*, i.e. *e*, we can skip 5 letters and continue without even checking them. So, on average, **the algorithm runs faster as the string length increases**. Isn't that great?


The preprocessing consists of creating two constant-time lookup tables: the **good suffix rule** table and the **bad character rule** table. These tables will tell us how much we have to shift the pointer(skip characters): the actual offset is the maximum of the offsets calculated by these rules.


### **Bad character's heuristic**

This heuristic is the easiest to understand. Imagine that you iterate a word. For each *i* value less than the length of the sequence, we build a pattern that takes the last *i* characters of the sequence with a non-matching character before them. Then we have to calculate the minimum number of characters we need to shift the pattern to get a match.

Let's see an example to make it clearer (here I was a little bit lazy and copied the example from wikipedia, don't judge me :P):

If we imagine we are searching for the word ANPANMAN we would get the following table:
<div align="center">
<table width="100%" style="border:1px solid black;border-collapse:collapse;">
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;padding:10px;"><b>i</b></td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><b>Pattern</b></td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;padding:10px;"><b>Shift</b></td>
            <td align="center" style="border:1px solid black; border-collapse:collapse;"><b>Explanation</b></td>
    </tr>
    <tr salign="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">0</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><s>N</s></td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">1</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"> The character before the N in ANPANMA<b>N</b> is an A, not an N, so we shift the pattern to the left.</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">1</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><s>A</s>N</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">8</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"> AN is not a subsequence of ANPANMAN so if we find this pattern we can skip the window we are looking at move 8 characters (full length of ANPANMAN).</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">2</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><s>M</s> AN</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">3</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"> The substring <s>M</s>AN coincides with AN<b>PAN</b>MAN which is 3 positions to the left, so we move it 3 positions.</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">3</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><s>N</s>MAN</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">6   </td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"> NMAN is not a subsequence of ANPANMAN but NMAN could be a subsequence 6 positions to the left: ('<b>NMAN</b>PANMAN').</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">4</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><s>A</s>NMAN</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">6</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">Same reason as i = 3</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">5</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><s>P</s>ANMAN</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">6</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">Same reason as i = 3</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">6</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><s>N</s> PANMAN</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">6</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">Same reason as i = 3</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">7</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;"><s>A</s> NPANMAN</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">6</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">Same reason as i = 3</td>
    </tr>
</table>
</div>
<br>

### **The Good sufix rule**

This rule is a bit more complex than the other one but it also exploits the technique of comparing first the end of the pattern towards the start and uses a preprocessing table.

TODO COMPLETE

The other table is not so difficult to calculate: we start from the last character of the seen substring and move to the first character. Each time we move to the left, if the character is not in the table, we add it. Its offset value is the distance to the farthest character on the right. The other characters are given the same value as the total sequence length.

<div align="center">
<table width="100%" style="border:1px solid black;border-collapse:collapse;">
<table>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse; padding:10px;"><b>Character</b></td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;padding:10px;"><b>Shift</b></td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">A</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">1</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">M</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">2</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">N</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">3</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">P</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">5</td>
    </tr>
    <tr align="center" style="border:1px solid black; border-collapse:collapse;">
        <td align="center" style="border:1px solid black; border-collapse:collapse;">Others</td>
        <td align="center" style="border:1px solid black; border-collapse:collapse;">8</td>
    </tr>
</table>

