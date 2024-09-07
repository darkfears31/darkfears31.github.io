---
title: "Serpentine from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Serpentine from PicoCTF
[page](https://play.picoctf.org/practice/challenge/251?page=9)


>Description
> Find the flag in the Python script.

While examining provided code we find that none of the choices provide flag.
```
 while True:
    print('a) Print encouragement')
    print('b) Print flag')
    print('c) Quit\n')
    choice = input('What would you like to do? (a/b/c) ')
    
    if choice == 'a':
      print_encouragement()
      
    elif choice == 'b':
      print('\nOops! I must have misplaced the print_flag function! Check my source code!\n\n')
      
    elif choice == 'c':
      sys.exit(0)
```

Let's go search what function gives us the flag. After bit searching we find function :
```
flag_enc = chr(0x15) + chr(0x07) + chr(0x08) + chr(0x06) + chr(0x27) + chr(0x21) + chr(0x23) + chr(0x15) + chr(0x5c) + chr(0x01) + chr(0x57) + chr(0x2a) + chr(0x17) + chr(0x5e) + chr(0x5f) + chr(0x0d) + chr(0x3b) + chr(0x19) + chr(0x56) + chr(0x5b) + chr(0x5e) + chr(0x36) + chr(0x53) + chr(0x07) + chr(0x51) + chr(0x18) + chr(0x58) + chr(0x05) + chr(0x57) + chr(0x11) + chr(0x3a) + chr(0x0f) + chr(0x0a) + chr(0x5b) + chr(0x57) + chr(0x41) + chr(0x55) + chr(0x0c) + chr(0x59) + chr(0x14)


def print_flag():
  flag = str_xor(flag_enc, 'enkidu')
  print(flag)
```

**def print_flag()** function decrypts encrypted flag. 
So lets take this function and give it to one of the **Choice** e.g. 
```
if choice == 'a':
      print_encouragement()
```
|
|
|
```
if choice == 'a':
      print_flag()
```
and save it.
Run the saved program and in choices choose **a** : 
```
giorgi@darkfears31:~/Desktop$ python3 serpentine.py 

    Y
  .-^-.
 /     \      .- ~ ~ -.
()     ()    /   _ _   `.                     _ _ _
 \_   _/    /  /     \   \                . ~  _ _  ~ .
   | |     /  /       \   \             .' .~       ~-. `.
   | |    /  /         )   )           /  /             `.`.
   \ \_ _/  /         /   /           /  /                `'
    \_ _ _.'         /   /           (  (
                    /   /             \  \
                   /   /               \  \
                  /   /                 )  )
                 (   (                 /  /
                  `.  `.             .'  /
                    `.   ~ - - - - ~   .'
                       ~ . _ _ _ _ . ~

Welcome to the serpentine encourager!


a) Print encouragement
b) Print flag
c) Quit

What would you like to do? (a/b/c) a
picoCTF{7h3_r04d_l355_7r4v3l3d_aa2340b2}
a) Print encouragement
b) Print flag
c) Quit
```
We got the flag: 
**picoCTF{7h3_r04d_l355_7r4v3l3d_aa2340b2}**
