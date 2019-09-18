---
layout: post
title: Hackthebox Bounty Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-20/bounty-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a machine "Bounty" on that website.<br>

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.93 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-18 11:58 EEST
Nmap scan report for 10.10.10.93
Host is up (0.040s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 147.03 seconds
{% endhighlight %}

Gobuster port 80:
{% highlight shell %}
{% endhighlight %}

### 2. Getting User

### 3. Getting Root


