---
layout: post
title: Hackthebox Celestial Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-22/celestial-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a machine "Celestial" on that website.<br>

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.85 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-22 11:46 EEST
Nmap scan report for 10.10.10.85
Host is up (0.039s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2743.74 seconds
{% endhighlight %}

gobuster port 3000:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.85:3000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .js
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.85:3000
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     js
[+] Timeout:        10s
===============================================================
2019/09/23 00:13:30 Starting gobuster
===============================================================
===============================================================
2019/09/23 00:43:06 Finished
===============================================================
{% endhighlight %}

### 2. Getting User




### 3. Getting Root


