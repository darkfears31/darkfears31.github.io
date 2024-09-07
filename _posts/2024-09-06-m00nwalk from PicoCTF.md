[page](https://play.picoctf.org/practice/challenge/26?category=4&difficulty=2&page=3&search=&solved=0)
>Description
>Decode this [message](https://jupiter.challenges.picoctf.org/static/d6fcea5e3c6433680ea4f914e24fab61/message.wav) from the moon.

>Hints
>How did pictures from the moon landing get sent back to Earth?
>What is the CMU mascot?, that might help select a RX option

I asked this questions to [chatGPT](https://chatgpt.com/share/190b1c19-3988-4964-8700-14e7ce8f4268) and got the answers:
1.sstv and app is Qsstv
2.Scotty
Install `qsstv` in your OS and then in `mode`choose `scottie 1`from AI answers.
open the recently downloaded `.wav`file and play it, make sure to press `start reciever`in top left corner. Then wait until this picture forms:
![](/assets/images/Screenshot from 2024-07-21 00-36-45.png)
And the is the flag:
**picoCTF{beep_boop_im_in_space}**
