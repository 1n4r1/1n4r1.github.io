---
layout: post
title: Running OWASP Juice Shop on Docker on Kali linux 2019.4
categories: OWASP
---
## Explanation
OWASP Juice Shop is a vulnerable web application which contains bunch of vulnerabilities in the <a href="https://www.owasp.org/index.php/Top_10-2017_Top_10">OWASP TOP 10</a>.<br>
This is the explanation of environment creation for OWASP Juice Shop with Docker.

## Environment
* OS: Kali linux 2019.4
* Docker: 19.03.4

## Solution

### 1. Installing prerequisits
{% highlight shell %}
root@kali:~# apt-get install docker.io docker-compose

---

root@kali:~# docker pull bkimminich/juice-shop

---
{% endhighlight %}

### 2. Launch OWASP Juice Shop

With the following command, we can launch OWASP Juice Shop on Docker.
{% highlight shell %}
root@kali:~# docker run --rm -p 3000:3000 bkimminich/juice-shop

> juice-shop@9.3.0 start /juice-shop
> node app

info: All dependencies in ./package.json are satisfied (OK)
info: Detected Node.js version v12.13.1 (OK)
info: Detected OS linux (OK)
info: Detected CPU x64 (OK)
info: Required file index.html is present (OK)
info: Required file styles.css is present (OK)
info: Required file main-es2015.js is present (OK)
info: Required file polyfills-es2015.js is present (OK)
info: Required file runtime-es2015.js is present (OK)
info: Required file vendor-es2015.js is present (OK)
info: Required file main-es5.js is present (OK)
info: Required file polyfills-es5.js is present (OK)
info: Required file runtime-es5.js is present (OK)
info: Required file vendor-es5.js is present (OK)
info: Configuration default validated (OK)
info: Port 3000 is available (OK)
info: Server listening on port 3000

{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2019-12-11/2019-12-11-13-35-40.png)

### 3. Version confirmation
{% highlight shell %}
root@kali:~# lsb_release -a
No LSB modules are available.
Distributor ID:	Kali
Description:	Kali GNU/Linux Rolling
Release:	2019.4
Codename:	kali-rolling
root@kali:~# 

root@kali:~# docker --version
Docker version 19.03.4, build 9013bf5
root@kali:~#
{% endhighlight %}
