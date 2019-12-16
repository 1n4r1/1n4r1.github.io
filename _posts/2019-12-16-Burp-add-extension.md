---
layout: post
title: Find reflected XXS with Burp extension reflector
categories: Burp
---

# Explanation
To automate the reflected XSS finding process, install a burp extension <a href="https://github.com/elkokc/reflector">"Reflector"</a><br>

# Environment
* OS: Kali linux 2019.4
* Burp Suite: Burp Suite Community Edition v2.1.04
* Plugin: reflector 2.1

# Solution

## 1. Setting up Burp Suite

At first, download the .jar file from <a href="https://github.com/elkokc/reflector/releases">https://github.com/elkokc/reflector/releases</a>.
![placeholder](https://inar1.github.io/public/images/2019-12-16/2019-12-16-11-34-42.png)

Then launch Burp Suite and go to "Extender" -> "Extensions" tab.<br>
Click "Add" and set "reflector2.1" as "Extension file".
![placeholder](https://inar1.github.io/public/images/2019-12-16/2019-12-16-11-02-18.png)

Now we had new tab "Reflector".
![placeholder](https://inar1.github.io/public/images/2019-12-16/2019-12-16-11-08-00.png)
