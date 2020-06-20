---
layout: post
title: Summary - Windows registry
categories: Windows
---

# Explanation
The Windows Registry is a hierarchical database that stores low-level settings for the Windows OS and applications using the registry.
This is a brief summary of How it is and how to browse / edit it.


## Opening with regedit.exe
First, open the registry with `regedit.exe`. We have 5 root keys there.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-21/2020-06-20-14-09-55.png)

### 1. HKEY_CLASSES_ROOT(HKCR)
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-21/2020-06-20-14-17-35.png)
A section to manage file type associations.<br>
This key provides a view of the registry that merges the information from `HKEY_LOCAL_MACHINE\Software\Classes` and `HKEY_CURRENT_USER\Software\Classes`.<br>
The `HKEY_LOCAL_MACHINE\Software\Classes` holds default settings that can apply to all users on the local computer.<br>
And the `HKEY_CURRENT_USER\Software\Classes` key contains settings that override that default settings.

### 2. HKEY_CURRENT_USER(HKCU)
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-21/2020-06-20-14-11-23.png)
Information about the user currently logged in.<br>
This is actually a link to `HKEY_USERS\<SID-FOR-CURRENT-USER>`.<br>
`HKCU\Software` holds user-level settings for the most of the software.

### 3. HKEY_LOCAL_MACHINE(HKLM)
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-21/2020-06-20-14-12-10.png)
Majority of the configuration information for the software we install and Windows operating system itself.

#### BCD00000000
Boot configuration Database

#### DRIVERS
Settings for display, 

#### HARDWARE
Holds data pertaining to the BIOS, processors and other hardware devices.

#### SAM
Database for Security Accounts Manager. Need SYSTEM account to access.

#### SECURITY
Stored in `C:\Windows\System32\config\SECURITY`. Need SYSTEM account to access.

#### SOFTWARE
Most commonly accessed from the HKLM hive. Organized alphabetically by the software vendor.<br>
Also, `HKEY_LOCAL_MACHINE\SOFTWARE\Classes` subkey of this key describes various UI details including extensions.

#### SYSTEM
Stored in `C:\Windows\System32\config\SYSTEM`. 

### 4. HKEY_USERS(HKU)
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-21/2020-06-20-14-12-45.png)
Stores all of the settings for all user profiles actively loaded on the system.<br>

### 5. HKEY_CURRENT_CONFIG(HKCC)
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-06-21/2020-06-20-14-17-35.png)
Information about the hardware profile that is used by the local computer at system setup.


## Stored data type
These are the examples of the stored data type of Windows registry.

* REG_BINARY : Binary value.
* REG_DWORD : DWORD (32bit), used for a regular integer value.
* REG_QWORD : QWORD (64bit), used for a 64-bit integer value.
* REG_SZ : String value.
* REG_EXPAND_SZ : String that can contain environment variables, often used for system paths.
* REG_MULTI_SZ : Multiple string used to represent values that contains lists or multiple values, separated by a NULL character 
* REG_RESOURCE_LIST : Series of nested arrays that is designed to store a resource list used by a hardware device driver or one of the physical devices it controls.
* REG_LINK : A Unicode symbolic link.
* REG_NONE : No defined type value.


## How to get SID of users
### Get SID of a local user
```shell
C:\Users\Administrator>wmic useraccount where name='Administrator' get sid
SID
S-1-5-21-299884335-592523710-3968369954-500
```

### Get SID for current logged in domain user
```shell
C:\Users\Administrator>whoami /user

USER INFORMATION
----------------

User Name              SID
====================== ===========================================
mydomain\administrator S-1-5-21-299884335-592523710-3968369954-500
```

## Browse Windows registry with command prompt




## Browse / Edit Windows registry with PowerShell
### Listing all exposed drives including HKLM and HKCU
```shell
PS C:\Users\Administrator> get-psdrive

Name           Used (GB)     Free (GB) Provider      Root                                               CurrentLocation
----           ---------     --------- --------      ----                                               ---------------
Alias                                  Alias
C                  16.14         33.37 FileSystem    C:\                                            Users\Administrator
Cert                                   Certificate   \
D                   0.06          0.00 FileSystem    D:\
Env                                    Environment
Function                               Function
HKCU                                   Registry      HKEY_CURRENT_USER
HKLM                                   Registry      HKEY_LOCAL_MACHINE
Variable                               Variable
WSMan                                  WSMan
```

### Navigate to the local machine registry root key
We can use `cd` command.
```shell
PS C:\Users\Administrator> cd HKLM:\
PS HKLM:\>
```

