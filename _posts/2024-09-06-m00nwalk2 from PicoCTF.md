[page](https://play.picoctf.org/practice/challenge/28?category=4&difficulty=2&page=3&search=&solved=0)
>Description
>Revisit the last transmission. We think this [transmission](https://jupiter.challenges.picoctf.org/static/599404f0bf7426a5a5c2deb538860cda/message.wav) contains a hidden message. There are also some clues [clue 1](https://jupiter.challenges.picoctf.org/static/599404f0bf7426a5a5c2deb538860cda/clue1.wav), [clue 2](https://jupiter.challenges.picoctf.org/static/599404f0bf7426a5a5c2deb538860cda/clue2.wav), [clue 3](https://jupiter.challenges.picoctf.org/static/599404f0bf7426a5a5c2deb538860cda/clue3.wav).

>Hints
>Use the clues to extract the another flag from the .wav file

So everything is the same except the `mode` its not `scottie 1` anymore its `auto`, that took me a while.
After that i tested all the `.wav`files and got all the `.png`from them:
![](/assets/images/message.png)![](/assets/images/clue1.png)![](/assets/images/clue2.png)![](/assets/images/clue3.png)

So something is hidden in the `message.wav`. Password is `hidden_stegosaurus`, some quote and `Alan Eliasen The FutureBoy`. I searched that and found the site which contained `steganography`tools. Then i went to terminal and used steghide on the `message.wav`with password provided:
```
$ steghide extract -sf message.wav -p hidden_stegosaurus
wrote extracted data to "steganopayload12154.txt".
$ cat steganopayload12154.txt
picoCTF{the_answer_lies_hidden_in_plain_sight}
```
And there is the flag:
**picoCTF{the_answer_lies_hidden_in_plain_sight}**
