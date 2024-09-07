---
title: "Speeds-and-feeds from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Speeds-and-feeds from PicoCTF
[page](https://play.picoctf.org/practice/challenge/116?page=13)

>Description
There is something on my shop network running at `nc mercury.picoctf.net 7032`, but I can't tell what it is. Can you?

Connecting to NetCat gives us big ahh code:
```
nc mercury.picoctf.net 7032
G17 G21 G40 G90 G64 P0.003 F50
G0Z0.1
G0Z0.1
G0X0.8276Y3.8621
G1Z0.1
G1X0.8276Y-1.9310
G0Z0.1
G0X1.1034Y3.8621
G1Z0.1
G1X1.1034Y-1.9310
```
In hint there is question:
>What language does a CNC machine use?

Answer is G-code.
>G-code is **a type of programming language used in the areas of Computer Numerical Control (CNC) and 3D printing for instructing machine tool movement**.

We can search G-code simulators and we find [NCviewer](https://ncviewer.com/) and we can upload file that contains our G-code.
and we get the answer: ![](/assets/images/img.png)