Or we can use `Set-Location` for PowerShell.
```shell
PS C:\Users\Administrator> set-location -path HKLM:\SOFTWARE
PS HKLM:\SOFTWARE>
```

### Output sub keys
```shell
PS HKLM:\SOFTWARE> Get-Childitem


    Hive: HKEY_LOCAL_MACHINE\SOFTWARE


Name                           Property
----                           --------
Classes
Clients
Intel
Microsoft
ODBC
Oracle
Partner
Policies
RegisteredApplications         File Explorer             :
                               SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Capabilities
                               Internet Explorer         : SOFTWARE\Microsoft\Internet Explorer\Capabilities
                               Paint                     :
                               SOFTWARE\Microsoft\Windows\CurrentVersion\Applets\Paint\Capabilities
                               Windows Address Book      : Software\Clients\Contacts\Address Book\Capabilities
                               Windows Disc Image Burner : Software\Microsoft\IsoBurn\Capabilities
                               Windows Media Player      : Software\Clients\Media\Windows Media Player\Capabilities
                               Windows Photo Viewer      : Software\Microsoft\Windows Photo Viewer\Capabilities
                               Windows Search            : Software\Microsoft\Windows Search\Capabilities
                               Wordpad                   :
                               Software\Microsoft\Windows\CurrentVersion\Applets\Wordpad\Capabilities
```

### Output registry entries in a readable form
```shell
PS HKLM:\> Get-ItemProperty -path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion


ProgramFilesDir          : C:\Program Files
CommonFilesDir           : C:\Program Files\Common Files
ProgramFilesDir (x86)    : C:\Program Files (x86)
CommonFilesDir (x86)     : C:\Program Files (x86)\Common Files
CommonW6432Dir           : C:\Program Files\Common Files
DevicePath               : C:\Windows\inf
MediaPathUnexpanded      : C:\Windows\Media
ProgramFilesPath         : C:\Program Files
ProgramW6432Dir          : C:\Program Files
SM_ConfigureProgramsName : Set Program Access and Defaults
SM_GamesName             : Games
PSPath                   : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVer
                           sion
PSParentPath             : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows
PSChildName              : CurrentVersion
PSDrive                  : HKLM
PSProvider               : Microsoft.PowerShell.Core\Registry
```

### Add a new key
```shell
PS HKCU:\> new-item 'HKCU:\Testkey'


    Hive: HKEY_CURRENT_USER


Name                           Property
----                           --------
Testkey


```

### Add a new property to a key
```shell
PS HKCU:\> new-itemproperty -LiteralPath 'HKCU:Testkey' -Name 'param1' -PropertyType 'String' -Value 'test'


param1       : test
PSPath       : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER\Testkey
PSParentPath : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER
PSChildName  : Testkey
PSDrive      : HKCU
PSProvider   : Microsoft.PowerShell.Core\Registry


```

### Retrieve properties from a key
```shell
PS HKCU:\> Get-ItemProperty -path HKCU:\Testkey


param1       : test
PSPath       : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER\Testkey
PSParentPath : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER
PSChildName  : Testkey
PSDrive      : HKCU
PSProvider   : Microsoft.PowerShell.Core\Registry


```

### Retrieve a value of single property from a key
```shell
PS HKCU:\> Get-ItemPropertyvalue -LiteralPath 'HKCU:Testkey' -Name param1
test
```

### Update a value of single property
```shell
PS HKCU:\> set-itemproperty -Literalpath 'HKCU:Testkey' -Name param1 -Value 'test-test'

PS HKCU:\> Get-ItemPropertyvalue -LiteralPath 'HKCU:Testkey' -Name param1
test-test
```

### Existing check of a key
```shell
PS HKCU:\> Test-Path -LiteralPath "HKCU:\testkey"
True

```

### Delete a key
```shell
PS HKCU:\> Remove-Item -LiteralPath "HKCU:\Testkey"

PS HKCU:\> Test-Path -LiteralPath "HKCU:\testkey"
False
```


## File path
1. Supporting files for all hives except `HKEY_CURRENT_USER`:
* `C:\Windows\System32\config`

2. Supporting files for `HKEY_CURRENT_USER`:
* `%USERPROFILE%\AppData\Local\Microsoft\Windows\Usrclass.dat` or `%LocalAppData%\Microsoft\Windows\Usrclass.dat`

## Reference

* [Windows registry information for advanced users](https://support.microsoft.com/en-ie/help/256986/windows-registry-information-for-advanced-users)
