---
title: "Specialer from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Specialer from PicoCTF
[page](https://play.picoctf.org/practice/challenge/378?category=5&difficulty=2&page=1&search=&solved=0)
> Description
Reception of Special has been cool to say the least. That's why we made an exclusive version of Special, called Secure Comprehensive Interface for Affecting Linux Empirically Rad, or just 'Specialer'. With Specialer, we really tried to remove the distractions from using a shell. Yes, we took out spell checker because of everybody's complaining. But we think you will be excited about our new, reduced feature set for keeping you focused on what needs it the most. Please start an instance to test your very own copy of Specialer.

>Hints
>What programs do you have access to?

After connecting i found out that i couldn't use `ls` nor `cat`. So from hints i knew that i had to search on what programs i had access to. To not waste time i asked ChatGPT:
```
how to check in SSH what commands i have access to while connected to ssh server
```
and i got the answer: `compgen -c` this code worked and got the list on what commands i had access to:
```
if
then
else
elif
fi
case
esac
for
select
while
until
do
done
in
function
time
{
}
!
[[
]]
coproc
.
:
[
alias
bg
bind
break
builtin
caller
cd
command
compgen
complete
compopt
continue
declare
dirs
disown
echo
enable
eval
exec
exit
export
false
fc
fg
getopts
hash
help
history
jobs
kill
let
local
logout
mapfile
popd
printf
pushd
pwd
read
readarray
readonly
return
set
shift
shopt
source
suspend
test
times
trap
true
type
typeset
ulimit
umask
unalias
unset
wait
bash
```
From this i only used `pwd` and `echo`.
First i asked GPT how to list files without using `ls`and i got this: 
`echo *`,`printf "%s\n" *` and `compgen -f`. I used `echo *`
Let's begin:
```
Specialer$ echo *
abra ala sim
Specialer$ cd abra
Specialer$ echo *
cadabra.txt cadaniel.txt
Specialer$ cat cadabra.txt 
-bash: cat: command not found
Specialer$ ./cadabra.txxt
-bash: ./cadabra.txxt: No such file or directory
Specialer$ ./cadabra.txt
-bash: ./cadabra.txt: Permission denied
```
I couldn't read file with running it so i had to search other ways e.g. reading file with `echo`
and i got to this in [stackoverflow](https://stackoverflow.com/questions/22377792/how-to-use-echo-command-to-print-out-content-of-a-text-file) :
```
echo "$(<a.txt )"
```
***
```
Specialer$ echo "$( < cadabra.txt)"
Nothing up my sleeve!
Specialer$ echo "$( < cadaniel.txt)"
Yes, I did it! I really did it! I'm a true wizard!
```
It worked! but we didn't get the flag so let's search in other directories. 
After not very long of searching i got to this:
```
Specialer$ cd ala
Specialer$ echo *
kazam.txt mode.txt
Specialer$ echo "$( < kazam.txt)"
return 0 picoCTF{y0u_d0n7_4ppr3c1473_wh47_w3r3_d01ng_h3r3_d5ef8b71}
```
Here we go the flag is:
**picoCTF{y0u_d0n7_4ppr3c1473_wh47_w3r3_d01ng_h3r3_d5ef8b71}**

***
To learn the challenge you can visit this [page](https://pwn.college/linux-luminarium/globbing/)
