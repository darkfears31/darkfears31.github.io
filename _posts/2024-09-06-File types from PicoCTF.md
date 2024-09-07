---
title: "File types from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# File types from PicoCTF
[page](https://play.picoctf.org/practice/challenge/268?page=8)

> #Description
> ContentsThis file was found among some files marked confidential but my pdf reader cannot read it, maybe yours can.You can download the file from [here](https://artifacts.picoctf.net/c/82/Flag.pdf).


```
giorgi@darkfears31:~/Desktop/picoctf$ ls 
Flag.pdf
giorgi@darkfears31:~/Desktop/picoctf$ file Flag.pdf 
Flag.pdf: shell archive text
giorgi@darkfears31:~/Desktop/picoctf$ sh Flag.pdf
x - created lock directory _sh00046.
x - extracting flag (text)
x - removed lock directory _sh00046.
giorgi@darkfears31:~/Desktop/picoctf$ ls
Flag.pdf  flag
giorgi@darkfears31:~/Desktop/picoctf$ file flag
flag: current ar archive
giorgi@darkfears31:~/Desktop/picoctf$ ar x flag
giorgi@darkfears31:~/Desktop/picoctf$ ls
Flag.pdf  flag
giorgi@darkfears31:~/Desktop/picoctf$ file flag
flag: cpio archive; device 234, inode 37435, mode 100644, uid 0, gid 0, modified Thu Mar 16 01:40:19 2023, 510 bytes "flag"
```

```
giorgi@darkfears31:~/Desktop/picoctf$ mkdir cpio && cpio -i -D cpio < flag && cd cpio
2 blocks
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag
flag: bzip2 compressed data, block size = 900k 
```

```
giorgi@darkfears31:~/Desktop/picoctf/cpio$ bzip2 -d flag
bzip2: Can't guess original name for flag -- using flag.out
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag.out
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag.out 
flag.out: gzip compressed data, was "flag", last modified: Thu Mar 16 01:40:19 2023, from Unix, original size modulo 2^32 328
giorgi@darkfears31:~/Desktop/picoctf/cpio$ mv flag.out flag.gz
giorgi@darkfears31:~/Desktop/picoctf/cpio$ gzip -d flag.gz
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag
flag: lzip compressed data, version: 1
giorgi@darkfears31:~/Desktop/picoctf/cpio$ lzip -d flag
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag.out
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag.out 
flag.out: LZ4 compressed data (v1.4+)
giorgi@darkfears31:~/Desktop/picoctf/cpio$ mv flag.out flag.lz4
giorgi@darkfears31:~/Desktop/picoctf/cpio$ lz4 -d flag.lz4 
Decoding file flag 
flag.lz4             : decoded 264 bytes                                       
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag  flag.lz4
```

```
lz4 flag.lz4
Decoding file flag 
flag already exists; do you want to overwrite (y/N) ? y
flag.lz4             : decoded 264 bytes                                       
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag  flag.lz4
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag
flag: LZMA compressed data, non-streamed, size 253
giorgi@darkfears31:~/Desktop/picoctf/cpio$ lzma -d flag
lzma: flag: Filename has an unknown suffix, skipping
giorgi@darkfears31:~/Desktop/picoctf/cpio$ mv flag flag.lzma
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag.lz4  flag.lzma
giorgi@darkfears31:~/Desktop/picoctf/cpio$ lzma -d flag.lzma
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag  flag.lz4
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag
flag: lzop compressed data - version 1.040, LZO1X-1, os: Unix
giorgi@darkfears31:~/Desktop/picoctf/cpio$ mv flag flag.lzop
giorgi@darkfears31:~/Desktop/picoctf/cpio$ lzop -d flag.lzop
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag  flag.lz4  flag.lzop
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag
flag: lzip compressed data, version: 1
giorgi@darkfears31:~/Desktop/picoctf/cpio$ lzip -d flag
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag.lz4  flag.lzop  flag.out
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag.out
flag.out: XZ compressed data, checksum CRC64
giorgi@darkfears31:~/Desktop/picoctf/cpio$ mv flag.out flag.xz
giorgi@darkfears31:~/Desktop/picoctf/cpio$ xz -d flag.xz
giorgi@darkfears31:~/Desktop/picoctf/cpio$ ls
flag  flag.lz4  flag.lzop
giorgi@darkfears31:~/Desktop/picoctf/cpio$ file flag 
flag: ASCII text
giorgi@darkfears31:~/Desktop/picoctf/cpio$ cat flag
7069636f4354467b66316c656e406d335f6d406e3170756c407431306e5f
6630725f3062326375723137795f39353063346665657d0a
```

Encoded Flag is : **7069636f4354467b66316c656e406d335f6d406e3170756c407431306e5f**
**6630725f3062326375723137795f39353063346665657d0a** 
let's go to [cyberchef](https://gchq.github.io/CyberChef/) and decode this HEX text and we get the flag: 
   *picoCTF{f1len@m3_m@n1pul@t10n_f0r_0b2cur17y_950c4fee}*
   
