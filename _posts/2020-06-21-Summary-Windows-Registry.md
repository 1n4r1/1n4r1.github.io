---
layout: post
title: Summary - Windows registry
categories: Windows
---

# Explanation
The Windows Registry is a hierarchical database that stores low-level settings for the Windows OS and applications using the registry.
This is a brief summary of How it is and how to browse / edit it.

## Opening with regedit.exe
At first, open the registry with `regedit.exe`. We have 5 entries there.
![placeholder](https://media.githubusercontent.com/media/inar1/inar1.github.io/master/public/images/2020-03-16/postman-badge.png)

### 1. HKEY_CLASSES_ROOT(HKCR)
Merged of 2 keys
`hoge`
`hoge`

### 2. HKEY_CURRENT_USER(HKCU)
Information about the user currently logged in.

### 3. HKEY_LOCAL_MACHINE(HKLM)


### 4. HKEY_USERS(HKU)


### 5. HKEY_CURRENT_CONFIG(HKCC)


## Stored data type

* REG_BINARY : binary
* REG_DWORD : hoge
* REG_SZ : hoge
* REG_EXPAND_SZ : hoge
* REG_MULTI_SZ : hoge
* REG_FULL_RESOURCE_DESCRIPTOR : hoge
* REG_LINK : hoge
* REG_NONE : 
* REG_QWORD : hoge


## Edit with PowerShell



## File path

1. Registry file for current user
* `%USERPROFILE%\NTUSER.DAT`
* `%USERPROFILE%\AppData\Local\Microsoft\Windows\UsrClass.dat`

2. Registry file for
* `C:\Windows\System32\config`

