---
layout: post
title: Setup memo for GDB Enhanced Features (GEF)
categories: Reversing
---

# Explanation
"GDB Enhanced Features(GEF)" is a GDB extension to provide additional commands for dynamic analysis and exploit development.<br>
To setup for Kali linux, we need some steps and this is a memo for that.

# Environment
* OS: Kali linux 2019.4
* GDB: gdb (Debian 8.3.1-1) 8.3.1

# Solution
### 1. Initial Setup

On the Github repository, we have some Explanation for installation.<br>
However, they didn't work and the following script achieves the purpose.
{% highlight shell %}
root@kali:~# git clone https://github.com/hugsy/gef.git

---

root@kali:~# echo source `pwd`/gef/gef.py >> ~/.gdbinit
{% endhighlight %}

With the following command, we can confirm the GEF running.<br>
However, we still have some missing commands.
{% highlight shell %}
root@kali:~# gdb -q test
GEF for linux ready, type `gef' to start, `gef config' to configure
77 commands loaded for GDB 8.3.1 using Python engine 3.7
[*] 3 commands could not be loaded, run `gef missing` to know why.
Reading symbols from test...
(No debugging symbols found in test)
gef➤  
{% endhighlight %}

With the command "gef missing", we can find the names of command missing.
{% highlight shell %}
gef➤  gef missing
[*] Command `set-permission` is missing, reason  →  Missing `keystone-engine` package for Python3, install with: `pip3 install keystone-engine`.
[*] Command `ropper` is missing, reason  →  Missing `ropper` package for Python3, install with: `pip3 install ropper`.
[*] Command `assemble` is missing, reason  →  Missing `keystone-engine` package for Python3, install with: `pip3 install keystone-engine`.
gef➤  
{% endhighlight %}

Then, install the prerequisites.
{% highlight shell %}
root@kali:~# apt-get install cmake

---

root@kali:~# pip3 install keystone-engine ropper unicorn

---
{% endhighlight %}

However, still some commands are missing.
{% highlight shell %}
gef➤  gef missing
[*] Command `set-permission` is missing, reason  →  Missing `keystone-engine` package for Python3, install with: `pip3 install keystone-engine`.
[*] Command `assemble` is missing, reason  →  Missing `keystone-engine` package for Python3, install with: `pip3 install keystone-engine`.
gef➤  
{% endhighlight %}

This time, we need to build the "keystone" manually.<br>
The source code can be downloaded from the <a href="https://github.com/keystone-engine/keystone/releases">release page of official repository</a>.
{% highlight shell %}
root@kali:~# ls -l | grep keystone
-rw-r--r--  1 root root  4326151 Jan 17 02:30 keystone-0.9.1.tar.gz

root@kali:~# tar xzvf keystone-0.9.1.tar.gz 

---

root@kali:~# cd keystone-0.9.1/
root@kali:~/keystone-0.9.1# mkdir build
root@kali:~/keystone-0.9.1# cd build/
root@kali:~/keystone-0.9.1/build# ../make-share.sh

---

root@kali:~/keystone-0.9.1/build# make install

---

root@kali:~/keystone-0.9.1/build# ldconfig
{% endhighlight %}

After that, we can confirm that we have no missing commands.
{% highlight shell %}
root@kali:~/keystone-0.9.1/build# gdb -q test
GEF for linux ready, type `gef' to start, `gef config' to configure
80 commands loaded for GDB 8.3.1 using Python engine 3.7
Reading symbols from test...
(No debugging symbols found in test)
gef➤  gef missing
[+] No missing command
gef➤ 
{% endhighlight %}
