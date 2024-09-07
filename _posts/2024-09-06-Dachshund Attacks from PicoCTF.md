---
title: "dachshund Attacks from HTB"
categories: [PicoCTF]
tags: [PicoCTF]
---
# dachshund Attacks from PicoCTF
[page](https://play.picoctf.org/practice/challenge/159?page=12)

> #Description
> What if `d` is too small? Connect with `nc mercury.picoctf.net 36463`.
> #Hints
> What do you think about my pet? [dachshund.jpg](https://mercury.picoctf.net/static/6b0ca75093bbcaf96c39eb47c048aef2/dachshund.jpg)

When i try to connect to NetCat it gives me this :
```
giorgi@darkfears31:~/Desktop/codes$ nc mercury.picoctf.net 36463
Welcome to my RSA challenge!
e: 31112241294171519915631069920318765874521238124168532913905531116281510964904836626531137188361696775726669633236731800874399678248260264947749912997148601784968269730005634476477862540208930191622619169157075204380154084337158306781664568496237616115649269687570597717334851672153843093195637690313050752117
n: 85635598126252735438881888442407668239192050715723339654463069053325509579219538866514020803999770195512118393682906468050212720686445902091367519198257476966342017681992984849960425289791021406002172094425972904591575441946196719352301220657778704847991912101569504824732852113678692431809991409458165613999
c: 54705091974349935726942548296315109290257106279595816508356656837164949627539269819937594060225175475478668362010485253394952517007665513305959496959527710482323736166210511879248833789638273135103101901983204721324048645485833687958154041543004697717488420405722769199726337538499734448671801220406476705868
```

From the picture i just downloaded we get nothing but the dog in the .jpg file.
After looking at this we need to somehow connect this to **The Wiener Attack**.

After long time of searching i found simple python code of **The wiener Attack**
Here it is: 
```
import owiener
from Crypto.Util.number import long_to_bytes
#--------Data--------#

N = <n> <----- change N
e = <e> <----- change e
c = <c> <----- change C

#--------Wiener's attack--------#

d = owiener.attack(e, N)

if d:
    m = pow(c, d, N)
    flag = long_to_bytes(m).decode()
    print(flag)
else:
    print("Wiener's Attack failed.")
```
If you type your N,e,c this code will decode it and give you the flag: 
**picoCTF{proving_wiener_2635457}**
