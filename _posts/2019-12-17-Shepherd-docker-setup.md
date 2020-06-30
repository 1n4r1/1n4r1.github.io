---
layout: post
title: Running OWASP Security Shepherd with Docker compose on Kali 2019.4
categories: OWASP
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/general/ss.png)

# Explanation
<a href="https://github.com/OWASP/SecurityShepherd">OWASP Security Shepherd</a> is a vulnerable web application for the practice.<br>
Unlike other vulnerable webapp like DVWA, Juice Shop, WebGoat,
1. has also challenges for mobile app security 
2. focuses on the vulnerability of web application "spec". not like ordinary XSS, SQLi and so on.
3. more focused on learning local proxy(like Burp Suite), request validation

We have several ways to set up this platform but this time Docker compose was used.<br>
This is a personal memo for the setup procedure.

# Environment
* OS: Kali linux 2019.4
* Docker: 19.03.4
* OWASP Security Shepherd: v3.2

# Solution
### 1. Installing prerequisits
{% highlight shell %}
root@kali:~# apt-get install docker.io docker-compose default-jdk maven

---

root@kali:~# git clone https://github.com/OWASP/SecurityShepherd.git
{% endhighlight %}

### 2. Initial setup
{% highlight shell %}
root@kali:~# cd SecurityShepherd/

root@kali:~/SecurityShepherd# mvn -Pdocker clean install -DskipTests

root@kali:~/SecurityShepherd# service docker start

root@kali:~/SecurityShepherd# docker-compose up -d # -d for get terminal back

---
{% endhighlight %}

### 3. Login
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-17/2019-12-17-15-06-15.png)
We can use the following credential for login.
{% highlight shell %}
admin:password
{% endhighlight %}

After that, change the current password.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-17/2019-12-17-15-06-43.png)

Now we can start the challenges.<br>
By clicking "Get Next Challenge", we cam proceed to the challenges.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-17/2019-12-17-14-52-30.png)

First challenge is for the HTTP request modification with local proxy.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-17/2019-12-17-15-25-29.png)

### 4. Select Open Floor mode
To see all challenges, we have to change the mode to the "Open Floor mode".<br>
Click "Admin" and go to "Module Management" -> "Challenge Module Layout".<br>
Enable the "Open Floor mode" by clicking the button.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-12-17/2019-12-17-22-43-33.png)

### 5. Remove Docker container
{% highlight shell %}
# stop all docker containers
root@kali:~/SecurityShepherd# docker-compose stop

# remove all docker containers
root@kali:~/SecurityShepherd# docker-compose down

# remove all docker containers and Security Shepherd images
root@kali:~/SecurityShepherd# docker-compose down --rmi all

# rebuild
root@kali:~/SecurityShepherd# docker-compose build

---

root@kali:~/SecurityShepherd# docker-compose up -d
{% endhighlight %}
