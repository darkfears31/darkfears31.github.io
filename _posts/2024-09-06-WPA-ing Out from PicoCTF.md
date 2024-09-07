[page](https://play.picoctf.org/practice/challenge/237?category=4&difficulty=2&page=2&solved=0)
>Description
>I thought that my password was super-secret, but it turns out that passwords passed over the AIR can be CRACKED, especially if I used the same wireless network password as one in the rockyou.txt credential dump.Use this '[pcap file](https://artifacts.picoctf.net/c/41/wpa-ing_out.pcap)' and the rockyou wordlist. The flag should be entered in the picoCTF{XXXXXX} format.

>Hints
>Aircrack-ng can make a pcap file catch big air...and crack a password.

Download the `.pcap` file and i cant find anything. Then from Hints i went to `aircrack-ng GUI` which i downloaded from [Github](https://github.com/t-gitt/aircrack-ng-gui). I opened It chose the `.pcap` file and then `rockyou.txt` and i let the program cook. After 1 sec there was the password:
![](/assets/images/Screenshot from 2024-07-19 21-34-50.png)
And the flag is:
**picoCTF{mickeymouse}**
