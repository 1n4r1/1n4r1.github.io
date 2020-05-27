---
layout: post
title: Active Directory initial setup
categories: Windows
---

# Explanation
[Active Directory](https://en.wikipedia.org/wiki/Active_Directory) is a directory service for Windows domain networks.<br>
This is a walkthrough of the initial setup using Windows Server 2019.

# Solution
## 1. Download ISO
Go to [Try Windows Server on-premises or on Azure](https://www.microsoft.com/en-us/windows-server/trial) and click "Download free trial".

We can download the ISO of Windows Server 2019.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-08-29-15.png)


## 2. Creating VM on Virtualbox
I don't wanna write since it's gonna be too lengthy!!


## 3. Installing Windows Server 2019
We have 2 steps to be careful.

### 1. Select "Windows Server 2019 Standard (Desktop Experience)"
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-09-11-55.png)

### 2. Select "Custom: Install Windows Only (advanced)"
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/)


## 4. Setup Active Directory
When we launch the server, Server manager is automatically started.<br>

### 1. Click "Add roles and features".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-10-55-52.png)

### 2. "Before you begin"
Click "Next".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-11-02-04.png)

### 3. "Select installation type"
Select "Role-based or feature-based installation".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-11-02-22.png)

### 4. "Select destination server"
Select a server. This time, we have only one candidate "WIN-NQ3R36R6PUN"
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-21-15-38.png)

### 5. "Select Server roles"
Select "Active Directory Domain Services" and add some features required for Active Directory.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-21-18-44.png)

Confirm "Active Directory Domain Services" is check and go next.

### 6. "Features"
We've already added features. Just click "Next".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-12-46-14.png)

### 7. "AD DS"
Explanation for Active Directory. Click "Next".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-21-23-17.png)

### 8. "Confirmation installation selections"
If no problem, click "Install".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-12-48-25.png)

After installation, we can promote this server to domain controller by clicking the link.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-12-50-54.png)

### 9. "Deployment Configuration"
If we click the "promote this server to domain controller", we come to this section.<br>
"Add a new forest" and set domain name, click "Next".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-08-36.png)

### 10. "Domain Controller Options"
Put password.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-12-56.png)

### 11. "DNS Options"
We don't have DNS service, we can just click "Next"

### 12. "Additional Options"
We can click "Next", however the name "AD" is not suitable for the actual use.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-16-30.png)

### 13. "Path"
Configuration for Database folder, log files and SYSVOL folder.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-18-39.png)

### 14. "Review Options"
Check the configuration and click "Next"
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-20-20.png)

### 15. "Prerequisite Check"
We get some errors.<br>
However, if we have a message "All prerequisite checks passed successfully. Click 'install' to begin installation"
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-22-33.png)


## 4. Testing
### 1. Login as administrator
Try to login as Active Directory Administrator.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-35-04.png)

### 2. Server Manager
Now we have a menu for "AD DS".<br>
Click "AD DS" and check if we have our server name "WIN-71O2SFQSVV3".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-39-22.png)

Also, if we right click, we can find that we have management menus for "AD DS".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-39-45.png)

Click "Active Directory Users and Computers".<br>
Then, go to "ad.mycooladmin.com" -> "Domain Controllers". We can find that this PC is recognized as a "Domain Controller".
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-43-04.png)

### 3. DNS configuration
If we have succeeded the configuration of DNS, it should be '127.0.0.1'.
`netsh interface ip show dnsservers`
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-05-28/2020-05-27-13-52-03.png)
