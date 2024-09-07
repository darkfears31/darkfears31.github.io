---
title: "GDB baby step 3 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# GDB baby step 3 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/397?page=1&search=gd)
>Description
>Now for something a little different. `0x2262c96b` is loaded into memory in the `main` function. Examine byte-wise the memory that the constant is loaded in by using the GDB command `x/4xb addr`. The flag is the four bytes as they are stored in memory. If you find the bytes `0x11 0x22 0x33 0x44` in the memory location, your flag would be: `picoCTF{0x11223344}`.Debug [this](https://artifacts.picoctf.net/c/531/debugger0_c).

>Hints
>You'll need to breakpoint the instruction after the memory load.
>Use the gdb command `x/4xb addr` with the memory location as the address `addr` to examine.

First make the file executable:
```
chmod +x debugger0_c
```
Then go to `gdb` and examine.
```
(gdb) disas main
Dump of assembler code for function main:
   0x0000000000401106 <+0>:	endbr64
   0x000000000040110a <+4>:	push   %rbp
   0x000000000040110b <+5>:	mov    %rsp,%rbp
   0x000000000040110e <+8>:	mov    %edi,-0x14(%rbp)
   0x0000000000401111 <+11>: mov    %rsi,-0x20(%rbp)
   0x0000000000401115 <+15>: movl   $0x2262c96b,-0x4(%rbp)
   0x000000000040111c <+22>: mov    -0x4(%rbp),%eax
   0x000000000040111f <+25>: pop    %rbp
   0x0000000000401120 <+26>: ret
```
***
**movl $0x2262c96b, -0x4(%rbp)**: Moves the immediate value 0x2262c96b into the memory location at RBP minus 0x4 (4 bytes below the current base pointer). Note the use of `movl`, which indicates moving a 32-bit value.
***
**mov -0x4(%rbp), %eax**: Moves the 32-bit value at the memory location RBP minus 0x4 into the EAX register. This effectively loads the immediate value previously stored at -0x4(%rbp) into EAX.
***
For better look at how file runs lets go to assembly layout:
```
(gdb) layout asm
```
![](/assets/images/Screenshot from 2024-07-20 19-31-10.png)
From hints we have to set breakpoint after the memory load which will be on `0x000000000040111f`.
```
(gdb) b *0x000000000040111f
```
This will set the new breakpoint and type `c`so the debugging will continue. Program will stop on the new breakpoint and we will be able to examine the memory.
![](/assets/images/Screenshot from 2024-07-20 19-36-22.png)
***
And there is the flag:
**picoCTF{0x6bc96222}**
