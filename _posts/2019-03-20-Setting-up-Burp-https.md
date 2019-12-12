---
layout: post
title: Setting up Burpsuite for HTTPS on Kali linux 2019.01
categories: Burp Suite
---

## Environment
* OS: Kali linux 2019.1
* Burp Suite: Burp Suite Community Edition v1.7.36
* Chrome: Version 73.0.3683.75


## Explanation
How to install a Burp SSL certification to chrome.<br>
I have done this more than 5 times but still I forget so took this memo.

## Solution
### 1. SSL Error
Without any settings, if we use Burp with https, browser shows this certification error.
![placeholder](https://inar1.github.io/public/images/2019-03-20/2019-03-19-23-35-56.png)

### 2. Download SSL cert
By accessing the Burp page on localhost, we can download the certificate "cacert.der".<br>
We have to click on the button "CA Certificate".
![placeholder](https://inar1.github.io/public/images/2019-03-20/2019-03-20-00-08-41.png)

### 3. Register the cert on google chrome
Go to settings and click "Advanced". There is a menu "Manage certificates".
![placeholder](https://inar1.github.io/public/images/2019-03-20/2019-03-19-23-42-27.png)

Click "Authorities", then "Import".<br>
After selected "cacert.der" downloaded, it shows some options.
![placeholder](https://inar1.github.io/public/images/2019-03-20/2019-03-19-23-46-12.png)

Only choosing the first one "Trust this certificate for identifying websites" is enough.<br>

### 4. Restart
Then, restart the chrome and it would be fine.

### 5. If still had a same error?
In this case, we can confirm the validity of certification on "Manage certificates".<br>
Click on "org-PortSwigger" and that certificate would be "untrusted".
![placeholder](https://inar1.github.io/public/images/2019-03-20/2019-03-20-00-03-21.png)

We can edit the certificate, or delete and install it again.
