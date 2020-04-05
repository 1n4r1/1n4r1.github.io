---
layout: post
title: Running OWASP ZAP on Kali Linux 2019.4
categories: OWASP
---

![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/general/zap.jpg)

# Explanation
<a href="https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project">OWASP ZAP</a> is a web vulnerability scanner that is one of the OWASP projects.<br>

# Environment
* OS: Kali linux 2019.4
* OWASP ZAP: v2.8.1 
* Target: OWASP Juice shop v9.3.1

# Solution
### 1. Initial setup
Since it's in the kali official repository, we need just "apt-get"
{% highlight shell %}
root@kali:~# apt-get install zaproxy
{% endhighlight %}

Next, launch the target application.<br>
This time, I used OWASP Juice shop with Vagrant and IP "192.168.33.10" was assigned.
{% highlight shell %}
root@kali:~# git clone https://github.com/bkimminich/juice-shop.git

---

root@kali:~# cd juice-shop/vagrant/

root@kali:~/juice-shop/vagrant# vagrant up

---
{% endhighlight %}

After that give a specific name for OWASP juice shop.<br>
This time, give the following line in "/etc/hosts".
{% highlight shell %}
192.168.33.10 juiceshop
{% endhighlight %}


### 2. Other setup

At first, open the ZAP GUI console.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-05-18-19-49.png)

Go to "Tools" -> "Options" -> "Local Proxies".<br>
By default, it is configured to use "http://localhost:8080".<br>
We have to configure the web browser to use a proxy on port 8080.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2019-12-31-10-50-57.png)

Then, change the mode to the "Protected mode" not to implement unintended attack.<br>
If we select the "Protected mode", we have to specify the target URL.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-01-12-52-45.png)

The web browser we use should be Firefox because it does not have any XSS protection.<br>
However, this time, Google Chrome was used.<br>
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-05-18-37-27.png)

If the configuration is correct, we can find the target URL in the "Site" section.<br>
This means now we can go to the next step.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-05-18-44-26.png)

Since we're using protected mode, we have to include the site into the "context".<br>
We need to right click the site, then go to "Include in Context".<br>
This time, we don't have any context so click the "New Context" and we can see this window.<br>
So click "OK".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-01-12-54-10.png)

After that, we can see that some entries are added to the site.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-01-12-52-45.png)


### 3. Active scanning

Now we have a target machine.<br>
Try to attack by right clicking the "http://juiceshop" -> "Attack" -> "Active Scan".<br>
We can confirm that tons of HTTP requests were being sent on the "Active Scan" tab.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-01-12-54-41.png)
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-01-13-01-32.png)

After the finishing, we can find some security alerts on the "Alerts" tab
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-01-13-06-41.png)


### 4. Saving the session

We can save the session data by going to "Snapshot Session As...", we cam save the current session.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-05-18-47-55.png)


### 5. Generate a report

We can create a report of the each test as HTML or XML file.<br>
Go to "Report" -> "Generate HTML Report...".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-01-06/2020-01-05-18-48-51.png)


### 6. Next task

This time, I could not find some vulnerabilities that OWASP juice shop has.<br>
Next time, try to focus on each vulnerability and by customizing policies, achieve this purpose
