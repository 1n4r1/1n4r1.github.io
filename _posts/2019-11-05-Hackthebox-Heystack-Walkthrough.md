---
layout: post
title: Hackthebox Heystack Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-11-05/heystack-badge.png)
# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Heystack".<br>

# Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
  root@kali:~# nmap -p- 10.10.10.115 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-04 15:42 EET
Nmap scan report for 10.10.10.115
Host is up (0.047s latency).
Not shown: 65532 filtered ports
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
| 2048 2a:8d:e2:92:8b:14:b6:3f:e4:2f:3a:47:43:23:8b:2b (RSA)
| 256 e7:5a:3a:97:8e:8e:72:87:69:a3:0d:d1:00:bc:1f:09 (ECDSA)
|_ 256 01:d2:59:b2:66:0a:97:49:20:5f:1c:84:eb:81:ed:95 (ED25519)
80/tcp open http nginx 1.12.2
|_http-server-header: nginx/1.12.2
|_http-title: Site doesn't have a title (text/html).
9200/tcp open http nginx 1.12.2
| http-methods:
|_ Potentially risky methods: DELETE
|_http-server-header: nginx/1.12.2
|_http-title: Site doesn't have a title (application/json; charset=UTF-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 161.51 seconds
{% endhighlight %}

HTTP enumeration:<br>
Sounds like only one page with heystack image available.
![placeholder](https://inar1.github.io/public/images/2019-11-05/heystack-badge.png)
{% highlight shell %}
{% endhighlight %}

Port 9200 enumeration:<br>
We can figure out that Elasticsearch is running.
![placeholder](https://inar1.github.io/public/images/2019-11-05/heystack-badge.png)
{% highlight shell %}

{% endhighlight %}

### 2. Getting User

At first, take a look at Elasticsearch.



Next, go back to port 80 again.<br>
We have only one file "needle.jpg" there.<br>
To check if something interesting there, we can use "strings" command.
{% highlight shell %}

{% endhighlight %}

We found base64 encoded data.<br>
Then, decode the message. We get a message with unknown language.
{% highlight shell %}

{% endhighlight %}

Google is always our friend. Translate the message.
![placeholder](https://inar1.github.io/public/images/2019-11-05/heystack-badge.png)



### 3. Getting Root



