---
title: "SansAlpha from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# SansAlpha from PicoCTF
[page](https://play.picoctf.org/practice/challenge/436?category=5&difficulty=2&page=1&search=&solved=0) 
> Description
The Multiverse is within your grasp! Unfortunately, the server that contains the secrets of the multiverse is in a universe where keyboards only have numbers and (most) symbols.`ssh -p 58046 ctf-player@mimas.picoctf.net`Use password: `6dd28e9b`

>Hints
>Where can you get some letters?

Let's connect to machine.
```
$ ssh -p 58046 ctf-player@mimas.picoctf.net
The authenticity of host '[mimas.picoctf.net]:58046 ([52.15.88.75]:58046)' can't be established.
ED25519 key fingerprint is SHA256:n/hDgUtuTTF85Id7k2fxmHvb6rrLrACHNM6xLZ46AqQ.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:33: [hashed name]
    ~/.ssh/known_hosts:35: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[mimas.picoctf.net]:58046' (ED25519) to the list of known hosts.
ctf-player@mimas.picoctf.net's password: 
Permission denied, please try again.
ctf-player@mimas.picoctf.net's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 6.5.0-1016-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

SansAlpha$
```

As said in Description we can't use commands which contains letters (e.g. cd, pwd, ls...) 
From Hints i had to search `linux cmd commands without letters` and came with this site:
[tldp.org](https://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x11655.htm) where i found commands without letters. 
We will be needing this in our task: 
```
? (question mark)
this can represent any _single_ character. If you specified something at the command line like "hd?" GNU/Linux would look for hda, hdb, hdc and every other letter/number between a-z, 0-9.

* (asterisk)
this can represent any number of characters (including zero, in other words, zero or more characters). If you specified a "cd*" it would use "cda", "cdrom", "cdrecord" and _anything_ that starts with “cd” also including “cd” itself. "m*l" could by mill, mull, ml, and anything that starts with an m and ends with an l.

[!]
This construct is similar to the [ ] construct, except rather than matching any characters inside the brackets, it'll match any character, as long as it is not listed between the [ and ]. This is a logical NOT. For example _rm myfile[!9]_ will remove all myfiles* (ie. myfiles1, myfiles2 etc) but won't remove a file with the number 9 anywhere within it's name.

. (dot)
will match _any single character_, equivalent to ? (question mark) in standard wildcard expressions. Thus, "m.a" matches "mpa" and "mea" but not "ma" or "mppa".
```
So let's begin:
```
SansAlpha$ /*
bash: /bin: Is a directory
```
We found our first directory.
```
SansAlpha$ ./*
bash: ./blargh: Is a directory

SansAlpha$ ./*/*
bash: ./blargh/flag.txt: Permission denied
```
We found the flag but we do not have permission to read it. First thing i know that this `flag.txt` is located in `./blargh/flag.txt`.
If we don't have permission to read the file maybe we can `base64` encode it. so let's search it.
Using logic `base64` place is `/bin/base64` so let's search it in ssh.
`/bin/bash64` --> `/???/??????`
```
SansAlpha$ /???/??????
/bin/base32: extra operand ‘/bin/base64’
Try '/bin/base32 --help' for more information.
```
This is because this:
```
$ /bin/base32 /bin/base36 /bin/base64
/bin/base32: extra operand ‘/bin/base64’
Try '/bin/base32 --help' for more information.

```
so we have to choose base64 with numbers and we have to execute command like this:
```
/bin/base64 ./blargh/flag.txt
```
It will be like this:
```
/???/????64 ./??????/????????
```
When i run it i get somehing like this: 
```
SansAlpha$ /???/????64 /???/????/??????/????????
/bin/base64: extra operand ‘/bin/x86_64’
Try '/bin/base64 --help' for more information.
```
we have to change something in our code so cmd will get that we are executing /bin/base64
```
SansAlpha$ /???/???[!_]64 ./??????/????????
cmV0dXJuIDAgcGljb0NURns3aDE1X211MTcxdjNyNTNfMTVfbTRkbjM1NV8xNDUyNTZlY30=
```
we had to change it because system got error because of `/bin/x86_64` so i used `[!_]` to tell terminal that do not search something that has `_` in that place, so it will get that i'm using `/bin/base64`. After execution of the code we got the base64 of the flag.txt. Let's decode it:
```
$ echo "cmV0dXJuIDAgcGljb0NURns3aDE1X211MTcxdjNyNTNfMTVfbTRkbjM1NV8xNDUyNTZlY30=" | base64 -d
return 0 picoCTF{7h15_mu171v3r53_15_m4dn355_145256ec}
```
I got the flag and it's:
picoCTF{7h15_mu171v3r53_15_m4dn355_145256ec}
***
To learn the challenge you can visit this [page](https://pwn.college/linux-luminarium/globbing/)
