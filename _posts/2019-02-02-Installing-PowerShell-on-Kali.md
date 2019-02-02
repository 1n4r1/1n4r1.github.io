---
layout: post
title: Installing PowerShell on Kali linux
---

## Environment
* OS: Kali linux 2018.4
{% highlight shell %}
root@kali:# uname -a
Linux kali 4.18.0-kali3-amd64 #1 SMP Debian 4.18.20-2kali2 (2018-11-30) x86_64 GNU/Linux
{% endhighlight %}
* Powershell: v6.1.2


## Explanation
Installing PowerShell on Kali linux for <a href="https://www.zeropointsecurity.co.uk/rastalabs/">RASTALABS</a>

## Solution
### 1. Installing Dependencies
As we try to install PowerShell according to the official website,
we have this error.
{% highlight shell %}
root@kali:# apt-get install -y powershell
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 powershell : Depends: libssl1.0.0 but it is not installable
E: Unable to correct problems, you have held broken packages.
{% endhighlight %}
However, we do not have the package "libssl1.0.0".
Then, we can download the package from <a href="https://packages.debian.org/jessie/amd64/libssl1.0.0/download">here</a> and install.

{% highlight shell %}
root@kali:# dpkg -i libssl1.0.0_1.0.1t-1+deb8u10_amd64.deb 
Selecting previously unselected package libssl1.0.0:amd64.
dpkg: regarding libssl1.0.0_1.0.1t-1+deb8u10_amd64.deb containing libssl1.0.0:amd64, pre-dependency problem:
 libssl1.0.0 pre-depends on multiarch-support
  multiarch-support is not installed.

dpkg: error processing archive libssl1.0.0_1.0.1t-1+deb8u10_amd64.deb (--install):
 pre-dependency problem - not installing libssl1.0.0:amd64
Errors were encountered while processing:
 libssl1.0.0_1.0.1t-1+deb8u10_amd64.deb
{% endhighlight %}

Sounds like we are still missing package "multiarch-support".
After installing the package, we can install libssl1.0.0 and powershell correctly.

{% highlight shell %}
root@kali:/home/sabonawa/Downloads# sudo apt-get install multiarch-support
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  multiarch-support
0 upgraded, 1 newly installed, 0 to remove and 137 not upgraded.
Need to get 213 kB of archives.
After this operation, 239 kB of additional disk space will be used.
Get:1 http://kali.koyanet.lv/kali kali-rolling/main amd64 multiarch-support amd64 2.28-2 [213 kB]
Fetched 213 kB in 1s (288 kB/s)             
Selecting previously unselected package multiarch-support.
(Reading database ... 424175 files and directories currently installed.)
Preparing to unpack .../multiarch-support_2.28-2_amd64.deb ...
Unpacking multiarch-support (2.28-2) ...
Setting up multiarch-support (2.28-2) ...
needrestart is being skipped since dpkg has failed
{% endhighlight %}

{% highlight shell %}
root@kali:# dpkg -i libssl1.0.0_1.0.1t-1+deb8u10_amd64.deb
root@kali:# apt-get install powershell
{% endhighlight %}

{% highlight shell %}
root@kali:# pwsh
PowerShell 6.1.2
Copyright (c) Microsoft Corporation. All rights reserved.

https://aka.ms/pscore6-docs
Type 'help' to get help.

PS /home/sabonawa/Downloads>
{% endhighlight %}
