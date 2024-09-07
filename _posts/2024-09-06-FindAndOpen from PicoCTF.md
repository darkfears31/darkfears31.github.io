---
title: "FindAndOpen from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# FindAndOpen from PicoCTF
[page](https://play.picoctf.org/practice/challenge/348?category=4&difficulty=1&page=1&search=)
>Description
>Someone might have hidden the password in the trace file.Find the key to unlock [this file](https://artifacts.picoctf.net/c/497/flag.zip). [This tracefile](https://artifacts.picoctf.net/c/497/dump.pcap) might be good to analyze.

So download the `.pcap` and search for the encrypted password and its located it: ![](/assets/images/Screenshot from 2024-07-21 23-01-41.png)
This is base64 encoded strings which needs to be added some letters to work:
```
$ echo "aaAABBHHPJGTFRLKVGhpcyBpcyB0aGUgc2VjcmV0OiBwaWNvQ1RGe1IzNERJTkdfTE9LZF8=" | base64 -d
i��<���This is the secret: picoCTF{R34DING_LOKd_
```
Enter this as password of `.zip`file and you will get the flag:
**picoCTF{R34DING_LOKd_fil56_succ3ss_c2e6d949}**
