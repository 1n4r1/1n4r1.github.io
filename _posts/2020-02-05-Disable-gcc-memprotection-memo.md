---
layout: post
title: Memo - disable memory protection for gcc
categories: Reversing
---

# Explanation
Memo to enable / disable the memory protection when compiling with gcc.

# Environment
* OS: Kali linux 2019.4

# Solution
### 1. Default

#### C lang Source code
{% highlight shell %}
root@kali:~# cat helloworld.c 
#include <stdio.h>

int main() {
   printf("Hello World!");
   return 0;
}
{% endhighlight %}

#### Compile with no option
{% highlight shell %}
root@kali:~# gcc helloworld.c -o hello_world

root@kali:~# checksec --file hello_world
[*] Checking for new versions of pwntools
    To disable this functionality, set the contents of /root/.pwntools-cache-3.7/update to 'never'.
[*] A newer version of pwntools is available on pypi (4.0.0 --> 4.0.1).
    Update with: $ pip install -U pwntools
[*] '/root/hello_world'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
{% endhighlight %}

#### Compile with stack protection
{% highlight shell %}
root@kali:~# gcc helloworld.c -o hello_world -fstack-protector-all

root@kali:~# checksec --file hello_world 
[*] '/root/hello_world'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
{% endhighlight %}

#### Confirm ASLR working
{% highlight shell %}
root@vagrant:/home/vagrant# echo 1 > /proc/sys/kernel/randomize_va_space

root@vagrant:/home/vagrant# ldd /usr/bin/test
	linux-vdso.so.1 (0x00007ffdf7710000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f2cd0092000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f2cd068f000)

root@vagrant:/home/vagrant# ldd /usr/bin/test
	linux-vdso.so.1 (0x00007ffc56f8d000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f1c23924000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f1c23f21000)

root@vagrant:/home/vagrant# ldd /usr/bin/test
	linux-vdso.so.1 (0x00007ffca45ef000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f702a599000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f702ab96000)
{% endhighlight %}

### 2. Disable memory protections
#### Disable ASLR
{% highlight shell %}
root@vagrant:/home/vagrant# echo 0 > /proc/sys/kernel/randomize_va_space

root@vagrant:/home/vagrant# ldd /usr/bin/test
	linux-vdso.so.1 (0x00007ffff7ffa000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ffff77d8000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ffff7dd5000)

root@vagrant:/home/vagrant# ldd /usr/bin/test
	linux-vdso.so.1 (0x00007ffff7ffa000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ffff77d8000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ffff7dd5000)

root@vagrant:/home/vagrant# ldd /usr/bin/test
	linux-vdso.so.1 (0x00007ffff7ffa000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ffff77d8000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ffff7dd5000)
{% endhighlight %}

#### Disable canary
{% highlight shell %}
root@kali:~# gcc helloworld.c -o hello_world -fno-stack-protector

root@kali:~# checksec --file hello_world 
[*] '/root/hello_world'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
{% endhighlight %}

#### Disable DEP
{% highlight shell %}
oot@kali:~# gcc helloworld.c -o hello_world -z execstack

root@kali:~# checksec --file hello_world 
[*] '/root/hello_world'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      PIE enabled
    RWX:      Has RWX segments
{% endhighlight %}

#### Disable PIE
{% highlight shell %}
root@kali:~# gcc helloworld.c -o hello_world -no-pie

root@kali:~# checksec --file hello_world 
[*] '/root/hello_world'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
root@kali:~# 
{% endhighlight %}

#### Disable ALL
{% highlight shell %}
root@kali:~# gcc helloworld.c -o hello_world -fno-stack-protector -no-pie -z execstack

root@kali:~# checksec --file hello_world 
[*] '/root/hello_world'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)
    RWX:      Has RWX segments
{% endhighlight %}
