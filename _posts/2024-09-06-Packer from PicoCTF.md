---
title: "Packer from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Packer from PicoCTF
[Page](https://play.picoctf.org/practice/challenge/421?page=1&search=packer&solved=0)

```
giorgi@darkfears31:~/Desktop$ strings out | head
UPX!@
P/PI
}_PzH
F^lx/
Bf/P
@/ j
ME=n
[GSd2
t>d4
8%'
```

> **UPX** is a free, secure, portable, extendable, high-performance **executable packer** for several executable formats.
> **UPX** is an advanced executable file compressor. UPX will typically reduce the file size of programs and DLLs by around 50%-70%, thus reducing disk space, network load times, download times and other distribution and storage costs.


 
```
Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.2       Markus Oberhumer, Laszlo Molnar & John Reiser    Jan 3rd 2024

Usage: upx [-123456789dlthVL] [-qvfk] [-o file] file..

Commands:
  -1     compress faster                   -9    compress better
  -d     decompress                        -l    list compressed file
  -t     test compressed file              -V    display version number
  -h     give more help                    -L    display software license
Options:
  -q     be quiet                          -v    be verbose
  -oFILE write output to 'FILE'
  -f     force compression of suspicious files
  -k     keep backup files
file..   executables to (de)compress

Type 'upx --help' for more detailed help.

UPX comes with ABSOLUTELY NO WARRANTY; for details visit https://upx.github.io
```


```
giorgi@darkfears31:~/Desktop$ upx -d out
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.2       Markus Oberhumer, Laszlo Molnar & John Reiser    Jan 3rd 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
upx: out: NotPackedException: not packed by UPX

Unpacked 1 files.
```

```
giorgi@darkfears31:~/Desktop$ strings out | grep pico  #gave nothing
```

```
giorgi@darkfears31:~/Desktop$ strings out | grep flag
Password correct, please see flag: 7069636f4354467b5539585f556e5034636b314e365f42316e34526933535f33373161613966667d
(mode_flags & PRINTF_FORTIFY) != 0
WARNING: Unsupported flag value(s) of 0x%x in DT_FLAGS_1.
version == NULL || !(flags & DL_LOOKUP_RETURN_NEWEST)
flag.c
_dl_x86_hwcap_flags
_dl_stack_flags
```

Encoded Flag is :
7069636f4354467b5539585f556e5034636b314e365f42316e34526933535f33373161613966667d

In [CyberChef](https://gchq.github.io/CyberChef/) we can decode flag and see that its : 
==picoCTF{U9X_UnP4ck1N6_B1n4Ri3S_371aa9ff}==
