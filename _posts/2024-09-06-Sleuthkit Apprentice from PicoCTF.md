---
title: "Sleuthkit Apprentice from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Sleuthkit Apprentice from PicoCTF 
[page](https://play.picoctf.org/practice/challenge/300?category=4&difficulty=2&page=1&search=)
>Description
>Download this disk image and find the flag.Note: if you are using the webshell, download and extract the disk image into `/tmp` not your home directory.   
>- [Download compressed disk image](https://artifacts.picoctf.net/c/137/disk.flag.img.gz)

So from the title we have to use [sleuthkit](https://en.wikipedia.org/wiki/The_Sleuth_Kit).
First `gunzip disk.flag.img.gz` 
Then use `sleuthkit` on the `disk.flag.img`:
```
$ mmls disk.flag.img 
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000206847   0000204800   Linux (0x83)
003:  000:001   0000206848   0000360447   0000153600   Linux Swap / Solaris x86 (0x82)
004:  000:002   0000360448   0000614399   0000253952   Linux (0x83)
-------------------------------------------------------------------------
$ fls -o 360448 disk.flag.img 
d/d 451:	home
d/d 11:	lost+found
d/d 12:	boot
d/d 1985:	etc
d/d 1986:	proc
d/d 1987:	dev
d/d 1988:	tmp
d/d 1989:	lib
d/d 1990:	var
d/d 3969:	usr
d/d 3970:	bin
d/d 1991:	sbin
d/d 1992:	media
d/d 1993:	mnt
d/d 1994:	opt
d/d 1995:	root
d/d 1996:	run
d/d 1997:	srv
d/d 1998:	sys
d/d 2358:	swap
V/V 31745:	$OrphanFiles
----------------------------------------------------------------
$ fls -o 360448 disk.flag.img  1995
r/r 2363:	.ash_history
d/d 3981:	my_folder
----------------------------------------------------------------
$ fls -o 360448 disk.flag.img 3981
r/r * 2082(realloc):	flag.txt
r/r 2371:	flag.uni.txt
------------------------------------------------------
$ icat -o 360448 disk.flag.img 2082
            3.449677            13.056403
$ icat -o 360448 disk.flag.img 2371
picoCTF{by73_5urf3r_adac6cb4}
```
***
And the is the flag:
**picoCTF{by73_5urf3r_adac6cb4}**
