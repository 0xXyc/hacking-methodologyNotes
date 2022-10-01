# Resolute (Medium)

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
* Attempt AS-REP Roasting



## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
