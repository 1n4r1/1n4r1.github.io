---
layout: post
title: Hackthebox Valentine Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-15/valentine.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Valentine`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.79 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-12 15:20 EEST
Nmap scan report for 10.10.10.79
Host is up (0.044s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2020-04-12T12:25:15+00:00; +4m06s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 4m05s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.65 seconds
```

### Adding entry to `/etc/hosts`
```shell
root@kali:~# cat /etc/hosts | grep valentine
10.10.10.79 valentine.htb
```

### Gobuster HTTP:
```shell
root@kali:~# gobuster dir -u http://10.10.10.79 -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.79
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/04/12 15:28:25 Starting gobuster
===============================================================
/.hta (Status: 403)
/.hta.php (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/cgi-bin/ (Status: 403)
/decode (Status: 200)
/decode.php (Status: 200)
/dev (Status: 301)
/encode (Status: 200)
/encode.php (Status: 200)
/index.php (Status: 200)
/index (Status: 200)
/index.php (Status: 200)
/server-status (Status: 403)
===============================================================
2020/04/12 15:29:05 Finished
===============================================================
```

### Gobuster HTTPS:
```shell
root@kali:~# gobuster dir -u https://valentine.htb -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php -k
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://valentine.htb
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/04/12 15:30:16 Starting gobuster
===============================================================
/.hta (Status: 403)
/.hta.php (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/cgi-bin/ (Status: 403)
/decode (Status: 200)
/decode.php (Status: 200)
/dev (Status: 301)
/encode (Status: 200)
/encode.php (Status: 200)
/index (Status: 200)
/index.php (Status: 200)
/index.php (Status: 200)
/server-status (Status: 403)
===============================================================
2020/04/12 15:30:56 Finished
===============================================================
```

## 2. Getting User
For the initial foothold, we need some guessing.<br>
On the top page, we have an image an woman with `bleeding heart`.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-15/omg.png)

Then, try to check if this has <a href="https://heartbleed.com/">Heartbleed</a>.<br>
We can confirm that this server is vulnerable.
```shell
root@kali:~# nmap -p 443 --script ssl-heartbleed valentine.htb
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-12 15:42 EEST
Nmap scan report for valentine.htb (10.10.10.79)
Host is up (0.042s latency).

PORT    STATE SERVICE
443/tcp open  https
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
|       http://www.openssl.org/news/secadv_20140407.txt 
|_      http://cvedetails.com/cve/2014-0160/

Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
```

Then, try `searchsploit`.
```shell
root@kali:~# searchsploit heartbleed
-------------------------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                                          |  Path
                                                                                                        | (/usr/share/exploitdb/)
-------------------------------------------------------------------------------------------------------- ----------------------------------------
OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure (Multiple SSL/TLS Versions)     | exploits/multiple/remote/32764.py
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (1)                                     | exploits/multiple/remote/32791.c
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (2) (DTLS Support)                      | exploits/multiple/remote/32998.c
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure                                        | exploits/multiple/remote/32745.py
-------------------------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

This time, `OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure` has been used.<br>
We have an interesting parameter `$text=`.
```shell

```

Decode the base64 text.
```shell

```


## 3. Getting Root

