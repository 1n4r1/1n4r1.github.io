---
layout: post
title: Changing All Authors and Committers
categories: Git
---

## Environment
* OS: Kali linux 2018.4
* Git: 2.20.1

## Explanation
Changing all authors and committers of git history.

## Solution
{% highlight shell %}
root@kali:~# git filter-branch -f --env-filter "GIT_AUTHOR_NAME='inar1'; GIT_AUTHOR_EMAIL='inar1@protonmail.com'; GIT_COMMITTER_NAME='inar1'; GIT_COMMITTER_EMAIL='inar1@protonmail.com';" HEAD
{% endhighlight %}
