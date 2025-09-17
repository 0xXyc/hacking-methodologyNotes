---
description: 09/16/2025
---

# Domain Reconnaissance Tools

## Introduction/Motivation

After we have some credentials from a few previously compromised hosts on the network, we can perform some domain reconnaissance to determine where we may be able to leverage these credentials across the network.

This section focuses entirely on enumerating the domain as a standard domain user. The primary focus will be on some of the different tools that can be used to query the domain and how we can obtain targeted information.

{% hint style="warning" %}
In some cases, performing domain reconnaissance in a high-integrity process is not required and in some cases (such as `SYSTEM`), it can be detrimental.
{% endhint %}

## PowerView - What is it?

[PowerView](https://github.com/PowerShellMafia/PowerSploit) has long been the de-facto tool for domain enumeration.

One of its biggest strengths focuses on queries returning proper PowerShell objects, which can be "piped" to other cmdlets.

This allows us to be able to "chain" multiple commands together in order to form complex and powerful queries.

```
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
```

## PowerView Queries

### `Get-Domain`

**This returns a domain object for the current or the domain specified via `-Domain`.**

Useful information includes the **domain name**, **forest name**, as well as the **domain controllers** associated within the domain.&#x20;

```
beacon> powershell Get-Domain

Forest                  : cyberbotic.io
DomainControllers       : {dc-2.dev.cyberbotic.io}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  : cyberbotic.io
PdcRoleOwner            : dc-2.dev.cyberbotic.io
RidRoleOwner            : dc-2.dev.cyberbotic.io
InfrastructureRoleOwner : dc-2.dev.cyberbotic.io
Name                    : dev.cyberbotic.io
```

### `Get-ForestDomain`

**Returns all domains within the current forest or the forest specified via `-Forest`.**

```
beacon> powershell Get-ForestDomain

Forest                  : cyberbotic.io
DomainControllers       : {dc-1.cyberbotic.io}
Children                : {dev.cyberbotic.io}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  : 
PdcRoleOwner            : dc-1.cyberbotic.io
RidRoleOwner            : dc-1.cyberbotic.io
InfrastructureRoleOwner : dc-1.cyberbotic.io
Name                    : cyberbotic.io

Forest                  : cyberbotic.io
DomainControllers       : {dc-2.dev.cyberbotic.io}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  : cyberbotic.io
PdcRoleOwner            : dc-2.dev.cyberbotic.io
RidRoleOwner            : dc-2.dev.cyberbotic.io
InfrastructureRoleOwner : dc-2.dev.cyberbotic.io
Name                    : dev.cyberbotic.io
```

### `Get-DomainPolicyData`

**Returns the default domain policy or the domain controller policy or a specified domain/domain controller.**

**It is useful for determining information such as the domain password policy (pair this information with your password attacking technique):**

```
beacon> powershell Get-DomainPolicyData | select -expand SystemAccess

MinimumPasswordAge           : 1
MaximumPasswordAge           : 42
MinimumPasswordLength        : 7
PasswordComplexity           : 1
PasswordHistorySize          : 24
LockoutBadCount              : 0
RequireLogonToChangePassword : 0
ForceLogoffWhenHourExpire    : 0
ClearTextPassword            : 0
LSAAnonymousNameLookup       : 0
```

### `Get-DomainUser`

**Returns all or specified user(s).**

To only return specific properties, you can use `-Properties`.

By default, all user objects for the current domain are returned.

Use `-Identity` to return a specific user.

```
beacon> powershell Get-DomainUser -Identity jking -Properties DisplayName, MemberOf | fl

displayname : John King
memberof    : {CN=Internet Users,CN=Users,DC=dev,DC=cyberbotic,DC=io, CN=Support 
              Engineers,CN=Users,DC=dev,DC=cyberbotic,DC=io}
```

&#x20;This shows that <mark style="color:yellow;">`jking`</mark> is a member of the _<mark style="color:yellow;">**Support Engineers**</mark>_ group.  Most privileges in a domain are delegated to domain groups, rather than individual users.  So if we can find where this domain group has privileged access, we know that members will also have those privileges.

{% hint style="danger" %}
&#x20;If you run this command without the `-Identity` parameter, prepare to wait a while for all the data to return.
{% endhint %}

### `Get-DomainComputer`

**Returns all computers or specific computer objects.**

```
beacon> powershell Get-DomainComputer -Properties DnsHostName | sort -Property DnsHostName

dnshostname              
-----------              
dc-2.dev.cyberbotic.io
fs.dev.cyberbotic.io
sql-2.dev.cyberbotic.io
web.dev.cyberbotic.io
wkstn-1.dev.cyberbotic.io
wkstn-2.dev.cyberbotic.io
```

### `Get-DomainOU`

**Search for all organization units (OUs) or specific OU objects.**

```
beacon> powershell Get-DomainOU -Properties Name | sort -Property Name

name              
----              
Domain Controllers
File Servers
Servers           
SQL Servers       
Web Servers       
Workstations
```

### `Get-DomainGroup`

**Returns all domain groups or specific domain group objects.**

```
beacon> powershell Get-DomainGroup | where Name -like "*Admins*" | select SamAccountName

samaccountname
--------------
Domain Admins 
Key Admins    
DnsAdmins     
MS SQL Admins 
Studio Admins
```

### `Get-DomainGroupMember`

**Returns the members of a specific domain group.**

```
beacon> powershell Get-DomainGroupMember -Identity "Domain Admins" | select MemberDistinguishedName

MemberDistinguishedName                             
-----------------------                             
CN=Nina Lamb,CN=Users,DC=dev,DC=cyberbotic,DC=io    
CN=Administrator,CN=Users,DC=dev,DC=cyberbotic,DC=io
```

### `Get-DomainGPO`

**Returns all Group Policy Objects (GPOs) or specific GPO objects.**

To enumerate all GPOs that are applied to a particular machine, use `-ComputerIdentity`.

```
beacon> powershell Get-DomainGPO -Properties DisplayName | sort -Property DisplayName

displayname                      
-----------                      
Computer Certificates
Default Domain Controllers Policy
Default Domain Policy
LAPS
Proxy Settings
Server Admins
Vulnerable GPO
Windows Defender
Windows Firewall
Workstation Admins
```

### `Get-DomainGPOLocalGroup`

**Returns all GPOs that modify group membership through Restricted Groups or Group Policy Preferences.**

You can manually find which OUs and by extension which computers these GPOs apply to.

```
beacon> powershell Get-DomainGPOLocalGroup | select GPODisplayName, GroupName

GPODisplayName     GroupName            
--------------     ---------            
Workstation Admins DEV\Support Engineers
Server Admins      DEV\Support Engineers
```

{% hint style="info" %}
&#x20;This shows that the _**Support Engineers**_ group is being assigned some sort of local access to the machines to which these GPOs apply.  Although the GPO naming convention suggests this is local admin access, it may also be a different localgroup such as Remote Desktop Users.
{% endhint %}

### `Get-DomainGPOUserLocalGroupMapping`

**Enumerates the machines where a specific domain user/group is a member of a specific local group.**

This is useful for finding where domain groups have local admin access, which is more of an automated approach to performing the manual cross-referencing described above.

```
beacon> powershell Get-DomainGPOUserLocalGroupMapping -LocalGroup Administrators | select ObjectName, GPODisplayName, ContainerName, ComputerName | fl

ObjectName     : Support Engineers
GPODisplayName : Server Admins
ContainerName  : {OU=Servers,DC=dev,DC=cyberbotic,DC=io}
ComputerName   : {web.dev.cyberbotic.io, sql-2.dev.cyberbotic.io, fs.dev.cyberbotic.io}

ObjectName     : Support Engineers
GPODisplayName : Workstation Admins
ContainerName  : {OU=Workstations,DC=dev,DC=cyberbotic,DC=io}
ComputerName   : {wkstn-1.dev.cyberbotic.io, wkstn-2.dev.cyberbotic.io}
```

{% hint style="info" %}
This output confirms that the Support Engineers domain group is a member of the local Administrators group on machines: `web`, `sql-2`, `fs`, `wkstn-1` and `wkstn-2`.
{% endhint %}

### `Get-DomainTrust`

**Returns all domain trusts for the current or specified domain.**

```
beacon> powershell Get-DomainTrust

SourceName      : dev.cyberbotic.io
TargetName      : cyberbotic.io
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 8/15/2022 4:00:00 PM
WhenChanged     : 8/15/2022 4:00:00 PM

SourceName      : dev.cyberbotic.io
TargetName      : dev-studio.com
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : 
TrustDirection  : Inbound
WhenCreated     : 8/16/2022 9:52:37 AM
WhenChanged     : 8/16/2022 9:52:37 AM
```

## SharpView - What is it?

[`SharpView`](https://github.com/tevora-threat/SharpView) was designed to be a <mark style="color:yellow;">**C# port of**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`PowerView`**</mark> and <mark style="color:yellow;">therefore has much the same functionality</mark>.

One downside is that it does not have the same "pipin" ability as `PowerShell`.

```
beacon> execute-assembly C:\Tools\SharpView\SharpView\bin\Release\SharpView.exe Get-Domain

Forest                         : cyberbotic.io
DomainControllers              : {dc-2.dev.cyberbotic.io}
Children                       : {}
DomainMode                     : Unknown
DomainModeLevel                : 7
Parent                         : cyberbotic.io
PdcRoleOwner                   : dc-2.dev.cyberbotic.io
RidRoleOwner                   : dc-2.dev.cyberbotic.io
InfrastructureRoleOwner        : dc-2.dev.cyberbotic.io
Name                           : dev.cyberbotic.io
```

## ADSearch - What is it? + Examples

[`ADSearch`](https://github.com/tomcarver16/ADSearch) has fewer built-in searches compared to `PowerView` and `SharpView`, but it does _<mark style="color:yellow;">**allow you to specify custom Lightweight Directory Access Protocol (LDAP) searches**</mark>_. &#x20;

These can be used to identify entries in the directory that match a given criteria.

For example, we can search for all objects whose category is "user" (e.g. domain users).

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "objectCategory=user"

[*] No domain supplied. This PC's domain will be used instead
[*] LDAP://DC=dev,DC=cyberbotic,DC=io
[*] CUSTOM SEARCH: 

[*] TOTAL NUMBER OF SEARCH RESULTS: 10
	[+] cn : Administrator
	[+] cn : Guest
	[+] cn : krbtgt
	[+] cn : CYBER$
	[+] cn : Bob Farmer
	[+] cn : John King
	[+] cn : Nina Lamb
	[+] cn : MS SQL Service
	[+] cn : Squid Proxy
	[+] cn : STUDIO$
```

Filters can be used to limit the search result further.

For instance, search for all domain groups which end in the word "admins".

```
 beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=group)(cn=*Admins))"

[*] No domain supplied. This PC's domain will be used instead
[*] LDAP://DC=dev,DC=cyberbotic,DC=io
[*] CUSTOM SEARCH: 
[*] TOTAL NUMBER OF SEARCH RESULTS: 5
	[+] cn : Domain Admins
	[+] cn : Key Admins
	[+] cn : DnsAdmins
	[+] cn : MS SQL Admins
	[+] cn : Studio Admins
```

**As a result, these can be made more complex and further `AND`, `OR`, and `NOT` conditions (bitwise logical operations):**

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=group)(cn=MS SQL Admins))" --attributes cn,member

[*] TOTAL NUMBER OF SEARCH RESULTS: 1
	[+] cn     : MS SQL Admins
	[+] member : CN=Developers,CN=Users,DC=dev,DC=cyberbotic,DC=io
```

Additionally, the `--json` parameter can be used to format the output in JSON.

```json
[
  {
    "cn": "MS SQL Admins",
    "member": "CN=Developers,CN=Users,DC=dev,DC=cyberbotic,DC=io"
  }
]
```
