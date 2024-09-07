---
title: "Disk, Disk, sleuth! II from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Disk, Disk, sleuth! II from PicoCTF
[page](https://play.picoctf.org/practice/challenge/137?page=13)
> #Description
>All we know is the file with the flag is named `down-at-the-bottom.txt`... Disk image:     [dds2-alpine.flag.img.gz](https://mercury.picoctf.net/static/544be9762e9f9c0adcbeb7bcf27f49a2/dds2-alpine.flag.img.gz)

It's **.gz** file so we use gunzip to unzip it.
```
gunzip dds2-alpine.flag.img
```
now we are left with **dds2-alpine.flag.img**. It's an **.img** file so i thought maybe there is something hidden in it. 
I used 
```
binwalk -e dds2-alpine.flag.img
```
And i was right. Binwalk extracted file 
```
-dds2-alpine.flag.img.extracted
```
Because we know what file we are searching for it's going to be easy: 
```
$ find . -name "down-at-the-bottom.txt"
./ext-root/root/down-at-the-bottom.txt
```
Now we know where our flag is hidden and head to that directory. The flag is : 
```
   _     _     _     _     _     _     _     _     _     _     _     _     _  
  / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \ 
 ( p ) ( i ) ( c ) ( o ) ( C ) ( T ) ( F ) ( { ) ( f ) ( 0 ) ( r ) ( 3 ) ( n )
  \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/ 
   _     _     _     _     _     _     _     _     _     _     _     _     _  
  / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \ 
 ( s ) ( 1 ) ( c ) ( 4 ) ( t ) ( 0 ) ( r ) ( _ ) ( n ) ( 0 ) ( v ) ( 1 ) ( c )
  \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/ 
   _     _     _     _     _     _     _     _     _     _     _  
  / \   / \   / \   / \   / \   / \   / \   / \   / \   / \   / \ 
 ( 3 ) ( _ ) ( 6 ) ( 9 ) ( a ) ( b ) ( 1 ) ( d ) ( c ) ( 8 ) ( } )
  \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/   \_/ 
```

**picoCTF{f0r3ns1c4t0r_n0v1c3_69ab1dc8}**
