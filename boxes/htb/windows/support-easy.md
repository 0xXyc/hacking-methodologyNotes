---
description: 10-22-22
---

# Support (Easy)

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-10-22 19:59:08Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -3s
| smb2-time: 
|   date: 2022-10-22T19:59:13
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required

PORT      STATE SERVICE    VERSION
5985/tcp  open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf     .NET Message Framing
49664/tcp open  msrpc      Microsoft Windows RPC
49668/tcp open  msrpc      Microsoft Windows RPC
49676/tcp open  ncacn_http Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc      Microsoft Windows RPC
49695/tcp open  msrpc      Microsoft Windows RPC
49711/tcp open  msrpc      Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

Enumerated UDP ports:

<pre><code>PORT    STATE SERVICE VERSION
53/udp  open  domain  (generic dns response: SERVFAIL)
| fingerprint-strings: 
|   NBTStat: 
|_    CKAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
<strong>| 123/udp open  ntp     NTP v3</strong></code></pre>

Notes:

* 53/DNS -- DNS Zone Transfer Failed
* 88/Kerberos -- <mark style="color:yellow;">Can we enumerate users and try AS-REPRoasting or Kerberoasting</mark>
* 445/SMB -- <mark style="color:yellow;">SMBMAP and SMBCLIENT</mark>
* 3268/LDAP -- <mark style="color:yellow;">Ldap enumeration -- JXplorer?</mark>
* 5985/WinRM -- If we have valid creds, we will be able to WinRM
* 123/udp NTP -- <mark style="color:yellow;">NTP v3</mark>???

## Enumeration

### Port 53 - DNS&#x20;

* Zone Transfer Attempted -- FAILED

```
dig axfr 10.129.92.248
```

### Port 88&#x20;

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
