---
title: "WebNet0 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# WebNet0 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/32?category=4&difficulty=2&page=3&search=&solved=0)
>Description 
>We found this [packet capture](https://jupiter.challenges.picoctf.org/static/0c84d3636dd088d9fe4efd5d0d869a06/capture.pcap) and [key](https://jupiter.challenges.picoctf.org/static/0c84d3636dd088d9fe4efd5d0d869a06/picopico.key). Recover the flag.

>Hints
>How can you decrypt the TLS stream?

First i asked [ChatGPT](https://chatgpt.com/share/3409d3a5-814d-41a9-a8d9-d77e03d5dedc)and told me that i should use `ssldump` and thats what i did:
```
$ ssldump -r capture.pcap -k picopico.key -d
```
It gave me lot of gibberish so i'm gonna paste one thing where flag is located:
```
    ---------------------------------------------------------------
1 12 0.3575 (0.0004)  S>C  application_data
    ---------------------------------------------------------------
    48 54 54 50 2f 31 2e 31 20 32 30 30 20 4f 4b 0d    HTTP/1.1 200 OK.
    0a 44 61 74 65 3a 20 46 72 69 2c 20 32 33 20 41    .Date: Fri, 23 A
    75 67 20 32 30 31 39 20 31 35 3a 35 36 3a 33 36    ug 2019 15:56:36
    20 47 4d 54 0d 0a 53 65 72 76 65 72 3a 20 41 70     GMT..Server: Ap
    61 63 68 65 2f 32 2e 34 2e 32 39 20 28 55 62 75    ache/2.4.29 (Ubu
    6e 74 75 29 0d 0a 4c 61 73 74 2d 4d 6f 64 69 66    ntu)..Last-Modif
    69 65 64 3a 20 4d 6f 6e 2c 20 31 32 20 41 75 67    ied: Mon, 12 Aug
    20 32 30 31 39 20 31 36 3a 34 37 3a 30 35 20 47     2019 16:47:05 G
    4d 54 0d 0a 45 54 61 67 3a 20 22 36 32 2d 35 38    MT..ETag: "62-58
    66 65 65 34 36 32 62 66 32 32 37 2d 67 7a 69 70    fee462bf227-gzip
    22 0d 0a 41 63 63 65 70 74 2d 52 61 6e 67 65 73    "..Accept-Ranges
    3a 20 62 79 74 65 73 0d 0a 56 61 72 79 3a 20 41    : bytes..Vary: A
    63 63 65 70 74 2d 45 6e 63 6f 64 69 6e 67 0d 0a    ccept-Encoding..
    43 6f 6e 74 65 6e 74 2d 45 6e 63 6f 64 69 6e 67    Content-Encoding
    3a 20 67 7a 69 70 0d 0a 50 69 63 6f 2d 46 6c 61    : gzip..Pico-Fla
    67 3a 20 70 69 63 6f 43 54 46 7b 6e 6f 6e 67 73    g: picoCTF{nongs
    68 69 6d 2e 73 68 72 69 6d 70 2e 63 72 61 63 6b    him.shrimp.crack
    65 72 73 7d 0d 0a 43 6f 6e 74 65 6e 74 2d 4c 65    ers}..Content-Le
    6e 67 74 68 3a 20 31 30 30 0d 0a 4b 65 65 70 2d    ngth: 100..Keep-
    41 6c 69 76 65 3a 20 74 69 6d 65 6f 75 74 3d 35    Alive: timeout=5
    2c 20 6d 61 78 3d 31 30 30 0d 0a 43 6f 6e 6e 65    , max=100..Conne
    63 74 69 6f 6e 3a 20 4b 65 65 70 2d 41 6c 69 76    ction: Keep-Aliv
    65 0d 0a 43 6f 6e 74 65 6e 74 2d 54 79 70 65 3a    e..Content-Type:
    20 74 65 78 74 2f 63 73 73 0d 0a 0d 0a 1f 8b 08     text/css.......
    00 00 00 00 00 00 03 4b ca 4f a9 54 a8 e6 52 50    .......K.O.T..RP
    28 48 4c 49 c9 cc 4b d7 2d c9 2f b0 52 30 2d 4a    (HLI..K.-./.R0-J
    cd b5 e6 aa e5 d2 2b 2e 49 2c 2a 49 2d d2 2d 49    ......+.I,*I-.-I
    cd 2d c8 49 2c 49 45 56 6a a5 60 0c 54 a6 60 a8    .-.I,IEVj.`.T.`.
    07 51 ad a0 50 92 5a 51 a2 9b 98 93 99 9e 67 a5    .Q..P.ZQ......g.
    90 9c 9a 07 d4 07 32 03 00 20 cc fb a0 62 00 00    ......2.. ...b..
    00                                                 .
```
And there is the flag:
**picoCTF{nongshim.shrimp.crackers}**
