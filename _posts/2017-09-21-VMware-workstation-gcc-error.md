---
layout: post
title: Fix VMware workstation gcc error
categories: VMware
---


## Environment
OS: Kali linux 2017.2


## Problem
When I finished installing VMware workstation 12.5.7 and tried to run,
I got this error below.

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2017-09-21/gcc-1.png)

However, I have already installed gcc version 6.4.0-5.
Somehow, it can not be found by vmware.


## Solution
Then, I ran the command by root user
> $ sudo vmplayer

After selecting the correct gcc-6 in /usr/bin, we can see this view

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2017-09-21/gcc-2.png)

Then, I succeeded to open vmware console correctly.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2017-09-21/gcc-3.png)
