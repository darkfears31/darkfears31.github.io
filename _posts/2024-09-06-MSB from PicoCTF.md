---
title: "MSB from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# MSB from PicoCTF
[page](https://play.picoctf.org/practice/challenge/359?category=4&difficulty=2&page=1)
>Description
>This image passes LSB statistical analysis, but we can't help but think there must be something to the visual artifacts present in this image...Download the image [here](https://artifacts.picoctf.net/c/303/Ninja-and-Prince-Genji-Ukiyoe-Utagawa-Kunisada.flag.png)
>Hints
>What's causing the 'corruption' of the image?

So let's download it and see that it's old ass art. From `strings` there is nothing readable, nor from `cat`,`zsteg` ,`steghide`, `binwalk`. 
MSB ---> Most Significant Byte
LSB ---> Least Significant Byte
So Lets go to stegsolve. Download it from [here](https://github.com/zardus/ctf-tools/blob/master/stegsolve/install).
```
$ java -jar stegsolve.jar 
```
With this `stegosolve` is started.
You will see the popup on the left top side of screen, click file and open the `.png`file you downloaded from site.
Then click Analyze end Data extract. 
In bit planes choose:
Red -->7
Blue -->7
Green -->7
Then save text and read it with:
```
strings <text_file> | less
```
When you will see the text just type: `/pico`to find the flag:
```
220a7069636f4354 467b31355f793075  ".picoCT F{15_y0u
725f71756535375f 7175317830373163  r_que57_ qu1x071c
5f30725f68337230 31635f3234643535  _0r_h3r0 1c_24d55
6265657d0a0a2254 686f752068617374  bee}.."T hou hast
```
***
The flag is:
**picoCTF{15_y0ur_que57_qu1x071c_0r_h3r01c_24d55bee}**
