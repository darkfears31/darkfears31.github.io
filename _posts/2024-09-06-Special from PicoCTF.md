---
title: "Special from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Special from PicoCTF
[page](https://play.picoctf.org/practice/challenge/377?category=5&difficulty=2&page=1&search=&solved=0)
>Description
>Don't power users get tired of making spelling mistakes in the shell? Not anymore! Enter Special, the Spell Checked Interface for Affecting Linux. Now, every word is properly spelled and capitalized... automatically and behind-the-scenes! Be the first to test Special in beta, and feel free to tell us all about how Special streamlines every development process that you face. When your co-workers see your amazing shell interface, just tell them: That's Special (TM)Start your instance to see connection details.`ssh -p 56363 ctf-player@saturn.picoctf.net`The password is `d8819d45`

>Hints
>Experiment with different shell syntax

In this SSH environment i can't use normal commands like `nano cd ls` etc. 
I'm not that knowledgeable so i had to search up the answer and i came across this:
```
${parameter=<command>}
```
The syntax `${parameter=ls}` is specific to the **Z Shell (zsh)** and can be a bit misleading if taken out of context. This syntax is used for parameter expansion with assignment, where the `parameter` is assigned the output of the `ls` command if it is not already set.
So this means that `parameter`is not assigned anywhere so it takes `<command>`as an argument and `$` calls it and it shows the answers.
```
Special$ ${parameter=ls}
${parameter=ls} 
blargh
Special$ ${parameter=cd blargh}
${parameter=cd blargh}
${parameter=cat blargh/flag.txt} 
picoCTF{5p311ch3ck_15_7h3_w0r57_0c61d335}
```
Here we go The flag is:
**picoCTF{5p311ch3ck_15_7h3_w0r57_0c61d335}**
