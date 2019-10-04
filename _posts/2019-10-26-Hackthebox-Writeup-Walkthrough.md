---
layout: post
title: Hackthebox Writeup Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-10-26/writeup-badge.png)
# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Writeup".<br>

#### Complation: 48th / 131 boxes

# Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.138 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-02 15:17 EEST
Nmap scan report for 10.10.10.138
Host is up (0.039s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Nothing here yet.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.42 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.138/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.138/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/10/04 16:57:01 Starting gobuster
===============================================================
[ERROR] 2019/10/04 16:57:07 [!] Get http://10.10.10.138/5: dial tcp 10.10.10.138:80: connect: connection refused
[ERROR] 2019/10/04 16:57:09 [!] Get http://10.10.10.138/6: dial tcp 10.10.10.138:80: connect: connection refused
Progress: 107 / 220561 (0.05%)^C
[!] Keyboard interrupt detected, terminating.
===============================================================
2019/10/04 16:57:10 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

Sounds like we can't use gobuster for this box.<br>
At first, try to look at the web page.
![placeholder](https://inar1.github.io/public/images/2019-10-26/writeup-badge.png)

There was nothing here. Next, look at the path which is specified in "http-robots.txt"<br>
We can find a web page which "CMS Made Simple" is used.
![placeholder](https://inar1.github.io/public/images/2019-10-26/writeup-badge.png)
![placeholder](https://inar1.github.io/public/images/2019-10-26/writeup-badge.png)






Try to run the script to crack the password of Simple hogehoge
{% highlight shell %}
[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
[+] Password cracked: raykayjay9
{% endhighlight %}



### 3. Getting Root


