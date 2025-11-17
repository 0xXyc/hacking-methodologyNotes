---
description: 11/18/2025
---

# Cheat Sheet Part 3

## Domain Reconnaissance

### PowerView

**Import script via PowerShell:**

```
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
```

#### Get-Domain

**Returns a domain object for the current domain or the domain specified with `-Domain`. Useful information includes the domain name, the forest name and the domain controllers:**

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

#### Get-DomainController

**Returns the domain controllers for the current or specified domain:**

```
beacon> powershell Get-DomainController | select Forest, Name, OSVersion | fl

Forest    : cyberbotic.io
Name      : dc-2.dev.cyberbotic.io
OSVersion : Windows Server 2022 Datacenter
```

#### Get-ForestDomain

**Returns all domains for the current forest or the forest specified by `-Forest`:**

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

#### Get-DomainPolicyData

**Returns the default domain policy or the domain controller policy for the current domain or a specified domain/domain controller. Useful for finding information such as the domain password policy:**

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

#### Get-DomainUser

Return all (or specific) user(s). To only return specific properties, use `-Properties`. By default, all user objects for the current domain are returned, use `-Identity` to return a specific user.

#### Return All Users

```
beacon> powershell Get-DomainUser -Properties DisplayName, MemberOf | fl

memberof : {CN=Group Policy Creator Owners,CN=Users,DC=dev,DC=cyberbotic,DC=io, CN=Domain 
           Admins,CN=Users,DC=dev,DC=cyberbotic,DC=io, CN=Administrators,CN=Builtin,DC=dev,DC=cyberbotic,DC=io}

memberof : CN=Guests,CN=Builtin,DC=dev,DC=cyberbotic,DC=io

memberof : CN=Denied RODC Password Replication Group,CN=Users,DC=dev,DC=cyberbotic,DC=io

displayname : Bob Farmer
memberof    : {CN=Internet Users,CN=Users,DC=dev,DC=cyberbotic,DC=io, 
              CN=Developers,CN=Users,DC=dev,DC=cyberbotic,DC=io}

displayname : John King
memberof    : {CN=Internet Users,CN=Users,DC=dev,DC=cyberbotic,DC=io, CN=MS SQL 
              Admins,CN=Users,DC=dev,DC=cyberbotic,DC=io, CN=Support Engineers,CN=Users,DC=dev,DC=cyberbotic,DC=io}

displayname : Nina Lamb
memberof    : {CN=Studio Admins,CN=Users,DC=dev,DC=cyberbotic,DC=io, CN=Internet 
              Users,CN=Users,DC=dev,DC=cyberbotic,DC=io, CN=Domain Admins,CN=Users,DC=dev,DC=cyberbotic,DC=io}

displayname : MS SQL Service

displayname : Squid Proxy

displayname : Honey Token
```

#### Return a Specific User

```
beacon> powershell Get-DomainUser -Identity jking -Properties DisplayName, MemberOf | fl

displayname : John King
memberof    : {CN=Internet Users,CN=Users,DC=dev,DC=cyberbotic,DC=io, CN=Support 
              Engineers,CN=Users,DC=dev,DC=cyberbotic,DC=io}
```

This shows that `jking` is a member of the **Support Engineers** group. &#x20;

Most privileges in a domain are delegated to domain groups, rather than individual users. &#x20;

So if we can find where this domain group has privileged access, we know that members will also have those privileges.

#### Get-DomainComputer

**Return all computers or specific computer objects:**

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

#### Get-DomainOU

**Search for all organization units (OUs) or specific OU objects:**

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

#### Get-DomainGroup

**Return all domain groups or specific domain group objects:**

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

#### Get-DomainGroupMember

**Return the members of a specific domain group:**

```
beacon> powershell Get-DomainGroupMember -Identity "Domain Admins" | select MemberDistinguishedName

MemberDistinguishedName                             
-----------------------                             
CN=Nina Lamb,CN=Users,DC=dev,DC=cyberbotic,DC=io    
CN=Administrator,CN=Users,DC=dev,DC=cyberbotic,DC=io
```

#### Get-DomainGPO

Return all Group Policy Objects (GPOs) or specific GPO objects. To enumerate all GPOs that are applied to a particular machine, use `-ComputerIdentity`.

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

#### Get-DomainGPOLocalGroup

Returns all GPOs that modify local group membership through Restricted Groups or Group Policy Preferences.  You can then manually find which OUs, and by extension which computers, these GPOs apply to.

```
beacon> powershell Get-DomainGPOLocalGroup | select GPODisplayName, GroupName

GPODisplayName     GroupName            
--------------     ---------            
Workstation Admins DEV\Support Engineers
Server Admins      DEV\Support Engineers
```

This shows that the Support Engineers group is being assigned some sort of local access to the machines to which these GPOs apply.  Although the GPO naming convention suggests this is local admin access, it may also be a different localgroup such as Remote Desktop Users.

#### Get-DomainGPOUserLocalGroupMapping

Enumerates the machines where a specific domain user/group is a member of a specific local group.  This is useful for finding where domain groups have local admin access, which is a more automated way to perform the manual cross-referencing described above.

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

&#x20;This output confirms that the Support Engineers domain group is a member of the local Administrators group on machines: web, sql-2, fs, wkstn-1 and wkstn-2.

#### Get-DomainTrust

Return all domain trusts for the current or specified domain.

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

### SharpView

[SharpView](https://github.com/tevora-threat/SharpView) was designed to be a C# port of PowerView and therefore has much the same functionality.  However, one downside is that it doesn't have the same piping ability as PowerShell.

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

### ADSearch

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

Filters can be used to limit the search result further.  For instance, search for all domain groups which end in the word "admins".

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

These can be made more complex with further AND, OR and NOT conditions.  All attributes can be returned using the `--full` parameter, or specific attributes with the `--attributes` parameter.

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=group)(cn=MS SQL Admins))" --attributes cn,member

[*] TOTAL NUMBER OF SEARCH RESULTS: 1
	[+] cn     : MS SQL Admins
	[+] member : CN=Developers,CN=Users,DC=dev,DC=cyberbotic,DC=io
```

Additionally, the `--json` parameter can be used to format the output in JSON.

```
[
  {
    "cn": "MS SQL Admins",
    "member": "CN=Developers,CN=Users,DC=dev,DC=cyberbotic,DC=io"
  }
]
```
