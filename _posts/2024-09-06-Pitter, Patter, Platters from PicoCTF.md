---
title: "Pitter-Patter-Platters from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Pitter-Patter-Platters from PicoCTF
[page](https://play.picoctf.org/practice/challenge/87?category=4&difficulty=2&page=2)
>Description
>'Suspicious' is written all over this disk image. Download [suspicious.dd.sda1](https://jupiter.challenges.picoctf.org/static/47f3cb40aed42fbd74fd644e11d08007/suspicious.dd.sda1)

If you download the file you have to upload it to `autopsy` and then read unallocated files to find the flag but i tried strings: 
```
 $ strings -e b suspicious.dd.sda1| rev
```
This decoded bytes into strings and then reversed it and i got the flag:
**picoCTF{b3_5t111_mL|_<3_ce709d16}**
