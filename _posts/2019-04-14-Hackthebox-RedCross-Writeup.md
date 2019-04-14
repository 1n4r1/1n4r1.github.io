---
layout: post
title: Hackthebox RedCross Writeup
categories: HackTheBox
---

<img src="/public/images/2019-04-14/redcross_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "RedCross" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -sC -sV 10.10.10.113
Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-13 19:17 EEST
Nmap scan report for intra.redcross.htb (10.10.10.113)
Host is up (0.035s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey: 
|   2048 67:d3:85:f8:ee:b8:06:23:59:d7:75:8e:a2:37:d0:a6 (RSA)
|   256 89:b4:65:27:1f:93:72:1a:bc:e3:22:70:90:db:35:96 (ECDSA)
|_  256 66:bd:a1:1c:32:74:32:e2:e6:64:e8:a5:25:1b:4d:67 (ED25519)
80/tcp  open  http     Apache httpd 2.4.25
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Did not follow redirect to https://intra.redcross.htb/
443/tcp open  ssl/http Apache httpd 2.4.25
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.25 (Debian)
| http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_Requested resource was /?page=login
| ssl-cert: Subject: commonName=intra.redcross.htb/organizationName=Red Cross International/stateOrProvinceName=NY/countryName=US
| Not valid before: 2018-06-03T19:46:58
|_Not valid after:  2021-02-27T19:46:58
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
|   http/1.1
|   http/1.1

~~~

|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|_  http/1.1
Service Info: Host: redcross.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4351.20 seconds
{% endhighlight %}

Since http/s access of 10.10.10.113 redirects to "https://intra.redcross.htb", we have to add following line to "/etc/hosts"
{% highlight shell %}
10.10.10.113 intra.redcross.htb
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u https://intra.redcross.htb/

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : https://intra.redcross.htb/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/04/13 20:27:10 Starting gobuster
=====================================================
/images (Status: 301)
/pages (Status: 301)
/documentation (Status: 301)
/javascript (Status: 301)
/server-status (Status: 403)
=====================================================
2019/04/13 20:40:54 Finished
=====================================================
{% endhighlight %}

Gobuster HTTP "/documentation":
{% highlight shell %}
root@kali:~# gobuster -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u https://intra.redcross.htb/documentation/ -x .doc,.pdf

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : https://intra.redcross.htb/documentation/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : doc,pdf
[+] Timeout      : 10s
=====================================================
2019/04/13 21:06:57 Starting gobuster
=====================================================
/account-signup.pdf (Status: 200)
=====================================================
2019/04/13 21:47:36 Finished
=====================================================
{% endhighlight %}


### 2. Getting User
We can find a login console on the top page.
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-13-19-37-17.png)

Besides, we can find an interesting pdf under the directory "/documents"
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-13-22-03-30.png)

By sending following message, we can create a new credential "guest:guest".
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-03-24-15-20-11.png)

We can login to the console with a credential "guest:guest".
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-03-24-15-20-11.png)

{% highlight shell %}
10.10.10.113 admin.redcross.htb
{% endhighlight %}

If we put a single quote in a UserID and submit, we receive followin message.<br>
This means this webapp has SQLinjection vulnerability.
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-03-24-15-20-11.png)

In this case, the url we are redirected is following.
{% highlight shell %}
https://intra.redcross.htb/?o=%27&page=app
{% endhighlgiht %}

Now we have following query and we have to put something into single quote.
{% highlight shell %}
or dest like ''
{% endhighlight %}

We can put % there and we can achieve following output.
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-22-23-51.png)

Sounds like we have admin webapp and we have sub domain for that.<br>
Add following line in "/etc/hosts" and try to access.
{% highlight shell %}
10.10.10.113 admin.redcross.htb
{% endhighlight %}

We can find another login console.
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-03-24-15-20-11.png)

we can try the credential "guest:guest". However, it shows a message we don't have enough privilege.
Then, try to do session replay attack.<br>
Open Burp Suite and check the "PHPSESSID" in the Cookie when we accessed "intra.redcross.htb".
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-03-24-15-20-11.png)

Then, turn intercept on and try to access "admin.redcross.htb".<br>
check the value of "PHPSESSID" in the cookie and change the value to the above session id.
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-03-24-15-20-11.png)





We cam use this credential for ssh login.
{% highlight shell %}
manolo:NIFmPpCb
{% endhighlight %}
