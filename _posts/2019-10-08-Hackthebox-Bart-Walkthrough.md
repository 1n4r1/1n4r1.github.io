---
layout: post
title: Hackthebox Bart Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-10-08/bart-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Bart".<br>

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.81 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-06 15:16 EEST
Nmap scan report for bart.htb (10.10.10.81)
Host is up (0.035s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to http://forum.bart.htb/
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 153.30 seconds
{% endhighlight %}

Checking HTTP header for form.bart.htb:
{% highlight shell %}
root@kali:~# curl http://10.10.10.81 -L --head
HTTP/1.1 302 Found
Content-Length: 0
Content-Type: text/html; charset=UTF-8
Location: http://forum.bart.htb/
Server: Microsoft-IIS/10.0
X-Powered-By: PHP/7.1.7
Date: Sun, 06 Oct 2019 12:24:33 GMT

HTTP/1.1 200 OK
Content-Length: 35529
Content-Type: text/html
Last-Modified: Sun, 04 Feb 2018 12:30:42 GMT
Accept-Ranges: bytes
ETag: "6ef115f9b39dd31:0"
Server: Microsoft-IIS/10.0
Date: Sun, 06 Oct 2019 12:24:33 GMT
{% endhighlight %}

gobuster forum.bart.htb:
{% highlight shell %}
root@kali:~# cat /etc/hosts | grep bart
10.10.10.81 bart.htb forum.bart.htb
{% endhighlight %}
{% highlight shell %}
root@kali:~# gobuster dir -u http://forum.bart.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -x .html,.php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://forum.bart.htb/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php
[+] Timeout:        10s
===============================================================
2019/10/06 15:28:24 Starting gobuster
===============================================================
/index.html (Status: 200)
/Index.html (Status: 200)
/INDEX.html (Status: 200)
===============================================================
2019/10/06 16:14:17 Finished
===============================================================
{% endhighlight %}

Checking HTTP header for bart.htb:
{% highlight shell %}
root@kali:~# curl -i http://bart.htb -L --head
HTTP/1.1 302 Found
Content-Length: 0
Content-Type: text/html; charset=UTF-8
Location: http://forum.bart.htb/
Server: Microsoft-IIS/10.0
X-Powered-By: PHP/7.1.7
Date: Mon, 07 Oct 2019 07:25:20 GMT

HTTP/1.1 200 OK
Content-Length: 35529
Content-Type: text/html
Last-Modified: Sun, 04 Feb 2018 12:30:42 GMT
Accept-Ranges: bytes
ETag: "6ef115f9b39dd31:0"
Server: Microsoft-IIS/10.0
Date: Mon, 07 Oct 2019 07:25:20 GMT
{% endhighlight %}

gobuster bart.htb:
{% highlight shell %}
root@kali:~# gobuster dir -u http://bart.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -x .html,.php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://bart.htb/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php
[+] Timeout:        10s
===============================================================
2019/10/07 10:15:09 Starting gobuster
===============================================================
Error: the server returns a status code that matches the provided options for non existing urls. http://bart.htb/77901071-6f6b-4f8f-bb0b-0f9c2362f66e => 200. To force processing of Wildcard responses, specify the '--wildcard' switch
{% endhighlight %}

Checking non-existing path for bart.htb:<br>
(Figure out 200 means 404 and 301,302 are the existing path)
{% highlight shell %}
root@kali:~# curl http://bart.htb/MysupercoolPath --head
HTTP/1.1 200 OK
Content-Length: 158607
Content-Type: image/jpeg
Last-Modified: Mon, 02 Oct 2017 13:24:19 GMT
Accept-Ranges: bytes
ETag: "8050f5c0813bd31:0"
Server: Microsoft-IIS/10.0
Date: Mon, 07 Oct 2019 07:30:31 GMT
{% endhighlight %}

gobuster bart.htb(discard 200):
{% highlight shell %}
root@kali:~# gobuster dir -u http://bart.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -x .html,.php -s '204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://bart.htb/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,html
[+] Timeout:        10s
===============================================================
2019/10/07 10:17:28 Starting gobuster
===============================================================
/index.php (Status: 302)
/forum (Status: 301)
/Index.php (Status: 302)
/monitor (Status: 301)
/Forum (Status: 301)
/INDEX.php (Status: 302)
/Monitor (Status: 301)
/MONITOR (Status: 301)
===============================================================
2019/10/07 14:21:02 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

By accessing "http://bart.htb/monitor", we can find a login console.
![placeholder](https://inar1.github.io/public/images/2019-10-08/bart-badge.png)

By clicking "Forgot password?" we can go to the password recovery page which we can enumerate users.
![placeholder](https://inar1.github.io/public/images/2019-10-08/bart-badge.png)

In "form.bart.htb", we can find following employees and emails at this company "BART".
{% highlight shell %}
Samantha Brown:s.brown@bart.local
Daniel Simmons:d.simmons@bart.htb
Robert Hilton:r.hilton@bart.htb
Harvey Potter:h.potter@bart.htb  # commented
Daniella Lamborghini: NO DATA
Jane Doe: NO DATA
{% endhighlight %}

By trying some guessing, we can find that the following credential is still valid for the login console.
{% highlight shell %}
harvey:potter
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-10-08/bart-badge.png)

Clicking on the "Internal chat". We can find another subdomain "internal-01.bart.htb"
![placeholder](https://inar1.github.io/public/images/2019-10-08/bart-badge.png)

Add "internal-01.bart.htb" in "/etc/hosts" and access.<br>
We can figure out that <a href="">""</a>
{% highlight shell %}
root@kali:~# cat /etc/hosts | grep bart
10.10.10.81 bart.htb forum.bart.htb internal-01.bart.htb
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-10-08/bart-badge.png)

By runnning again, we can find "Register.php". 
{% highlight shell %}
root@kali:~# gobuster dir -u http://internal-01.bart.htb/simple_chat/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://internal-01.bart.htb/simple_chat/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php
[+] Timeout:        10s
===============================================================
2019/10/08 10:13:19 Starting gobuster
===============================================================
/index.php (Status: 302)
/login.php (Status: 302)
/register.php (Status: 302)
/media (Status: 301)
/chat.php (Status: 302)
/css (Status: 301)
/includes (Status: 301)
/Index.php (Status: 302)
/Login.php (Status: 302)
/js (Status: 301)
/logout.php (Status: 302)
/Media (Status: 301)
/Register.php (Status: 302)
/login_form.php (Status: 200)
/Chat.php (Status: 302)
/INDEX.php (Status: 302)
/CSS (Status: 301)
/JS (Status: 301)
/Logout.php (Status: 302)
/MEDIA (Status: 301)
/Includes (Status: 301)
/LogIn.php (Status: 302)
/LOGIN.php (Status: 302)
===============================================================
2019/10/08 10:57:16 Finished
===============================================================
{% endhighlight %}

By registering, we can log in to the website.
![placeholder](https://inar1.github.io/public/images/2019-10-08/bart-badge.png)

Then, click the "log" link. we can see something like web application log.
![placeholder](https://inar1.github.io/public/images/2019-10-08/bart-badge.png)



### 3. Getting Root



