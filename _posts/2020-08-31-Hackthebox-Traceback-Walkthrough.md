---
layout: post
title: Hackthebox Traceback Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-31/traceback.png)

# Explanation
[Hackthebox](https://www.hackthebox.eu/) is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Traceback`.

# Solution
## 1. Initial Enumeration
#### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.181 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-30 16:38 JST
Nmap scan report for 10.10.10.181
Host is up (0.23s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 742.50 seconds
```

## 2. Getting User


## 3. Getting Root

