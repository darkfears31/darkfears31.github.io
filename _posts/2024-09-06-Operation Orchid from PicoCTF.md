---
title: "Operation-Orchid from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Operation-Orchid from PicoCTF
[page](https://play.picoctf.org/practice/challenge/285?category=4&difficulty=2&page=1)
>Description
>Download this disk image and find the flag.Note: if you are using the webshell, download and extract the disk image into `/tmp` not your home directory.
>- [Download compressed disk image](https://artifacts.picoctf.net/c/212/disk.flag.img.gz)

First it's a `.gz` file so use `gunzip` and we are left with `disk.flag.img`. After that use sleuth kit:
```
$ mmls disk.flag.img
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000206847   0000204800   Linux (0x83)
003:  000:001   0000206848   0000411647   0000204800   Linux Swap / Solaris x86 (0x82)
004:  000:002   0000411648   0000819199   0000407552   Linux (0x83)
```
Then we have to `mount` it into some file, I chose `tmp` file and named it `something`.
```
$ sudo mount disk.flag.img /tmp/something -o offset=$((411648*512))
```
The offset (`411648*512`) is used to skip a specific number of bytes at the beginning of the disk image file. This is often necessary when dealing with images that contain multiple partitions or when you need to mount a specific partition within the image. The calculation here suggests that the data you want to mount starts at byte `411648 * 512 = 210763776` in the image file.
```
/tmp$ cd something/
/tmp/something$ ls
bin  boot  dev  etc  home  lib  lost+found  media  mnt  opt  proc  root  run  sbin  srv  swap  sys  tmp  usr  var
/tmp/something$ sudo su
[sudo] password for giorgi: 
┌──(root㉿darkfears31)-[/tmp/something]
└─# cd root

┌──(root㉿darkfears31)-[/tmp/something/root]
└─# ls
flag.txt.enc

┌──(root㉿darkfears31)-[/tmp/something/root]
└─# ls -all
total 4
drwx------  2 root root 1024 Oct  6  2021 .
drwxr-xr-x 22 root root 1024 Oct  6  2021 ..
-rw-------  1 root root  202 Oct  6  2021 .ash_history
-rw-r--r--  1 root root   64 Oct  6  2021 flag.txt.enc
┌──(root㉿darkfears31)-[/tmp/something/root]
└─# cat .ash_history 
touch flag.txt
nano flag.txt 
apk get nano
apk --help
apk add nano
nano flag.txt 
openssl
openssl aes256 -salt -in flag.txt -out flag.txt.enc -k unbreakablepassword1234567
shred -u flag.txt
ls -al
halt
```
From here we know that `flag.txt.enc` is encoded with `openSSl aes256`+ salted and the password is `unbreakablepassword1234567`. So we have to reverse that code to decode and get the flag:
```
┌──(root㉿darkfears31)-[/tmp/something/root]
└─# openssl aes256 -d -salt -in flag.txt.enc -out flag.txt -k unbreakablepassword1234567; cat flag.txt
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
bad decrypt
80965819F77F0000:error:1C800064:Provider routines:ossl_cipher_unpadblock:bad decrypt:../providers/implementations/ciphers/ciphercommon_block.c:107:
picoCTF{h4un71ng_p457_0a710765}
```
***
And the flag is: 
**picoCTF{h4un71ng_p457_0a710765}**
