---
layout: post
title: Memo - Enforcing code regulation with shellcheck
categories: Kali
---

# Explanation
Memo about code regulation tool for shell script "shellcheck"

# Environment
* OS: Kali linux 2019.4

# Solution
## 1. Installation

### Package installation
{% highlight shell %}
root@kali:~# sudo apt-get install shellcheck
{% endhighlight %}

### Cloning the vim plugin syntastic into the directory for Vim plugins
{% highlight shell %}
1n4r1@kali:~$ mkdir -p ~/.vim/pack/plugins/start

1n4r1@kali:~$ cd ~/.vim/pack/plugins/start/

1n4r1@kali:~/.vim/pack/plugins/start$ git clone --depth=1 https://github.com/vim-syntastic/syntastic.git
Cloning into 'syntastic'...
remote: Enumerating objects: 406, done.
remote: Counting objects: 100% (406/406), done.
remote: Compressing objects: 100% (279/279), done.
remote: Total 406 (delta 215), reused 176 (delta 66), pack-reused 0
Receiving objects: 100% (406/406), 324.54 KiB | 1.24 MiB/s, done.
Resolving deltas: 100% (215/215), done.

1n4r1@kali:~/.vim/pack/plugins/start$
{% endhighlight %}

### Adding the following lines in the .vimrc
We have more info on the <a href="https://github.com/vim-syntastic/syntastic">official repository of syntastic</a>
{% highlight shell %}
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*

let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0
{% endhighlight %}

After that, Vim starts to display the additional console for shellcheck.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-03-01/2020-02-29-09-03-23.png)

## 2. Tips
### What code regulation is enforced?
We have rules from SC1000 to SC2236.<br>
If we wanna take a look at what exactly these rules are, try to look at the following page.<br>
<a href="https://github.com/koalaman/shellcheck/wiki/Checks">https://github.com/koalaman/shellcheck/wiki/Checks</a>

### Command-line usage
* To ignore specific rules...
{% highlight shell %}
1n4r1@kali:~/kali-setup$ shellcheck -e SC1090 kali-init.sh
1n4r1@kali:~/kali-setup$ 
{% endhighlight %}

* To adopt only specific rules...
{% highlight shell %}
1n4r1@kali:~/kali-setup$ shellcheck -i SC1090 kali-init.sh

In kali-init.sh line 52:
. ~/.bashrc
  ^-------^ SC1090: Can't follow non-constant source. Use a directive to specify location.

For more information:
  https://www.shellcheck.net/wiki/SC1090 -- Can't follow non-constant source....
{% endhighlight %}

* To specify the output format(like gcc, checkstyle, json)...
{% highlight shell %}
1n4r1@kali:~/kali-setup$ shellcheck --format=gcc  kali-init.sh
kali-init.sh:52:3: warning: Can't follow non-constant source. Use a directive to specify location. [SC1090]
{% endhighlight %}

* To specify the shell...(the default one is bash)
{% highlight shell %}
1n4r1@kali:~/kali-setup$ shellcheck --shell=sh kali-init.sh

In kali-init.sh line 52:
. ~/.bashrc
  ^-------^ SC1090: Can't follow non-constant source. Use a directive to specify location.


In kali-init.sh line 59:
echo -e "\n\n===== Don't forget reboot!! ====="
     ^-- SC2039: In POSIX sh, echo flags are undefined.

For more information:
  https://www.shellcheck.net/wiki/SC1090 -- Can't follow non-constant source....
  https://www.shellcheck.net/wiki/SC2039 -- In POSIX sh, echo flags are undef...
1n4r1@kali:~/kali-setup$ 
{% endhighlight %}

* To specify the severity level...
{% highlight shell %}
1n4r1@kali:~/kali-setup$ shellcheck -S warning kali-init.sh

In kali-init.sh line 52:
. ~/.bashrc
  ^-------^ SC1090: Can't follow non-constant source. Use a directive to specify location.

For more information:
  https://www.shellcheck.net/wiki/SC1090 -- Can't follow non-constant source....
1n4r1@kali:~/kali-setup$ shellcheck -S error kali-init.sh

1n4r1@kali:~/kali-setup$
{% endhighlight %}

* To enable list checks...
{% highlight shell %}
1n4r1@kali:~/kali-setup$ shellcheck --list-optional kali-init.sh
name:    add-default-case
desc:    Suggest adding a default case in `case` statements
example: case $? in 0) echo 'Success';; esac
fix:     case $? in 0) echo 'Success';; *) echo 'Fail' ;; esac

name:    avoid-nullary-conditions
desc:    Suggest explicitly using -n in `[ $var ]`
example: [ "$var" ]
fix:     [ -n "$var" ]

name:    check-unassigned-uppercase
desc:    Warn when uppercase variables are unassigned
example: echo $VAR
fix:     VAR=hello; echo $VAR

name:    quote-safe-variables
desc:    Suggest quoting variables without metacharacters
example: var=hello; echo $var
fix:     var=hello; echo "$var"

name:    require-variable-braces
desc:    Suggest putting braces around all variable references
example: var=hello; echo $var
fix:     var=hello; echo ${var}

1n4r1@kali:~/kali-setup$
{% endhighlight %}


### To ignore specific rules only once in the code
Put a directive like the following
{% highlight shell %}
# shellcheck disable=SC2142
alias hischeck="history|awk '{print \$4}'|sort|uniq -c|sort -n"
{% endhighlight %}


### Bad shell script codes example
* <a href="https://github.com/koalaman/shellcheck/blob/master/README.md#user-content-gallery-of-bad-code">https://github.com/koalaman/shellcheck/blob/master/README.md#user-content-gallery-of-bad-code</a>
