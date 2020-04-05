---
layout: post
title: Reversing.kr Easy Unpack Writeup
categories: Reversing
---

## Environment
* Host OS: Kali linux 2018.4
* Guest OS: Windows 7 Service Pack 1
* Virtualization: Virtualbox 5.2.22 
* PE packer/analyzer: PEiD v0.95

## Explanation
<a href="http://reversing.kr">Reversing.kr</a> is a website which has some of reverse engineering challenges.
This is a write-up of Easy Unpack on that website.


## Solution
### 1. Reading the ReadMe.txt
As we open the readme.txt, what we can see is following message.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-29/2018-12-29-11-09-39.png)
Sounds like we have to figure out which address is the Original Entry Point.

### 2. Running the app
When we run the app, we have a small dialogue Clicking does not work for anything
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-29/2018-12-28-20-39-16.png)

### 3. Finding an entry point
We can find the OEP easily with a software "PEiD".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-29/2018-12-29-11-51-21.png)
After opened Easy_UnpackMe.exe with "PEiD", click "->" button to open the menu.
Then, proceed like
{% highlight shell %}
Plugins -> PEiD Generic Unpacker
{% endhighlight %}
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-29/2018-12-29-11-55-18.png)
With this plugin, we can find the OEP.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2018-12-29/2018-12-29-12-01-01.png)
According to this information, the key of this challange is "00401150".

