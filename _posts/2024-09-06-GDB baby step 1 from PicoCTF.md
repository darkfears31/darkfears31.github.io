---
title: "GDB baby step 1 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# GDB baby step 1 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/395?page=1&search=gd)
>Description
>Can you figure out what is in the `eax` register at the end of the `main` function? Put your answer in the picoCTF flag format: `picoCTF{n}` where `n` is the contents of the `eax` register in the decimal number base. If the answer was `0x11` your flag would be `picoCTF{17}`.Disassemble [this](https://artifacts.picoctf.net/c/512/debugger0_a).

>Hints
>`main` is actually a recognized symbol that can be used with gdb commands.

Let's open this file in `gdb` and dissasemble the main to see what is in the `eax`register:
```
$ gdb -q
(gdb) file ./debugger0_a 
Reading symbols from ./debugger0_a...
(No debugging symbols found in ./debugger0_a)
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x0000000000001000  _init
0x0000000000001030  __cxa_finalize@plt
0x0000000000001040  _start
0x0000000000001070  deregister_tm_clones
0x00000000000010a0  register_tm_clones
0x00000000000010e0  __do_global_dtors_aux
0x0000000000001120  frame_dummy
0x0000000000001129  main
0x0000000000001140  __libc_csu_init
0x00000000000011b0  __libc_csu_fini
0x00000000000011b8  _fini
(gdb) disas main
Dump of assembler code for function main:
   0x0000000000001129 <+0>:	endbr64
   0x000000000000112d <+4>:	push   %rbp
   0x000000000000112e <+5>:	mov    %rsp,%rbp
   0x0000000000001131 <+8>:	mov    %edi,-0x4(%rbp)
   0x0000000000001134 <+11>:	mov    %rsi,-0x10(%rbp)
   0x0000000000001138 <+15>:	mov    $0x86342,%eax
   0x000000000000113d <+20>:	pop    %rbp
   0x000000000000113e <+21>:	ret
End of assembler dump.
```
Take the `0x86342` from `$0x86342,%eax`and convert the hex to decimal to get the flag:
**picoCTF{549698}**
