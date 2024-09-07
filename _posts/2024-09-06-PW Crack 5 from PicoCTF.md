---
title: "Pw-Crack-5 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Pw-Crack-5 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/249?category=5&difficulty=2&page=1)
>Description
Can you crack the password to get the flag?Download the password checker [here](https://artifacts.picoctf.net/c/33/level5.py) and you'll need the encrypted [flag](https://artifacts.picoctf.net/c/33/level5.flag.txt.enc) and the [hash](https://artifacts.picoctf.net/c/33/level5.hash.bin) in the same directory too. Here's a [dictionary](https://artifacts.picoctf.net/c/33/dictionary.txt) with all possible passwords based on the password conventions we've seen so far.

>Hint 2
>You may need to trim the whitespace from the dictionary word before hashing. Look up the Python string function, `strip`

I'm tired so i can't really explain so code should look like this:
```
import hashlib

  

### THIS FUNCTION WILL NOT HELP YOU FIND THE FLAG --LT ########################
def str_xor(secret, key):
#extend key to secret length
new_key = key
i = 0
while len(new_key) < len(secret):
new_key = new_key + key[i]
i = (i + 1) % len(key)
return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c,new_key_c) in zip(secret,new_key)])

###############################################################################

  

flag_enc = open('level5.flag.txt.enc', 'rb').read()
correct_pw_hash = open('level5.hash.bin', 'rb').read()
dictionary = open('dictionary.txt', 'r').readlines()

def hash_pw(pw_str):
pw_bytes = bytearray()
pw_bytes.extend(pw_str.encode())
m = hashlib.md5()
m.update(pw_bytes)
return m.digest()

def level_5_pw_check():
#user_pw = input("Please enter correct password for flag: ")
for passwd in dictionary:
user_pw = passwd.strip()
user_pw_hash = hash_pw(user_pw)
if( user_pw_hash == correct_pw_hash ):
print("Welcome back... your flag, user:")
decryption = str_xor(flag_enc.decode(), user_pw)
print(decryption)
return
print("That password is incorrect")

level_5_pw_check()
for i in dictionary:
level_5_pw_check(i.strip())
```
The flag is:
**picoCTF{h45h_sl1ng1ng_fffcda23}**
