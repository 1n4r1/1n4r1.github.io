---
layout: post
title: Hackthebox Haystack Walkthrough
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
{% highlight shell %}
root@kali:~# curl http://10.10.10.115:9200/
{
  "name" : "iQEYHgS",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "pjrX7V_gSFmJY-DxP4tCQg",
  "version" : {
    "number" : "6.4.2",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "04711c2",
    "build_date" : "2018-09-26T13:34:09.098244Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
{% endhighlight %}

### 2. Getting User

At first, take a look at port 80<br>.
We have only one file "needle.jpg" there.<br>
{% highlight html %}
root@kali:~# curl http://10.10.10.115
<html>
<body>
<img src="needle.jpg" />
</body>
</html>
{% endhighlight %}

To check if something interesting there, we can use "strings" command.
{% highlight shell %}
root@kali:~# strings needle.jpg 
JFIF
Exif
paint.net 4.1.1
UNICODE
$3br
%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
	#3R
&'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
sc,x

~~~

bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg==
{% endhighlight %}

We found base64 encoded data.<br>
Then, decode the message. We get a message with unknown language.
{% highlight shell %}
root@kali:~# echo 'bGEgYWd1amEgZW4gZWwgcGFqYXIgZXMgImNsYXZlIg==' | base64 -d
la aguja en el pajar es "clave"
{% endhighlight %}

Google is always our friend. Translate the message.
![placeholder](https://inar1.github.io/public/images/2019-11-05/heystack-badge.png)
{% highlight shell %}
la aguja en el pajar es "clave"

->
the needle in the haystack is "key"
{% endhighlight %}

Then, go back to elasticsearch.


To send a query to elasticsearch, we need a parameter "q".
{% highlight shell %}
root@kali:~# curl http://10.10.10.115:9200/_search?q=clave
{"took":136,"timed_out":false,"_shards":{"total":11,"successful":11,"skipped":0,"failed":0},"hits":{"total":2,"max_score":5.9335938,"hits":[{"_index":"quotes","_type":"quote","_id":"45","_score":5.9335938,"_source":{"quote":"Tengo que guardar la clave para la maquina: dXNlcjogc2VjdXJpdHkg "}},{"_index":"quotes","_type":"quote","_id":"111","_score":5.3459888,"_source":{"quote":"Esta clave no se puede perder, la guardo aca: cGFzczogc3BhbmlzaC5pcy5rZXk="}}]}}
{% endhighlight %}

We found other information. Translate it again.<br>
We got a credential for user "security" to login with ssh.
{% highlight shell%}
root@kali:~# ssh security@10.10.10.115
security@10.10.10.115's password: 
Last login: Tue Nov  5 14:44:15 2019 from 10.10.14.13
[security@haystack ~]$ id
uid=1000(security) gid=1000(security) groups=1000(security) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
{% endhighlight %}

As always, user.txt is in the home directory.
{% highlight shell %}
[security@haystack ~]$ ls -la
total 16
drwx------. 2 security security  99 Feb  6  2019 .
drwxr-xr-x. 3 root     root      22 Nov 28  2018 ..
lrwxrwxrwx. 1 root     root       9 Jan 25  2019 .bash_history -> /dev/null
-rw-r--r--. 1 security security  18 Apr 10  2018 .bash_logout
-rw-r--r--. 1 security security 193 Apr 10  2018 .bash_profile
-rw-r--r--. 1 security security 231 Apr 10  2018 .bashrc
-rw-r--r--. 1 security security  33 Feb  6  2019 user.txt

[security@haystack ~]$ cat user.txt 
04d18bc79dac1d4d48ee0a940c8eb929
{% endhighlight %}

### 3. Getting Root

Since we can't use netstat, use "ss" command.<br>
We have one interesting port 5601.
1. "-4": to use IPv4
2. "-l": listing all listening ports
3. "-n": display port numbers
{% highlight shell %}
[security@haystack ~]$ ss -4 -l -n
Netid  State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
udp    UNCONN     0      0      127.0.0.1:323                        *:*                  
tcp    LISTEN     0      128            *:80                         *:*                  
tcp    LISTEN     0      128            *:9200                       *:*                  
tcp    LISTEN     0      128            *:22                         *:*                  
tcp    LISTEN     0      128    127.0.0.1:5601                       *:*        
{% endhighlight %}

To access the "127.0.0.1:5601" from our localhost, we need port forwarding.<br>
We can find "Kibana" which is date visualization UI used with Elasticsearch.
{% highlight shell %}
ssh -L 5601:127.0.0.1:5601 security@10.10.10.115 -N
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-11-05/heystack-badge.png)

By clicking the "Management" tab, we can figure out that the version of Kibana is "6.4.2"
