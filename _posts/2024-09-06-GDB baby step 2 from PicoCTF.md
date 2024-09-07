---
title: "GDB baby step 2 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# GDB baby step 2 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/396?page=1&search=gd)
>Description
>Can you figure out what is in the `eax` register at the end of the `main` function? Put your answer in the picoCTF flag format: `picoCTF{n}` where `n` is the contents of the `eax` register in the decimal number base. If the answer was `0x11` your flag would be `picoCTF{17}`.Debug [this](https://artifacts.picoctf.net/c/520/debugger0_b).

>Hints
>You could calculate `eax` yourself, or you could set a breakpoint for after the calculcation and inspect `eax` to let the program do the heavy-lifting for you.

First make file executable
```
chmod +x debugger0_b
```
Then open the file in the `gdb`and then read `info functions`.
use the `asm layout` to see how the program works and where it stops:
![](/assets/images/Screenshot from 2024-07-20 18-33-21.png)
first breakpoint is at `0x40110b` because of that program can't read the `%eax` so from the hints we have to set another breakpoint after the `0x401141` function at the `0x401142 - ret`function:
```
b *0x401142
```
This will set the new breakpoint. Run the program and when it stops click `c` for it to continue till it reaches second breakpoint. After that you can use:
```
print/d $eax
```
With this you will print `eax` in decimal with the help of `/d`and you will get the decimal.
***
The flag is:
**picoCTF{307019}**
