---
layout: post
title: Update forked repository
categories: Git
---

## Environment
* OS: Kali linux 2019.1
* Git: 2.20.1

## Explanation
Update forked repository after update of remote master branch.

## Solution
{% highlight shell %}
# register remote repository as "upstream".
$ git remote add upstream https://github.com/XXXXXXX/XXXXXXX.git

# update information of remote repository local repository has
$ git fetch upstream

# checkout to local master branch
$ git checkout master

# merge original master branch to local master
$ git merge --ff upstream/master
{% endhighlight %}
