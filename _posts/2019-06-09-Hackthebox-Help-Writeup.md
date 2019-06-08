---
layout: post
title: Hackthebox Help Writeup
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-06-09/help_badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Help" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:/home/sabonawa/hackTB# nmap -sV -sC -p- 10.10.10.121
Starting Nmap 7.70 ( https://nmap.org ) at 2019-01-30 08:06 EET
Nmap scan report for 10.10.10.121
Host is up (0.035s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.73 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:/home/sabonawa# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.121

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.121/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/01/30 08:32:04 Starting gobuster
=====================================================
/support (Status: 301)
/javascript (Status: 301)
/server-status (Status: 403)
=====================================================
2019/01/30 08:46:04 Finished
=====================================================
{% endhighlight %}

### 2. Getting User
In /support, we can confirm "HelpdeskZ" is running.
![placeholder](https://inar1.github.io/public/images/2019-06-09/help_badge.png)

By searchsploit, we can find a vulnerability of HelpdeskZ.
{% highlight shell %}
root@kali:~# searchsploit helpdeskz
----------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                           |  Path
                                                                                         | (/usr/share/exploitdb/)
----------------------------------------------------------------------------------------- ----------------------------------------
HelpDeskZ 1.0.2 - Arbitrary File Upload                                                  | exploits/php/webapps/40300.py
HelpDeskZ < 1.0.2 - (Authenticated) SQL Injection / Unauthorized File Download           | exploits/php/webapps/41200.py
----------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

### 3. Getting Root

