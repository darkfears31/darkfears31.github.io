---
title: "Chrono from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Chrono from PicoCTF
[page](https://play.picoctf.org/practice/challenge/347?category=5&difficulty=2&page=1&search=&solved=0)
>Description
>How to automate tasks to run at intervals on linux servers?

How to automate tasks to run at intervals on linux servers?
|
To automate tasks to run at intervals on Linux servers, you can use cron jobs. Cron is a time-based job scheduler in Unix-like operating systems. It allows users to schedule scripts or commands to run automatically at specified times and intervals.

Here's a step-by-step guide on how to set up a cron job:
1. **Open the Crontab File:** Open your cron table (crontab) file with the following command. This will open the crontab file in the default text editor.

Let's head to `/` directory and `cd challenge` but i can't access it so let's search `crontab` as hinted in `description` 
I found readable file in `/etc` directory so i used `cat` ono it:
```
cat /etc/crontab
```
I got the flag:
**picoCTF{Sch3DUL7NG_T45K3_L1NUX_1b4d8744}**
