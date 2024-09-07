---
title: "ARMssembly 0 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# ARMssembly 0 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/160?category=3&difficulty=2&page=4&solved=0)
>Description
>What integer does this program print with arguments `1765227561` and `1830628817`? File: [chall.S](https://mercury.picoctf.net/static/37069d9462289016ea1869ef4c993912/chall.S) Flag format: picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb})

Download the file and make it executable with this code: 
```
$ aarch64-linux-gnu-as -o chall.o chall.S
$ aarch64-linux-gnu-gcc -static -o chall chall.o
```
The commands you provided are for assembling and linking an assembly source file (`chall.S`) for the AArch64 architecture on a Linux system. Let's break down each command:

1. `aarch64-linux-gnu-as -o chall.o chall.S`:
    
    - **`aarch64-linux-gnu-as`**: This is the GNU assembler for the AArch64 architecture.
    - **`-o chall.o`**: This option specifies the output file name, which in this case is `chall.o`.
    - **`chall.S`**: This is the input assembly source file that will be assembled.
    
    This command assembles the `chall.S` file into an object file named `chall.o`.
    
2. `aarch64-linux-gnu-gcc -static -o chall chall.o`:
    
    - **`aarch64-linux-gnu-gcc`**: This is the GNU Compiler Collection for the AArch64 architecture.
    - **`-static`**: This option tells the linker to create a statically linked executable, meaning all required libraries will be included within the final executable.
    - **`-o chall`**: This option specifies the output file name, which in this case is `chall`.
    - **`chall.o`**: This is the input object file that was created by the assembler in the previous step.
    
    This command links the object file `chall.o` to create an executable named `chall`.
    

In summary, these commands first assemble an assembly source file (`chall.S`) into an object file (`chall.o`) and then link that object file into a statically linked executable named `chall` for the AArch64 architecture.
***
Then execute it with given numbers and you will get this:
```
$ ./chall 1765227561 1830628817
Result: 1830628817
```
Convert `Result`to hex and you will get the flag:
**picoCTF{6D1D2DD1}**
