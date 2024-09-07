---
title: "Pixelated from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Pixelated from PicoCTF
[page](https://play.picoctf.org/practice/challenge/100?page=14)
>Description
I have these 2 images, can you make a flag out of them? [scrambled1.png](https://mercury.picoctf.net/static/1594c3f1980e3fb93df09a6d02f53904/scrambled1.png) [scrambled2.png](https://mercury.picoctf.net/static/1594c3f1980e3fb93df09a6d02f53904/scrambled2.png)

This is Demonstration of Visual Cryptography.
![](/assets/images/Visual_crypto_animation_demo.gif)
this is two image which came from one image, so we need to join them together, i couldn't find any tool to download or join this two images together so i went to [github and downloaded the code](https://github.com/HHousen/PicoCTF-2021/blob/master/Cryptography/Pixelated/script.py).
```
import numpy as np
from PIL import Image

# Open images
im1 = Image.open("scrambled1.png")
im2 = Image.open("scrambled2.png")

# Make into Numpy arrays
im1np = np.array(im1)
im2np = np.array(im2)

# Add images
result = im2np + im1np
# Convert back to PIL image and save
Image.fromarray(result).save('result.png')
```
***
> Note that the images and this code should be in the same directory.

***
Let's run the code and see the result:
![](/assets/images/Screenshot from 2024-07-13 19-04-31.png)
Flag is:
**picoCTF{1b867c3e}**
