---
layout: post
title: Reversing.kr Image Prc Writeup
categories: Reversing
---

## Environment
* Host OS: Kali linux 2018.4
* Guest OS: Windows 7 Service Pack 1
* Virtualization: Virtualbox 5.2.22 
* Debugger: IDA Pro Free 5.0
* PE header viewer: PEview 0.9.9
* Hex Editor: HxD 2.0

## Explanation
<a href="http://reversing.kr">Reversing.kr</a> is a website which has some of reverse engineering challenges.
This is a write-up of ImagePrc on that website.


## Solution
### 1. Running the app
When we run the app, we have a small dialogue.
What we can do are just writing something on the bitmap and put the ‘check’ button. When we put check button, another window pops up and shows message ‘wrong’. 
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-27/2018-12-24-17-46-51.png)

### 2. Consideration
It is easy to figure out the point this application creates a bitmap. What we need is we have to look for the point “CreateCompatibleBitmap” Windows API is used. 
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-27/2018-12-25-21-29-39.png)

After sat a breakpoint and run the instruction one by one, we can find a place Loading 2 resources. We can easily determine this because "LoadResource" Windows API is being executed and these resource files are loaded and compared.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-27/2018-12-25-22-09-19.png)

In the specified address, we can see many lines of "FFFF".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-27/2018-12-26-16-27-20.png)
We can assume this is BMP image file included in the exe file. This is because
1. This has to be an image file
2. There are a lot of number of "FF" which means white in bmp file.

### 3. Analyzing PE header
To analyze this exe file, we can use PEview.exe
In section .rsrc, MANIFEST 0065 0412, there is a bmp format image file
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-27/2018-12-27-21-18-57.png)
To see this image file, we have to create a new BMP image whose size is 200x150 and open it with HxD hex editor.
In addition, we have to create a blank BMP image file which has same size.
We have to copy lines from 9060 to the bottom and replace the body of the new BMP file.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-27/2018-12-27-21-26-51.png)
Once we open the modified image file, we can get this image below.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-27/2018-12-27-21-29-33.png)
Now we know the password for this challenge is "GOT".
