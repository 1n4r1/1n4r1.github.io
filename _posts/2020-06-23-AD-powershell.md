---
layout: post
title: Memo - Operating AD DS with PowerShell
categories: Windows
---

# Explanation
Brief memo about how to operate / recon AD DS with PowerShell

# Environment
* Windows Server 2016 Standard Evaluation
* Powershell 5.1

```shell
PS C:\> Get-ComputerInfo -Property Windows*


WindowsBuildLabEx              : 14393.693.amd64fre.rs1_release.161220-1747
WindowsCurrentVersion          : 6.3
WindowsEditionId               : ServerStandardEval
WindowsInstallationType        : Server
WindowsInstallDateFromRegistry : 6/1/2020 6:37:57 AM
WindowsProductId               : 00378-00000-00000-AA739
WindowsProductName             : Windows Server 2016 Standard Evaluation
WindowsRegisteredOrganization  :
WindowsRegisteredOwner         : Windows User
WindowsSystemRoot              : C:\Windows
```

```shell
PS C:\> echo $PSversiontable

Name                           Value
----                           -----
PSVersion                      5.1.14393.693
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.14393.693
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

# Solution
## Import
Only for the current console. If needed, put the command in the script of `$profile`.
```shell
PS C:\> Import-Module activedirectory

PS C:\> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.0.0    activedirectory                     {Add-ADCentralAccessPolicyMember, Add-ADComputerServiceAccou...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     1.2        PSReadline                          {Get-PSReadlineKeyHandler, Get-PSReadlineOption, Remove-PSRe...
```

## Listing commandlets in Active Directory module
```shell
PS C:\> Get-Command -Module activedirectory

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Add-ADCentralAccessPolicyMember                    1.0.0.0    activedirectory
Cmdlet          Add-ADComputerServiceAccount                       1.0.0.0    activedirectory
Cmdlet          Add-ADDomainControllerPasswordReplicationPolicy    1.0.0.0    activedirectory
Cmdlet          Add-ADFineGrainedPasswordPolicySubject             1.0.0.0    activedirectory

---

```

## Mounting AD: drive and operate like a file system
```shell
PS C:\Users\Administrator> cd AD:

PS AD:\> cd "dc=mydomain,dc=local"

PS AD:\dc=mydomain,dc=local> dir

Name                 ObjectClass          DistinguishedName
----                 -----------          -----------------
Builtin              builtinDomain        CN=Builtin,DC=mydomain,DC=local
Computers            container            CN=Computers,DC=mydomain,DC=local
Domain Controllers   organizationalUnit   OU=Domain Controllers,DC=mydomain,DC=local
ForeignSecurityPr... container            CN=ForeignSecurityPrincipals,DC=mydomain,DC=local
Infrastructure       infrastructureUpdate CN=Infrastructure,DC=mydomain,DC=local
Keys                 container            CN=Keys,DC=mydomain,DC=local
LostAndFound         lostAndFound         CN=LostAndFound,DC=mydomain,DC=local
Managed Service A... container            CN=Managed Service Accounts,DC=mydomain,DC=local
NTDS Quotas          msDS-QuotaContainer  CN=NTDS Quotas,DC=mydomain,DC=local
Program Data         container            CN=Program Data,DC=mydomain,DC=local
System               container            CN=System,DC=mydomain,DC=local
testunit             organizationalUnit   OU=testunit,DC=mydomain,DC=local
TPM Devices          msTPM-Information... CN=TPM Devices,DC=mydomain,DC=local
Users                container            CN=Users,DC=mydomain,DC=local

PS AD:\dc=mydomain,dc=local> cd cn=users

PS AD:\cn=users,dc=mydomain,dc=local> ls

Name                 ObjectClass          DistinguishedName
----                 -----------          -----------------
Administrator        user                 CN=Administrator,CN=Users,DC=mydomain,DC=local
Allowed RODC Pass... group                CN=Allowed RODC Password Replication Group,CN=Users,DC=mydomain,DC=local
Cert Publishers      group                CN=Cert Publishers,CN=Users,DC=mydomain,DC=local
Cloneable Domain ... group                CN=Cloneable Domain Controllers,CN=Users,DC=mydomain,DC=local
DefaultAccount       user                 CN=DefaultAccount,CN=Users,DC=mydomain,DC=local
Denied RODC Passw... group                CN=Denied RODC Password Replication Group,CN=Users,DC=mydomain,DC=local
DnsAdmins            group                CN=DnsAdmins,CN=Users,DC=mydomain,DC=local
DnsUpdateProxy       group                CN=DnsUpdateProxy,CN=Users,DC=mydomain,DC=local
Domain Admins        group                CN=Domain Admins,CN=Users,DC=mydomain,DC=local
Domain Computers     group                CN=Domain Computers,CN=Users,DC=mydomain,DC=local
Domain Controllers   group                CN=Domain Controllers,CN=Users,DC=mydomain,DC=local
Domain Guests        group                CN=Domain Guests,CN=Users,DC=mydomain,DC=local
Domain Users         group                CN=Domain Users,CN=Users,DC=mydomain,DC=local
Enterprise Admins    group                CN=Enterprise Admins,CN=Users,DC=mydomain,DC=local
Enterprise Key Ad... group                CN=Enterprise Key Admins,CN=Users,DC=mydomain,DC=local
Enterprise Read-o... group                CN=Enterprise Read-only Domain Controllers,CN=Users,DC=mydomain,DC=local
Group Policy Crea... group                CN=Group Policy Creator Owners,CN=Users,DC=mydomain,DC=local
Guest                user                 CN=Guest,CN=Users,DC=mydomain,DC=local
Key Admins           group                CN=Key Admins,CN=Users,DC=mydomain,DC=local
krbtgt               user                 CN=krbtgt,CN=Users,DC=mydomain,DC=local
Protected Users      group                CN=Protected Users,CN=Users,DC=mydomain,DC=local
RAS and IAS Servers  group                CN=RAS and IAS Servers,CN=Users,DC=mydomain,DC=local
Read-only Domain ... group                CN=Read-only Domain Controllers,CN=Users,DC=mydomain,DC=local
Schema Admins        group                CN=Schema Admins,CN=Users,DC=mydomain,DC=local

# If go to OU
PS AD:\cn=users,dc=mydomain,dc=local> cd ../

PS AD:\dc=mydomain,dc=local> cd "ou=Domain Controllers"

PS AD:\ou=Domain Controllers,dc=mydomain,dc=local> ls

Name                 ObjectClass          DistinguishedName
----                 -----------          -----------------
WIN-K0TMKMC41V4      computer             CN=WIN-K0TMKMC41V4,OU=Domain Controllers,DC=mydomain,DC=local

```

Reference: [Mount Active Directory as a drive in PowerShell](https://4sysops.com/archives/mount-active-directory-as-a-drive-in-powershell/#changing-to-a-domain-or-an-ou)

## Listing all users on a domain
```shell
PS C:\> Get-ADUser -Filter *

---

```

## Showing property of a specific user
```shell
PS C:\Users\Administrator> get-ADUser -Identity Administrator


DistinguishedName : CN=Administrator,CN=Users,DC=mydomain,DC=local
Enabled           : True
GivenName         :
Name              : Administrator
ObjectClass       : user
ObjectGUID        : 9966e66a-0e0f-407a-811c-06b3937a3823
SamAccountName    : Administrator
SID               : S-1-5-21-299884335-592523710-3968369954-500
Surname           :
UserPrincipalName :
```

## Checking Password Policy
```shell
PS C:\Users\Administrator> Get-ADDefaultDomainPasswordPolicy


ComplexityEnabled           : True
DistinguishedName           : DC=mydomain,DC=local
LockoutDuration             : 00:30:00
LockoutObservationWindow    : 00:30:00
LockoutThreshold            : 0
MaxPasswordAge              : 42.00:00:00
MinPasswordAge              : 1.00:00:00
MinPasswordLength           : 7
objectClass                 : {domainDNS}
objectGuid                  : 11d78b80-7a3f-4187-a841-656090e12b5d
PasswordHistoryCount        : 24
ReversibleEncryptionEnabled : False
```

As we can see, the property `ComplexityEnabled` is `True`.<br>
This means the password should contain characters from three of the following categories.
1. Uppercase characters
2. Lowercase characters
3. Base 10 digits (0 ~ 9)
4. Special characters

## Adding a new Organizational unit
```shell
PS C:\Users\Administrator> New-ADOrganizationalUnit -Name "testunit" -Path "DC=mydomain,DC=local"
```

## Adding a new user for existing organizational unit
```shell
PS C:\Users\Administrator> New-ADUser testuser -GivenName Doe -Surname John -Path "OU=testunit,DC=mydomain,DC=l
ocal" -UserPrincipalName "testuser@mydomain.local" -AccountPassword (ConvertTo-SecureString -AsPlainText "MyPas
sw0rd!" -Force)

PS C:\Users\Administrator> $user = get-ADuser -Identity testuser

PS C:\Users\Administrator> $user | set-ADUser -Enabled $true

PS C:\Users\Administrator> get-ADuser -Identity testuser


DistinguishedName : CN=testuser,OU=testunit,DC=mydomain,DC=local
Enabled           : True
GivenName         : Doe
Name              : testuser
ObjectClass       : user
ObjectGUID        : 6cb9d195-3294-4be9-9cd5-44aff15dc136
SamAccountName    : testuser
SID               : S-1-5-21-299884335-592523710-3968369954-1104
Surname           : John
UserPrincipalName : testuser@mydomain.local
```

## Find Groups of a specific user
```shell
PS C:\Users\Administrator> Get-ADPrincipalGroupMembership -Identity testuser


distinguishedName : CN=Domain Users,CN=Users,DC=mydomain,DC=local
GroupCategory     : Security
GroupScope        : Global
name              : Domain Users
objectClass       : group
objectGUID        : afa11224-eddf-4927-aecd-440d0ac358a8
SamAccountName    : Domain Users
SID               : S-1-5-21-299884335-592523710-3968369954-513
```

## Find members of a specific OU
```shell
PS C:\Users\Administrator> Get-ADUser -Filter * -SearchBase "OU=testunit,DC=mydomain,DC=local"


DistinguishedName : CN=testuser,OU=testunit,DC=mydomain,DC=local
Enabled           : True
GivenName         : Doe
Name              : testuser
ObjectClass       : user
ObjectGUID        : c98d50b5-c8af-4bc5-a3b8-a4db9f3816aa
SamAccountName    : testuser
SID               : S-1-5-21-299884335-592523710-3968369954-1105
Surname           : John
UserPrincipalName : testuser@mydomain.local
```

## Using LDAP query to find users
```shell
PS C:\Users\Administrator> Get-ADUser -LDAPFilter "(Name=testuser)"


DistinguishedName : CN=testuser,OU=testunit,DC=mydomain,DC=local
Enabled           : True
GivenName         : Doe
Name              : testuser
ObjectClass       : user
ObjectGUID        : c98d50b5-c8af-4bc5-a3b8-a4db9f3816aa
SamAccountName    : testuser
SID               : S-1-5-21-299884335-592523710-3968369954-1105
Surname           : John
UserPrincipalName : testuser@mydomain.local
```

## Search for a computer with specific IPv4 address
```shell
PS C:\Users\Administrator> Get-ADComputer -Filter 'IPV4Address -eq "10.0.2.15"'


DistinguishedName : CN=WIN-K0TMKMC41V4,OU=Domain Controllers,DC=mydomain,DC=local
DNSHostName       : WIN-K0TMKMC41V4.mydomain.local
Enabled           : True
Name              : WIN-K0TMKMC41V4
ObjectClass       : computer
ObjectGUID        : f8f4913d-3007-4a74-b215-421c0e8b18dd
SamAccountName    : WIN-K0TMKMC41V4$
SID               : S-1-5-21-299884335-592523710-3968369954-1000
UserPrincipalName :
```

## Listing all Group Policy Object
```shell
PS C:\Users\Administrator> Get-GPO -All


DisplayName      : Default Domain Policy
DomainName       : mydomain.local
Owner            : MYDOMAIN\Domain Admins
Id               : 31b2f340-016d-11d2-945f-00c04fb984f9
GpoStatus        : AllSettingsEnabled
Description      :
CreationTime     : 6/1/2020 12:27:11 AM
ModificationTime : 6/1/2020 1:34:02 AM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 3, SysVol Version: 3
WmiFilter        :

DisplayName      : Default Domain Controllers Policy
DomainName       : mydomain.local
Owner            : MYDOMAIN\Domain Admins
Id               : 6ac1786c-016f-11d2-945f-00c04fb984f9
GpoStatus        : AllSettingsEnabled
Description      :
CreationTime     : 6/1/2020 12:27:11 AM
ModificationTime : 6/1/2020 12:27:10 AM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 1, SysVol Version: 1
WmiFilter        :
```

## Listing all A records
```shell
PS C:\Users\Administrator> Get-DnsServerResourceRecord -ZoneName "mydomain.local" -RRType "A"

HostName                  RecordType Type       Timestamp            TimeToLive      RecordData
--------                  ---------- ----       ---------            ----------      ----------
@                         A          1          6/19/2020 5:00:00 AM 00:10:00        10.0.2.15
DomainDnsZones            A          1          6/19/2020 5:00:00 AM 00:10:00        10.0.2.15
ForestDnsZones            A          1          6/19/2020 5:00:00 AM 00:10:00        10.0.2.15
win-k0tmkmc41v4           A          1          0                    00:20:00        10.0.2.15
```

## Listing all SRV records
```shell
PS C:\Users\Administrator> Get-DnsServerResourceRecord -ZoneName "mydomain.local" -RRType "SRV"

HostName                  RecordType Type       Timestamp            TimeToLive      RecordData
--------                  ---------- ----       ---------            ----------      ----------
_gc._tcp.Default-First... SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][3268][WIN-K0TMKMC41V4.mydomain.local.]
_kerberos._tcp.Default... SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][88][WIN-K0TMKMC41V4.mydomain.local.]
_ldap._tcp.Default-Fir... SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][389][WIN-K0TMKMC41V4.mydomain.local.]
_gc._tcp                  SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][3268][WIN-K0TMKMC41V4.mydomain.local.]
_kerberos._tcp            SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][88][WIN-K0TMKMC41V4.mydomain.local.]
_kpasswd._tcp             SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][464][WIN-K0TMKMC41V4.mydomain.local.]
_ldap._tcp                SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][389][WIN-K0TMKMC41V4.mydomain.local.]
_kerberos._udp            SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][88][WIN-K0TMKMC41V4.mydomain.local.]
_kpasswd._udp             SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][464][WIN-K0TMKMC41V4.mydomain.local.]
_ldap._tcp.Default-Fir... SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][389][WIN-K0TMKMC41V4.mydomain.local.]
_ldap._tcp.DomainDnsZones SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][389][WIN-K0TMKMC41V4.mydomain.local.]
_ldap._tcp.Default-Fir... SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][389][WIN-K0TMKMC41V4.mydomain.local.]
_ldap._tcp.ForestDnsZones SRV        33         6/19/2020 10:00:0... 00:10:00        [0][100][389][WIN-K0TMKMC41V4.mydomain.local.]
```
