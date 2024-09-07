---
title: "Trivial-Flag-Transfer-Protocol from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Trivial-Flag-Transfer-Protocol from PicoCTF
[page](https://play.picoctf.org/practice/challenge/103?page=14)
>   Description
Figure out how they moved the [flag](https://mercury.picoctf.net/static/4fe0f4357f7458c6892af394426eab55/tftp.pcapng).

After downloading the file i see that it's wireshark capture file. I press "CTRL+F" and search "picoCTF" but get nothing. From the title **Trivial Flag Transfer Protocol** somehow i needed to figure out to filter the packets so i did research and found **tftp** packets in wireshark.
in Wireshark i filtered out the packets: ![](/assets/images/Screenshot from 2024-07-13 16-36-32.png)
Looking at this we see that they sent files, so i have to download them like this:
File --> Export  objects --> TFTP --> download all.
Then locate the files and first read the **Insctructions.txt**
```
$ cat instructions.txt
GSGCQBRFAGRAPELCGBHEGENSSVPFBJRZHFGQVFTHVFRBHESYNTGENAFSRE.SVTHERBHGNJNLGBUVQRGURSYNTNAQVJVYYPURPXONPXSBEGURCYNA
```
It's an **rot13** encrypted text so lets go to [CyberChef]():
>TFTPDOESNTENCRYPTOURTRAFFICSOWEMUSTDISGUISEOURFLAGTRANSFER.FIGUREOUTAWAYTOHIDETHEFLAGANDIWILLCHECKBACKFORTHEPLAN

This text says **THE PLAN** so let's check it out:
```
$ cat plan
VHFRQGURCEBTENZNAQUVQVGJVGU-QHRQVYVTRAPR.PURPXBHGGURCUBGBF
```
It's still **rot13** encrypted text so decrypt it:
***
IUSEDTHEPROGRAMANDHIDITWITH-DUEDILIGENCE.CHECKOUTTHEPHOTOS
***
It says it used program so let's "dpkg" it. This **Program.deb** installs steghide so maybe something is hidden in photos.
Let's extract files from
***
picture1.bmp  picture2.bmp  picture3.bmp
***
![](/assets/images/Screenshot from 2024-07-13 16-50-41.png)
So we will need passphrase to extract it. Let's assume that it's DUEDILIGENCE because **plan** said it hid it with "DUEDILIGENCE".
There is 3 ".bmp" file so let's try it on all of them.
```
$ steghide extract -sf picture1.bmp
Enter passphrase:
steghide: could not extract any data with that passphrase!
giorgi@darkfears31:~/Desktop/picoctf$ steghide extract -sf picture2.bmp
Enter passphrase:
steghide: could not extract any data with that passphrase!
giorgi@darkfears31:~/Desktop/picoctf$ steghide extract -sf picture3.bmp
Enter passphrase:
wrote extracted data to "flag.txt".
```
Here it is : "flag.txt"
let's read it and we got the flag:
**picoCTF{h1dd3n_1n_pLa1n_51GHT_18375919}**
