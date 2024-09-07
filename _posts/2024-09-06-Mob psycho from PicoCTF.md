---
title: "mobpsycho from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# mobpsycho from PicoCTF
[page](https://play.picoctf.org/practice/challenge/420?difficulty=2&page=2)
>Descritpion
>Can you handle APKs?Download the android apk [here](https://artifacts.picoctf.net/c_titan/143/mobpsycho.apk).

>Hints
>Did you know you can `unzip` APK files?

So If you download file and `unzip`it you will see bunch of files and use this command:
```
find . -type f -name "flag.txt"
```
this will find `flag.txt` in:
```
./res/color/flag.txt
```
now read it:
```
$ cat ./res/color/flag.txt
7069636f4354467b6178386d433052553676655f4e5838356c346178386d436c5f37303364643965667d
```
decrypt it with [cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Hex('None')) and you will get the flag:
**picoCTF{ax8mC0RU6ve_NX85l4ax8mCl_703dd9ef}**
