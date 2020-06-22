---
layout: post
title: Git skip https auth
categories: Git
---

## Environment
* OS: Kali linux 2019.1
* Git: 2.20.1

## Explanation
How to skip authiorization when we access remote repository via https<br>
Example:
{% highlight shell %}
root@kali:~/1n4r1.github.io# git push origin git-skip-auth
Username for 'https://github.com': 1n4r1
Password for 'https://1n4r1@github.com': 
{% endhighlight %}

## Solution
We can use <a href="https://git-scm.com/docs/gitcredentials">git-credential</a>.<br>
Since I'm using Kali, I can not use any authentication procedure of OS.<br>
So I have to store my credential in local directory.
{% highlight shell %}
git config --global credential.helper store
{% endhighlight %}

We have to login with username/password one time after this command.<br>
Then, the auth data will be stored here
{% highlight shell %}
root@kali:~# cat .git-credentials 
https://1n4r1:SuperStrongFakePassword!!@github.com
root@kali:~#
{% endhighlight %}

Now, we don't have to put credentials any more.
