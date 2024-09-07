---
title: "Mus1c from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Mus1c from PicoCTF
[page](https://play.picoctf.org/practice/challenge/15?difficulty=2&page=10&search=&solved=0)
> Description
I wrote you a [song](https://jupiter.challenges.picoctf.org/static/c594d8d915de0129d92b4c41e25a2313/lyrics.txt). Put it in the picoCTF{} flag format.

>Hints
>Do you think you can master rockstar?

I downloaded the file and i got this:

```
Pico's a CTFFFFFFF
my mind is waitin
It's waitin

Put my mind of Pico into This
my flag is not found
put This into my flag
put my flag into Pico


shout Pico
shout Pico
shout Pico

My song's something
put Pico into This

Knock This down, down, down
put This into CTF

shout CTF
my lyric is nothing
Put This without my song into my lyric
Knock my lyric down, down, down

shout my lyric

Put my lyric into This
Put my song with This into my lyric
Knock my lyric down

shout my lyric

Build my lyric up, up ,up

shout my lyric
shout Pico
shout It

Pico CTF is fun
security is important
Fun is fun
Put security with fun into Pico CTF
Build Fun up
shout fun times Pico CTF
put fun times Pico CTF into my song

build it up

shout it
shout it

build it up, up
shout it
shout Pico
```
It's music text. From Hints i thought i could get something, i searched "master rockstar" and i got this site [codewithrockstar](https://codewithrockstar.com/online) i entered the text and got the decimal encrypted text:
```
114
114
114
111
99
107
110
114
110
48
49
49
51
114
```
I went to [cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Decimal('Line%20feed',false)) , Entered this decimals and got this: 
>rrrocknrn0113r

Wrapped it with picoCTF{} and got the flag:
**picoCTF{rrrocknrn0113r}**
