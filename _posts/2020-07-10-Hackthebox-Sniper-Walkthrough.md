---
layout: post
title: Hackthebox Sniper Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Sniper`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.151 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-07 00:55 JST
Nmap scan report for 10.10.10.151
Host is up (0.23s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Sniper Co.
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h03m27s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-07-06T23:04:23
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 385.02 seconds
```

### SMB enumeration:
```shell
root@kali:~# smbmap -H 10.10.10.151
[!] Authentication error on 10.10.10.151
```

### HTTP enumeration:
```shell
root@kali:~# gobuster dir -u http://10.10.10.151 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.151
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/07 09:29:36 Starting gobuster
===============================================================
/images (Status: 301)
/blog (Status: 301)
/user (Status: 301)
/css (Status: 301)
/js (Status: 301)
===============================================================
2020/07/07 10:02:46 Finished
===============================================================
```

## 2. Getting User
There are 2 interesting directories. `/blog` and `/user`.
#### /blog:
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

#### /user:
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

We can easily find a file inclusion vulnerability in the `lang` parameter of `/blog/index.php`
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

We can't execute remote file inclusion via HTTP.<br>
However, we can carry out a remote file inclusion attack via SMB.


## 3. Getting Root

