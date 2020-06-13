---
layout: post
title: Getting started Nessus
categories: Vulnerability Scanner
---

# Explanation
[Nessus](https://www.tenable.com/products/nessus) is a proprietary vulnerability scanner developed by Tenable, Inc.<br>
This is a walkthrough of the initial setup and first scan of Nessus free edition using Kali Linux 2020.

# Environment
* Kali Linux 2020
* Nessus: 8.10.1

# Solution
## 1. Installation
Since we can't install it with `apt-get`, we have to download the deb package from the [website](https://www.tenable.com/downloads/nessus) and install manually.
```shell
root@kali:~# dpkg -i Nessus-8.10.1-debian6_amd64.deb 
Selecting previously unselected package nessus.
(Reading database ... 460345 files and directories currently installed.)
Preparing to unpack Nessus-8.10.1-debian6_amd64.deb ...
Unpacking nessus (8.10.1) ...
Setting up nessus (8.10.1) ...
Unpacking Nessus Scanner Core Components...

 - You can start Nessus Scanner by typing /etc/init.d/nessusd start
 - Then go to https://kali:8834/ to configure your scanner

Processing triggers for systemd (245.5-3) ...
```

To run Nessus, we can use the following command.
```shell
root@kali:~# systemctl start nessusd

root@kali:~# systemctl status nessusd
● nessusd.service - LSB: Starts and stops the Nessus
     Loaded: loaded (/etc/init.d/nessusd; generated)
     Active: active (running) since Wed 2020-06-10 22:15:39 EEST; 4s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 54984 ExecStart=/etc/init.d/nessusd start (code=exited, status=0/SUCCESS)
      Tasks: 11 (limit: 19010)
     Memory: 99.4M
     CGroup: /system.slice/nessusd.service
             ├─54986 /opt/nessus/sbin/nessus-service -D -q
             └─54987 nessusd -q

Jun 10 22:15:39 kali systemd[1]: Starting LSB: Starts and stops the Nessus...
Jun 10 22:15:39 kali nessusd[54984]: Starting Nessus : .
Jun 10 22:15:39 kali systemd[1]: Started LSB: Starts and stops the Nessus.
```


## 2. Gaining activation code
Go to [Obtain an Activation Code](https://www.tenable.com/products/nessus/activation-code).<br>
This time, select free edition and push "Register Now".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)

After that, fill out the following information.<br>
The activation code will be sent to the registered email address.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)


## 3. Setting up
Go to `https://localhost:8834`.<br>
Select `Nessus Essential`.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)

We can skip the next step because we already have an active code.<br>
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)

Then, create an username and password.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)

After that, it takes some time to finish the initialization.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)


## 4. Creating a new scan
We have `Scans` Tab on console. Click and select `New Scan` on the right side.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)

After that, we need to choose the scan template.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)

Then, setting up the target machine.<br>
At least we need to specify the IP address and push "save".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)

When finished the configuration, go to "My Scans".<br>
On the right side, we have an icon to launch a scan.


## 5. Scan result
Click the scan and go to "Vulnerabilities". We can see the information about vulnerabilities.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-11/forest.png)
