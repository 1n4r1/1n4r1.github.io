---
layout: post
title: Memo - Enable Git Large File Storage on GitHub
categories: Git
---

# Explanation
Git Large File Storage is an extension to replaces binary file with text pointers inside Git repository.<br>
This is a memo of enabling GFS on GitHub,

# Environment
* OS: Kali linux 2019.4

# Solution
## 1. Installation

### Package installation
{% highlight shell %}
root@kali:~# git lfs install
Git LFS initialized.
{% endhighlight %}

### Tracking PNG files
We need the following command for the tracking .png files.
{% highlight shell %}
$ git lfs track "*.png"
Tracking "*.png"
{% endhighlight %}

We can confirm that now we have ".gitattributes"<br>
There is an entry for ".png" file.
{% highlight shell %}
$ cat .gitattributes 
*.png filter=lfs diff=lfs merge=lfs -text
{% endhighlight %}

After that, we need to commit these files again.
{% highlight shell %}
$ git status
On branch Github-LFS
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   public/images/2017-09-21/gcc-1.png
        modified:   public/images/2017-09-21/gcc-2.png
        modified:   public/images/2017-09-21/gcc-3.png
        modified:   public/images/2018-12-01/2018-12-05-09-49-17.png
        modified:   public/images/2018-12-01/2018-12-05-09-50-24.png

---
{% endhighlight %}

Then, commit and push the branch.<br>
We see some diagnostic information about .png file upload.
{% highlight shell %}
$ git push origin Github-LFS

---

Enumerating objects: 761, done.0/310), 21 MB | 965 KB/s                                              
Counting objects: 100% (761/761), done.
Delta compression using up to 8 threads
Compressing objects: 100% (376/376), done.
Writing objects: 100% (385/385), 50.71 KiB | 1.37 MiB/s, done.
Total 385 (delta 5), reused 0 (delta 0)

---
{% endhighlight %}
