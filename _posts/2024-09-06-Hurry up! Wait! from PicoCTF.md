---
title: "Hurry Up! Wait! from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Hurry Up! Wait! from PicoCTF
[page](https://play.picoctf.org/practice/challenge/165?page=12)


> **Description**
> [svchost.exe](https://mercury.picoctf.net/static/61cffe4f7cc04119c857dabd015d6c4a/svchost.exe)

Looking at this CTF category we see that it's reverse engineering, but lets run the program.
For some reason i cant run it i.g. because i'm on Kali  i can't run it. But this program will take forever to run for some reason.

Let's go to Ghidra my beloved reverse engineering tool.
Scrolling trough functions we see some unusual function named **undefined FUN_0010298a()**
```
                             **************************************************************
                             *       FUNCTION           *
                             **************************************************************
                             undefined FUN_0010298a()
             undefined         AL:1           <RETURN>
                             FUN_0010298a                                    XREF[3]:     FUN_00101fcc:0010201a(c), 
                                                                                          00102ea0, 00103588(*)  
        0010298a 55              PUSH       RBP
        0010298b 48 89 e5        MOV        RBP,RSP
        0010298e 48 bf 00        MOV        RDI,0x38d7ea4c68000
                 80 c6 a4 
                 7e 8d 03 00
        00102998 e8 d3 f1        CALL       <EXTERNAL>::ada__calendar__delays__delay_for     undefined ada__calendar__delays_
                 ff ff
        0010299d e8 74 fc        CALL       FUN_00102616                                     undefined FUN_00102616()
                 ff ff
        001029a2 e8 03 fb        CALL       FUN_001024aa                                     undefined FUN_001024aa()
                 ff ff
        001029a7 e8 c6 f9        CALL       FUN_00102372                                     undefined FUN_00102372()
                 ff ff
        001029ac e8 31 fc        CALL       FUN_001025e2                                     undefined FUN_001025e2()
                 ff ff
        001029b1 e8 9c fe        CALL       FUN_00102852                                     undefined FUN_00102852()
                 ff ff
        001029b6 e8 cb fe        CALL       FUN_00102886                                     undefined FUN_00102886()
                 ff ff
        001029bb e8 fa fe        CALL       FUN_001028ba                                     undefined FUN_001028ba()
                 ff ff
        001029c0 e8 5d ff        CALL       FUN_00102922                                     undefined FUN_00102922()
                 ff ff
        001029c5 e8 dc f9        CALL       FUN_001023a6                                     undefined FUN_001023a6()
                 ff ff
        001029ca e8 67 f7        CALL       FUN_00102136                                     undefined FUN_00102136()
                 ff ff
        001029cf e8 32 f8        CALL       FUN_00102206                                     undefined FUN_00102206()
                 ff ff
        001029d4 e8 31 f9        CALL       FUN_0010230a                                     undefined FUN_0010230a()
                 ff ff
        001029d9 e8 28 f8        CALL       FUN_00102206                                     undefined FUN_00102206()
                 ff ff
        001029de e8 97 fb        CALL       FUN_0010257a                                     undefined FUN_0010257a()
                 ff ff
        001029e3 e8 06 ff        CALL       FUN_001028ee                                     undefined FUN_001028ee()
                 ff ff
        001029e8 e8 21 fa        CALL       FUN_0010240e                                     undefined FUN_0010240e()
                 ff ff
        001029ed e8 f4 fc        CALL       FUN_001026e6                                     undefined FUN_001026e6()
                 ff ff
        001029f2 e8 8b fd        CALL       FUN_00102782                                     undefined FUN_00102782()
                 ff ff
        001029f7 e8 f2 fe        CALL       FUN_001028ee                                     undefined FUN_001028ee()
                 ff ff
        001029fc e8 a5 f9        CALL       FUN_001023a6                                     undefined FUN_001023a6()
                 ff ff
        00102a01 e8 08 fa        CALL       FUN_0010240e                                     undefined FUN_0010240e()
                 ff ff
        00102a06 e8 33 f9        CALL       FUN_0010233e                                     undefined FUN_0010233e()
                 ff ff
        00102a0b e8 96 f9        CALL       FUN_001023a6                                     undefined FUN_001023a6()
                 ff ff
        00102a10 e8 5d f9        CALL       FUN_00102372                                     undefined FUN_00102372()
                 ff ff
        00102a15 e8 ec f7        CALL       FUN_00102206                                     undefined FUN_00102206()
                 ff ff
        00102a1a e8 87 f9        CALL       FUN_001023a6                                     undefined FUN_001023a6()
                 ff ff
        00102a1f e8 32 ff        CALL       FUN_00102956                                     undefined FUN_00102956()
                 ff ff

```

At start we can see why this program delays to run.
```
ada__calendar__delays__delay_for(1000000000000000);
```

Let's move on examining the functions.
Clicking to first function 
```
void FUN_00102616(void)

{
  ada__text_io__put__4(&DAT_00102cd8,&DAT_00102cb8);
  return;
}
```
something named **ada__text_io__put__4** inputs a text. 
```
(&DAT_00102cd8,&DAT_00102cb8)
```
Lets examine **DAT_00102cd8** and program takes us to letter "p". 
**&DAT_00102cb8** gives us "01H" i don't know what it is but we don't need it.


Let's go to next function 
```

void FUN_001024aa(void)

{
  ada__text_io__put__4(&DAT_00102cd1,&DAT_00102cb8);
  return;
}
```
**&DAT_00102cd1** gives us "i".
move on to next function.
```
void FUN_00102372(void)

{
  ada__text_io__put__4(&DAT_00102ccb,&DAT_00102cb8);
  return;
}
```
**&DAT_00102ccb** ---> "c". Now we know that we're dealing with the flag. I don't have that much knowledge to automate the process so im going to go trough all the functions and we get the flag: 
**picoCTF{d15a5m_ftw_dfbdc5d}**
