---
title: "GDB baby step 4 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# GDB baby step 4 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/398?page=1&search=gd)
>Description
>`main` calls a function that multiplies `eax` by a constant. The flag for this challenge is that constant in decimal base. If the constant you find is 0x1000, the flag will be `picoCTF{4096}`.Debug [this](https://artifacts.picoctf.net/c/532/debugger0_d).

First make file executable: `chmod +x debugger0_d`
Go to `gdb` and examine how file works. During examining of the main function we see that it calls `<func1>` and we have to find the number it gives, so we have to follow that `<func1>`. We''ll use:
```
Step-in --> si
Step-over --> ni
```
don't forget to use this:
```
(gdb) layout asm
```
type `run` and then `ni`press `enter`till you reach this:
```
  0x0000000000401142 <+38>:	call   0x401106 <func1>
```
![](/assets/images/Screenshot from 2024-07-20 20-26-57.png)
Then use `Step-in`to follow the `<func1>`:![](/assets/images/Screenshot from 2024-07-20 20-27-53.png)
There is the `imul`containing our hex we are searching for: 0x3269.
```
(gdb) print/d 0x3269
	$1 = 12905
```
And that's the flag:
**picoCTF{12905}**
