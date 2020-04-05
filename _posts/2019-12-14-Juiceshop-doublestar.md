---
layout: post
title: Writeup of OWASP Juice Shop 2 stars challenge
categories: OWASP
---

# Explanation
<a href="https://www2.owasp.org/www-project-juice-shop/">OWASP Juice Shop</a> is one of the vulnerable application from OWASP written in Node.js, Express and Angular for the practice.<br>
The application contains a vast number of hacking challenges from 1 star challenge to 5 star challenges.<br>
This is a writeup of 2 stars challenge.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-13-17-30.png)

# Environment
* OS: Kali linux 2019.4
* OWASP Juice Shop: v9.3.0

# Solution

### 1. Admin Section

> Access the administration section of the store.

This is kinda guessing task.<br>
By accessing the <a href="http://localhost:3000/#/administration">http://localhost:3000/#/administration</a>, we can achieve the purpose.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-14-01-29.png)

### 2. Classic Stored XSS
#### Prerequisite
login as a user

> Perform an XSS attack with &lt;script>alert(`xss`)</script> on a legacy page within the application.

Login as any user and go to <a href="http://localhost:3000/#/profile">http://localhost:3000/#/profile</a>.<br>
Set "&lt;script>alert('xss')</script>" as a Username and submit. We can get the following output.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-11-38-16.png)

This sanitaization is not enough.<br>
By setting the following payload, we can execute stored XSS.
{% highlight html %}
<<script>ascript>alert('xss')</script>
{% endhighlight %}


#### 3. Deprecated Interface
### Prerequisite
login as a user

> Use a deprecated B2B interface that was not properly shut down.

Go to <a href="http://localhost:3000/#/complain">http://localhost:3000/#/complain</a>.<br>
By sending XML file, we can achieve this challenge.
{% highlight shell %}
root@kali:~# cat note.xml 
<note nighteye="disabled">
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>

root@kali:~# 
{% endhighlight %}

### 4. Five-Star Feedback
#### Prerequisite
login as a user

> Get rid of all 5-star customer feedback.

After logged in, go to <a href="http://localhost:3000/#/administration">http://localhost:3000/#/administration</a> which we found in the previous challenge.
By clicking the trash bins, delete the 5-star feedback.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-14-05-25.png)
<br>


### 5. Login Admin

> Log in with the administrator's user account.

The login console has SQL injection.<br>
Go to <a href="http://localhost:3000/#/login">http://localhost:3000/#/login</a> and use following username and random password for login credential.<br>
we can login as a user "admin@juice-sh.op".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-14-07-06.png)

### 6.  Login MC SafeSearch

> Log in with MC SafeSearch's original user credentials without applying SQL Injection or any other bypass.

Guessing task I don't like.<br>
If we google the word 'MC SafeSearch', we can find <a href="https://imvdb.com/video/mc-safesearch/protect-ya-passwordz">this video</a>.<br>
In this video, we can find out that the name of MC's dog is "Mr.Noodles".<br>
With the following credential, we can login as a user "mc.safesearch".
{% highlight shell %}
mc.safesearch@juice-sh.op:Mr. N00dles
{% endhighlight %}

### 7. Password Strength
#### Prerequisite
2 star Challenge 5: "Login Admin"

> Log in with the administrator's user credentials without previously changing them or applying SQL Injection.

If the prerequisite is done already, we already know the username is "admin@juice-sh.op".<br>
Launch Burp Suite and go to <a href="http://localhost:3000/#/login">http://localhost:3000/#/login</a>, try to login with random password.<br>
Then, We can find the following traffic.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-13-38-46.png)

This time, use Burp Intruder.<br>
At first, right click the traffic and go to "Send to Intruder".<br>
Specify just "password" with the tab "Positions".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-13-40-42.png)

Next, go to "Paylad" tab. Load "/usr/share/durb/wordlists/others/best1050.txt".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-13-45-27.png)

Then, start attack
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-13-49-48.png)

By filtering, we can find the correct password "admin123".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-13-58-08.png)

### 8. Security Policy
#### Prerequisite
login as a user

> Behave like any "white-hat" should before getting into the action.

We have to just go to "Account" -> "Privacy & Security" -> "Privacy Policy".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-13-55-10.png)

### 9. View Basket
#### Prerequisite
login as a user

> View another user's shopping basket.

Launch Burp Suite and open the page <a href="http://localhost:3000/#/basket">http://localhost:3000/#/basket</a>.<br>
We can find the following traffic. By changing the sending uri to "/rest/basket/2"(With Burp Repeater or whatever), we can clear the challenge.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-02-59-16.png)

### 10. Weird Crypto
#### Prerequisite
login as a user

> Inform the shop about an algorithm or library it should definitely not use the way it does.

We can send the following message and it completes the challenge.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2019-12-14/2019-12-14-13-13-39.png)
