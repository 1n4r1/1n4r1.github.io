---
layout: post
title: Hackthebox Vault Writeup
categories: HackTheBox
---

<img src="/public/images/2019-04-07/vault_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Vault" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.109 -sV -sC 
Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-05 15:02 EET
Nmap scan report for 10.10.10.109
Host is up (0.036s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a6:9d:0f:7d:73:75:bb:a8:94:0a:b7:e3:fe:1f:24:f4 (RSA)
|   256 2c:7c:34:eb:3a:eb:04:03:ac:48:28:54:09:74:3d:27 (ECDSA)
|_  256 98:42:5f:ad:87:22:92:6d:72:e6:66:6c:82:c1:09:83 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.36 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/ -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/03/05 15:06:18 Starting gobuster
=====================================================
/index.php (Status: 200)
/server-status (Status: 403)
=====================================================
2019/03/05 15:35:07 Finished
=====================================================
{% endhighlight %}

In index.php, there is a message "We are proud to announce our first client: Sparklays (Sparklays.com still under construction)".<br>
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-13-29-39.png)
Try to access to /sparklays.

Gobuster HTTP /sparklays:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/sparklays -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/sparklays/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/03/05 16:00:28 Starting gobuster
=====================================================
/login.php (Status: 200)
/admin.php (Status: 200)
/design (Status: 301)
=====================================================
2019/03/05 16:29:17 Finished
=====================================================
{% endhighlight %}

Gobuster HTTP /sparklays/design:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/sparklays/design -x .php,.html

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/sparklays/design/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php,html
[+] Timeout      : 10s
=====================================================
2019/03/06 14:13:00 Starting gobuster
=====================================================
/uploads (Status: 301)
/design.html (Status: 200)
=====================================================
2019/03/06 14:58:17 Finished
=====================================================
{% endhighlight %}

### 2. Getting User

### 3. Getting Root


