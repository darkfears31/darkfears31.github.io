---
title: "forky from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# forky from PicoCTF
[page](https://play.picoctf.org/practice/challenge/24?difficulty=2&page=1&search=forky)
>Description
>In [this program](https://jupiter.challenges.picoctf.org/static/78c5ee78a6593e52ba5e5b08bc9f13c1/vuln), identify the last integer value that is passed as parameter to the function doNothing().

>Hints
>What happens when you fork? The flag is picoCTF{IntegerYouFound}. For example, if you found that the last integer passed was 1234, the flag would be picoCTF{1234}

Let's run `vuln` in`ghidra` and see the function:
```
undefined4 main(void)

{
  int *piVar1;
  
  piVar1 = (int *)mmap((void *)0x0,4,3,0x21,-1,0);
  *piVar1 = 1000000000;
  fork();
  fork();
  fork();
  fork();
  *piVar1 = *piVar1 + 0x499602d2;
  doNothing(*piVar1);
  return 0;
}

```
Best explanation to what `fork` does:
```
fork ();   // Line 1
fork ();   // Line 2
fork ();   // Line 3
       L1       // There will be 1 child process 
    /     \     // created by line 1.
  L2      L2    // There will be 2 child processes
 /  \    /  \   //  created by line 2
L3  L3  L3  L3  // There will be 4 child processes 
                // created by line 3
```
`Total Number of Processes = 2^n, where n is the number of fork system calls.`
So the thing is last input that will be in `doNothing(*piVar1)` will be: $$1000000000 + 16*0x499602d2$$
Let's convert `1000000000` to hex with this [site](https://www.rapidtables.com/convert/number/decimal-to-hex.html): 3B9ACA00 --> 0x3B9ACA00.
In C compiler type this code: 
```
#include <stdio.h> 
int main() { 
    int result = 0x3B9ACA00 + 16 * 0x499602d2;
    printf("%d/n", result);
    return 0;
}

```
this will give output of: -721750240
So the flag will be:
**picoCTF{-721750240v}**
