---
title: "eavesdrop from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# eavesdrop from PicoCTF
[page](https://play.picoctf.org/practice/challenge/264?category=4&difficulty=2&page=1)

>Description
>Download this packet capture and find the flag.
>- [Download packet capture](https://artifacts.picoctf.net/c/133/capture.flag.pcap)

>Hints
>All we know is that this packet capture includes a chat conversation and a file transfer.

So The file is `.pcap`. Let's search trough it.
First i went trough `HTTP` and found nothing and i was left with `TCP`, i followed stream and in 0 stream i found the conversation as mentioned in `Hints`:
```
Hey, how do you decrypt this file again?

You're serious?

Yeah, I'm serious

*sigh* openssl des3 -d -salt -in file.des3 -out file.txt -k supersecretpassword123

Ok, great, thanks.

Let's use Discord next time, it's more secure.

C'mon, no one knows we use this program like this!

Whatever.

Hey.

Yeah?

Could you transfer the file to me again?

Oh great. Ok, over 9002?

Yeah, listening.

Sent it

Got it.

You're unbelievable
```
Great we found how the flag is encrpted:
```
openssl des3 -d -salt -in file.des3 -out file.txt -k supersecretpassword123
```
Now lets go up the stream and on stream number 2 i found the salted flag, convert it to `raw` and save it as `file.des3` then run the code mentioned above. After all that you will get the flag:
**picoCTF{nc_73115_411_5786acc3}**
