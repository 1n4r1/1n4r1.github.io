---
layout: post
title: Git rollback memo
categories: Git
---

## Environment
* OS: Kali linux 2019.1
* Git: 2.20.1

## Explanation
How to rollback each procedure of git

## Solution
{% highlight shell %}
# rollback all unstaged modification
$ git reset --hard

# rollback staged files(in repository root directory)
$ git reset .

# rollback newest commit(go back to when 2nd newest commit happened) 
# keep files modified and staged
$ git reset --soft HEAD^

# rollback commit and discard its changing(go back to when 2nd newest commit happened)
$ git reset --hard HEAD^

{% endhighlight %}
