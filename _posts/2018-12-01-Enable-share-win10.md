---
layout: post
title: Enabling Share Folder on Windows 10
categories: Windows
---

## Environment
* Host OS: Kali linux 2018.3
* Guest OS: Windows 10 Enterprise Evaluation (Virtualbox)
* Virtualization: Virtualbox 5.2.20_Debian

## Problem
How to enable Share Folder on Windows 10 with Virtualbox 5 ?

## Solution
1. Configuring share folder on virtualbox
From <a href="https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/">this web site</a>, we can download the official Windows 10 virtualbox .ova file. After the importing of the win10 box, we can configure the share folder on virtualbox console 
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2018-12-01/2018-12-05-09-53-13.png)

2. Run windows 10
With button "start", we can run the virtual machine

3. Open "Network & Internet"
From "Settings", we can open the "Network & Internet" tab.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2018-12-01/2018-12-05-09-49-17.png)

4. Open "Sharing options"
By clicking the "Status" from "Network & Internet",
we can go to the "Sharing options" window.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2018-12-01/2018-12-05-09-50-24.png)

As we can see, there are 2 option buttons and by changing the value of that
we can enable the Share Folder.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2018-12-01/2018-12-05-09-51-13.png)

After the saving the configuration, we can browse the network share folder.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2018-12-01/2018-12-05-17-24-14.png)
