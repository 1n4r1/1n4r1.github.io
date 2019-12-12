---
layout: post
title: Burp interception for localhost application with FoxyProxy
categories: Burp
---

# Explanation
To use Burp interception just for the localhost application, install browser extension <a href="https://getfoxyproxy.org/">"FoxyProxy"</a><br>
This time, "FoxyProxy" was used for OWASP Juice Shop on the localhost:3000.

# Environment
* OS: Kali linux 2019.4
* Burp Suite: Burp Suite Community Edition v2.1.04
* Browser: Google Chrome 78.0.3904.108 (Official Build) (64-bit)
* FoxyProxy: FoxyProxy Standard 3.0.7.1 

# Solution

## 1. Installing FoxyProxy

At first, go to the Chrome webstore and install Chrome extension FoxyProxy.
![placeholder](https://inar1.github.io/public/images/2019-12-12/2019-12-11-23-36-55.png)
<br>

## 2. Launch Burp Suite

Next, launch Burp Suite.<br>
This time, default setting (IP: 127.0.0.1, port: 8080) was used.
![placeholder](https://inar1.github.io/public/images/2019-12-12/2019-12-12-01-27-55.png)
<br>

## 3. Edit "/etc/hosts"

Add following line to the "/etc/hosts" to give an additional name "juice-shop" for localhost.<br>
(We can't use the name "localhost" for this purpose.)
{% highlight shell %}
127.0.0.1 juice-shop
{% endhighlight %}
<br>

## 4. Setup FoxyProxy

Then, open the extension icon on the right of Chrome header and select "options"<br>
Click "add New Proxy" and open the "Proxy settings" window.<br>
Go to "Proxy Details" and set configuration for running Burp Suite.
![placeholder](https://inar1.github.io/public/images/2019-12-12/2019-12-12-00-03-48.png)

Next, go to "URL Patterns" and "Add new pattern".<br>
We can use Wildcard for the domain name for the proxy.
![placeholder](https://inar1.github.io/public/images/2019-12-12/2019-12-12-01-24-17.png)

Finally, go to top page of FoxyProxy configuration and enable the proxy configuration.<br>
Choose "Use proxies based on their pre-defined patterns and priorities" as Proxy mode.
![placeholder](https://inar1.github.io/public/images/2019-12-12/2019-12-12-00-08-47.png)
<br>

## 4. Check the configuration with Burp Suite.

Access "http://sec-juice:3000" with Google Chrome and take a look at "HTTP history" tab on Burp Suite.<br>
We can confirm that we can analyze the traffic to localhost.
![placeholder](https://inar1.github.io/public/images/2019-12-12/2019-12-12-01-23-33.png)
