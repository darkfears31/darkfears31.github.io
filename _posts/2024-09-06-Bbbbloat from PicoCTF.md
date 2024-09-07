---
title: "bbbbloat from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# bbbbloat from PicoCTF
[page](https://play.picoctf.org/practice/challenge/255?page=9)

> Description
>Can you get the flag? Reverse engineer this [binary](https://artifacts.picoctf.net/c/45/bbbbloat).

Download it and we see that this file type is ELF. Ghidra supports it so we reverse engineer it in Ghidra.

```
giorgi@darkfears31:~/Desktop$ chmod +x bbbbloat 
giorgi@darkfears31:~/Desktop$ ./bbbbloat 
What's my favorite number? 4422
Sorry, that's not it!
```

In Ghidra we search for function that has this text **"What's my favorite number?"** in **main** function. And Voila in Decompiler we see this: 

```
undefined8 FUN_00101307(void)

{
  long in_FS_OFFSET;
  int local_48;
  undefined4 local_44;
  char *local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_38 = 0x4c75257240343a41;
  local_30 = 0x3062396630664634;
  local_28 = 0x65623066635f3d33;
  local_20 = 0x4e326560623535;
  local_44 = 0xd2c49;
  printf("What\'s my favorite number? ");
  local_44 = 0xd2c49;
  __isoc99_scanf(&DAT_00102020,&local_48);
  local_44 = 0xd2c49;
  if (local_48 == 0x86187) {
    local_44 = 0xd2c49;
    local_40 = (char *)FUN_00101249(0,&local_38);
    fputs(local_40,stdout);
    putchar(10);
    free(local_40);
  }
  else {
    puts("Sorry, that\'s not it!");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

If we inspect that IF statement below the **printf("What\'s my favorite number? ");**  we see that 
```
if (local_48 == 0x86187) 
```
 Local_48 is in hexadecimals and right beside to it there is Decimal number:**549255**
```
giorgi@darkfears31:~/Desktop$ ./bbbbloat 
What's my favorite number? 549255
picoCTF{cu7_7h3_bl047_36dd316a}
```
