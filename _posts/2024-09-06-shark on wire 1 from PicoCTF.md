---
title: "shark-on-wire-1 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# shark-on-wire-1 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/30?page=17&search=)
> Description
We found this [packet capture](https://jupiter.challenges.picoctf.org/static/483e50268fe7e015c49caf51a69063d0/capture.pcap). Recover the flag.

>Hints
>What are streams?

From hints i knew that i should've searched trough streams.
I opened file and immediately saw that there was three types of packets: **TCP, UDP and ARP**.
ARP has no stream but TCP and UDP has.
So let's search trough TCP and UDP streams.
![](/assets/images/Screencast from 2024-07-14 15-40-39.mp4)
There is nothing in TCP streams so now let's go trough UDP streams.
![](/assets/images/Screencast from 2024-07-14 15-44-07.mp4)
Here we go, Flag is:
**picoCTF{StaT31355_636f6e6e}**
***
