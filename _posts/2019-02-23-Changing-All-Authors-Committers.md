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

This causes conflict between local git repo and remote.<br>
After this command, we have to use git push --force.

{% highlight shell %}
root@kali:~/inar1.github.io# git push origin master
Username for 'https://github.com': inar1
Password for 'https://inar1@github.com': 
To https://github.com/inar1/inar1.github.io.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/inar1/inar1.github.io.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.


root@kali:~/inar1.github.io# git push -- force origin master
fatal: 'force' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
{% endhighlight %}
