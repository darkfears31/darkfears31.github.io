---
title: "BabyEncryption from HTB"
categories: [Hack The Box]
tags: [Hack The Box]
---
# BabyEncryption from HackTheBox
[page](https://app.hackthebox.com/challenges/BabyEncryption)


> Description
> You are after an organised crime group which is responsible for the illegal weapon market in your country. As a secret agent, you have infiltrated the group enough to be included in meetings with clients. During the last negotiation, you found one of the confidential messages for the customer. It contains crucial information about the delivery. Do you think you can decrypt it?

There is two files. First `chall.py`which encrypts the text and second the encrypted text.
```
$ cat chall.py 
import string
from secret import MSG

def encryption(msg):
    ct = []
    for char in msg:
        ct.append((123 * char + 18) % 256)
    return bytes(ct)

ct = encryption(MSG)
f = open('./msg.enc','w')
f.write(ct.hex())
f.close()
```

```
$ cat msg.enc 
6e0a9372ec49a3f6930ed8723f9df6f6720ed8d89dc4937222ec7214d89d1e0e352ce0aa6ec82bf622227bb70e7fb7352249b7d893c493d8539dec8fb7935d490e7f9d22ec89b7a322ec8fd80e7f8921
```

So we have to reverse the encryption. I don't have knowledge of coding so i went to `chatgpt`and got this code:
```
$ cat ll.py 
def decryption(ct):
    msg = []
    mod_inv_123 = pow(123, -1, 256)  # Precompute the modular inverse of 123 mod 256
    for byte in ct:
        original_char = chr(((byte - 18) * mod_inv_123) % 256)  # Calculate the original character
        msg.append(original_char)
    return ''.join(msg)

# Read the encrypted hex string from the file
with open('msg.enc', 'r') as f:
    hex_string = f.read().strip()  # Ensure no extra whitespace is read

# Convert the hex string back to bytes
ct_bytes = bytes.fromhex(hex_string)

# Decrypt the message
decrypted_msg = decryption(ct_bytes)
print(decrypted_msg)
```
I ran it and got the flag:
```
$ python3 ll.py 
Th3 nucl34r w1ll 4rr1v3 0n fr1d4y.
HTB{l00k_47_y0u_r3v3rs1ng_3qu4710n5_c0ngr475}
```
**HTB{l00k_47_y0u_r3v3rs1ng_3qu4710n5_c0ngr475}**
