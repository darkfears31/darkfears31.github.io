---
title: "Photon Lockdown from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# Photon Lockdown from HTB
[page](https://app.hackthebox.com/challenges/Photon%20Lockdown)
> Description
> We've located the adversary's location and must now secure access to their Optical Network Terminal to disable their internet connection. Fortunately, we've obtained a copy of the device's firmware, which is suspected to contain hardcoded credentials. Can you extract the password from it?

Download the file and `unzip`it with password.
Then run the `file`command to learn what type of file it is:
```
$ file rootfs 
rootfs: Squashfs filesystem, little endian, version 4.0, zlib compressed, 10936182 bytes, 910 inodes, blocksize: 131072 bytes, created: Sun Oct  1 07:02:43 2023
```
It is a `squashfs`compressed file. To extract files do `unsquashfs`:
```
$ sudo unsquashfs rootfs
Parallel unsquashfs: Using 8 processors
865 inodes (620 blocks) to write

[==========================================================================================================================================================|] 1485/1485 100%

created 440 files
created 45 directories
created 187 symlinks
created 238 devices
created 0 fifos
created 0 sockets
created 0 hardlinks
giorgi@darkfears31:~/Desktop/HTB/ONT$ ls
fwu_ver  hw_ver  rootfs  squashfs-root
```
And there is the new directory. We have to find the flag in it.
```
grep --include=*.{txt,conf,xml,php} -rnw '.' -e 'HTB' 2>/dev/null
```
now this particular command what this will do is check all file if there is any text, config, `xml` and `php` files contain the word `HTB` if there is any that will print it and in the `config_default.xml` finally found the flag.
```
$ grep --include=*.{txt,conf,xml,php} -rnw '.' -e 'HTB' 2>/dev/null
./etc/config_default.xml:244:<Value Name="SUSER_PASSWORD" Value="HTB{N0w_Y0u_C4n_L0g1n}"/>
```
And the flag is:
**HTB{N0w_Y0u_C4n_L0g1n}**
