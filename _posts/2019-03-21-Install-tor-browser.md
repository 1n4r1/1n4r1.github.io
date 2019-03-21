---
layout: post
title: Installing Tor browser on Kali linux 2019.01
categories: Kali
---

## Environment
* OS: Kali linux 2019.1


## Explanation
How to install Tor browser on Kali linux(Not manually).<br>
If we install <a href="https://www.torproject.org/projects/torbrowser.html.en">Tor Browser</a> manually, it runs as root user.<br>
Besides, it's not good way for maintenance.<br>


## Solution
Just one simple apt-get is enough.
{% highlight shell %}
sudo apt-get install tor torbrowser-launcher
{% endhighlight %}
