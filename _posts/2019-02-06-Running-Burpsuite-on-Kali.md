---
layout: post
title: Running Burp suite on Kali linux 2018.4
categories: Burp
---

## Environment
* OS: Kali linux 2018.4
{% highlight shell %}
root@kali:# apt-get update 
root@kali:# apt-get upgrade
root@kali:# uname -a
Linux kali 4.18.0-kali3-amd64 #1 SMP Debian 4.18.20-2kali2 (2018-11-30) x86_64 GNU/Linux
root@kali:~# date
Wed Feb  6 19:13:54 EET 2019
{% endhighlight %}
* Burp Suite: Burp Suite Community Edition v1.7.36


## Explanation
By default setting, it is not capable to run Burp Suite on Kali linux 2018.4 because it uses JRE 10.0.2.

## Solution
### 1. Launch Burp Suite
As we try to launch the Burp Suite,
we have this error.
{% highlight shell %}
root@kali:~# burpsuite
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by burp.uie (file:/usr/bin/burpsuite) to field javax.crypto.JceSecurity.isRestricted
WARNING: Please consider reporting this to the maintainers of burp.uie
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
Your JRE appears to be version 10.0.2 from Oracle Corporation
Burp has not been fully tested on this platform and you may experience problems.
java.lang.IllegalArgumentException: Window must not be zero
{% endhighlight %}

### changing Java version
According to <a href="https://support.portswigger.net/customer/portal/questions/17360581-burp-suite-won-t-start-at-all-with-java-1-">this website</a>, Burp works best with Java 8.
Since Openjdk8 is package candidate in the Kali Linux official repository, we can take advantage of that.
{% highlight shell %}
apt-get install openjdk-8-jdk openjdk-8-jre
{% endhighlight %}
We can change the version of Java with "update-alternatives".
{% highlight shell %}
root@kali:# update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-10-openjdk-amd64/bin/java      1101      auto mode
  1            /usr/lib/jvm/java-10-openjdk-amd64/bin/java      1101      manual mode
  2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number: 
{% endhighlight %}
What we need to do is just type "2" and Enter.
{% highlight shell %}
root@kali:# java -version
openjdk version "1.8.0_171"
OpenJDK Runtime Environment (build 1.8.0_171-8u171-b11-2-b11)
OpenJDK 64-Bit Server VM (build 25.171-b11, mixed mode)
{% endhighlight %}
After that, we can confirm that Burp is working correctly.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-02-06/2019-02-06-20-22-10.png)
