---
layout: post
title: Hackthebox RedCross Writeup
categories: HackTheBox
---

<img src="/public/images/2019-04-14/redcross_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "RedCross" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.113
Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-04 23:27 EEST
Nmap scan report for 10.10.10.113
Host is up (0.040s latency).
Not shown: 65532 filtered ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 105.46 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~#  gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.113/

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.113/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/04/07 13:42:34 Starting gobuster
=====================================================
2019/04/07 13:42:34 [-] Wildcard response found: http://10.10.10.113/c716bd3c-9130-43dc-a6f1-8a9f13bdd561 => 301
2019/04/07 13:42:34 [!] To force processing of Wildcard responses, specify the '-fw' switch.
=====================================================
2019/04/07 13:42:34 Finished
=====================================================
{% endhighlight %}

### 2. Getting User


### 3. Getting Root


