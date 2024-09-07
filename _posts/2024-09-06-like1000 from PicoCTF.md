---
title: "Like1000 from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Like1000 from PicoCTF
[page](https://play.picoctf.org/practice/challenge/81?category=4&difficulty=2&page=2)
>Description
>This [.tar file](https://jupiter.challenges.picoctf.org/static/52084b5ad360b25f9af83933114324e0/1000.tar) got tarred a lot.
>Hints
>Try and script this, it'll save you a lot of time

Download the file and it's named `1000.tar`first lets extract the file with:
```
tar -xf 1000.tar
```
And we are left with `999.tar` from Hints i guess there is nested `.tar`files in it so we have to write script i wrote script in bash:
```
#!bin/bash

# Loop through the numbers 1000 down to 1
for ((i=1000; i>=1; i--))
do
  # Extract the tar file with the current number
  tar -xf "$i.tar"
done
```
This code goes from 1000 to one and extracts the files.
Make this code exacutable `chmod +x <file_name>`and run it. We are left with 1000 `.tar` files and flag.png open it and there is the flag:
**picoCTF{l0t5_0f_TAR5}** 
***
To delete all of that `.tar` files you can modify the code little bit:
```
#!bin/bash

# Loop through the numbers 1000 down to 1
for ((i=1000; i>=1; i--))
do
  # Extract the tar file with the current number
  rm -r "$i.tar"
done
```
