---
title: "Operation-Oni from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Operation-Oni from PicoCTF
[page](https://play.picoctf.org/practice/challenge/284?category=4&difficulty=2&page=1)
>Description
>Download this disk image, find the key and log into the remote machine.Note: if you are using the webshell, download and extract the disk image into `/tmp` not your home directory.
 [Download disk image](https://artifacts.picoctf.net/c/70/disk.img.gz)
 - Remote machine: `ssh -i key_file -p <port> ctf-player@saturn.picoctf.net`

First download it and it's `.gz` file for that use `gunzip` and we are left with `disk.img`.
Then i did this:
```
$ mmls disk.img 
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000206847   0000204800   Linux (0x83)
003:  000:001   0000206848   0000471039   0000264192   Linux (0x83)
$ sudo mount disk.img something -o offset=$((206848*512))
```
I mounted `disk.img` into `something`. I became superuser with `sudo su` and proceeded doing this:
```
┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf]
└─# cd something/

┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf/something]
└─# ls
bin  boot  dev  etc  home  lib  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf/something]
└─# cd root

┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf/something/root]
└─# ls

┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf/something/root]
└─# ls -all
total 4
drwx------  3 root root 1024 Oct  6  2021 .
drwxr-xr-x 21 root root 1024 Oct  6  2021 ..
-rw-------  1 root root   36 Oct  6  2021 .ash_history
drwx------  2 root root 1024 Oct  6  2021 .ssh

┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf/something/root]
└─# cat .ash_history 
ssh-keygen -t ed25519
ls .ssh/
halt
┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf/something/root]
└─# cd .ssh/

┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf/something/root/.ssh]
└─# ls
id_ed25519  id_ed25519.pub

┌──(root㉿darkfears31)-[/home/giorgi/Desktop/picoctf/something/root/.ssh]
└─# cat id_ed25519
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBgrXe4bKNhOzkCLWOmk4zDMimW9RVZngX51Y8h3BmKLAAAAJgxpYKDMaWC
gwAAAAtzc2gtZWQyNTUxOQAAACBgrXe4bKNhOzkCLWOmk4zDMimW9RVZngX51Y8h3BmKLA
AAAECItu0F8DIjWxTp+KeMDvX1lQwYtUvP2SfSVOfMOChxYGCtd7hso2E7OQItY6aTjMMy
KZb1FVmeBfnVjyHcGYosAAAADnJvb3RAbG9jYWxob3N0AQIDBAUGBw==
-----END OPENSSH PRIVATE KEY-----

```
I took the key that we need to connect to `ssh` and created file named `key_file`as requested in `Description` 
```
$ nano key_file
$ chmod 600 key_file 
$ ssh -i key_file -p 60683 ctf-player@saturn.picoctf.net
The authenticity of host '[saturn.picoctf.net]:60683 ([13.59.203.175]:60683)' can't be established.
ED25519 key fingerprint is SHA256:XBSvB1lk28EctsAVdKJtsl0A7C5bonqPrvHCYH8aEy4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[saturn.picoctf.net]:60683' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 6.5.0-1016-aws x86_64)

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

ctf-player@challenge:~$ ls
flag.txt
ctf-player@challenge:~$ cat flag.txt 
picoCTF{k3y_5l3u7h_b5066e83}
```
***
And here is the flag:
**picoCTF{k3y_5l3u7h_b5066e83}**
