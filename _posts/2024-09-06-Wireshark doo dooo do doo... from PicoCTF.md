---
title: "wireshark-doo-dooo-do-doo from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# wireshark-doo-dooo-do-doo from PicoCTF
[page](https://play.picoctf.org/practice/challenge/115?page=14)
>Description
Can you find the flag? [shark1.pcapng](https://mercury.picoctf.net/static/ea41c400c3c7b4a63406e5e607d362ab/shark1.pcapng).

open the **shark1.pcapng** file and first i do is "CTRL+F" to search anything that has "pico" or "flag" in it and i found nothing.
So we have to divide this messages and read their "Stream".
We can divide them with :
```
tcp.stream eq <number>
```
if we type **tcp.stream eq 1** and follow their stream we get this:
![](/assets/images/Screenshot from 2024-07-13 02-10-59.png)
This gives us nothing so we search in **tcp.stream eq 2** and still we find nothing till we reach **tcp.stream eq 5**. If we follow the stream now we get this:

![](/assets/images/Screenshot from 2024-07-13 02-13-02.png)
and here is our **rot13** encoded flag:
```
cvpbPGS{c33xno00_1_f33_h_qrnqorrs}
```
Let's take it to [CyberChef](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,13)&input=Cg) we get the flag:
```
picoCTF{p33kab00_1_s33_u_deadbeef}
```
