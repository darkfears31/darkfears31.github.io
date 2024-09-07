---
title: "John_Pollard from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# John_Pollard from PicoCTF
[page](https://play.picoctf.org/practice/challenge/6?category=2&difficulty=2&page=2)
>Description
>Sometimes RSA [certificates](https://jupiter.challenges.picoctf.org/static/c882787a19ed5d627eea50f318d87ac5/cert) are breakable

>Hints
>The flag is in the format picoCTF{p,q}
>Try swapping p and q if it does not work

Just search `RSA certificate decoder`and i got to this site: [certlogik](https://certlogik.com/decoder/), and enter the certificate and search for modulus , It's: `4966306421059967`. 
Then go to [FactorDB](https://factordb.com/index.php?query=) and enter the modulus to factorize it. After that you will get the `p and q`
```
p= 67867967
q= 73176001
```
Then the flag will be:
**picoCTF{73176001,67867967}**
