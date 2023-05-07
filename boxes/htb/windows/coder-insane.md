---
description: 05/05/2023
---

# Coder (Insane)

<figure><img src="../../../.gitbook/assets/Coder.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-05-05 20:56:17Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: coder.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc01.coder.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc01.coder.htb
| Issuer: commonName=coder-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-06-30T04:24:26
| Not valid after:  2023-06-30T04:24:26
| MD5:   7a8963c8621ca1c1a510001350d86800
|_SHA-1: 93e2a8bd85dc5a8767ffa4c131003634d14db0d7
|_ssl-date: 2023-05-05T20:57:16+00:00; +7h58m26s from scanner time.
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| ssl-cert: Subject: commonName=default-ssl/organizationName=HTB/stateOrProvinceName=CA/countryName=US
| Issuer: commonName=coder-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-11-04T17:25:43
| Not valid after:  2032-11-01T17:25:43
| MD5:   e5fea439d8356660c2b778e578a1244e
|_SHA-1: 733cf4571caafdaa8ad1e8fb0abc6fec7f932977
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_ssl-date: 2023-05-05T20:57:16+00:00; +7h58m26s from scanner time.
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap
|_ssl-date: 2023-05-05T20:57:16+00:00; +7h58m26s from scanner time.
| ssl-cert: Subject: commonName=dc01.coder.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc01.coder.htb
| Issuer: commonName=coder-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-06-30T04:24:26
| Not valid after:  2023-06-30T04:24:26
| MD5:   7a8963c8621ca1c1a510001350d86800
|_SHA-1: 93e2a8bd85dc5a8767ffa4c131003634d14db0d7
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: coder.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc01.coder.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc01.coder.htb
| Issuer: commonName=coder-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-06-30T04:24:26
| Not valid after:  2023-06-30T04:24:26
| MD5:   7a8963c8621ca1c1a510001350d86800
|_SHA-1: 93e2a8bd85dc5a8767ffa4c131003634d14db0d7
|_ssl-date: 2023-05-05T20:57:16+00:00; +7h58m26s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: coder.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc01.coder.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc01.coder.htb
| Issuer: commonName=coder-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-06-30T04:24:26
| Not valid after:  2023-06-30T04:24:26
| MD5:   7a8963c8621ca1c1a510001350d86800
|_SHA-1: 93e2a8bd85dc5a8767ffa4c131003634d14db0d7
|_ssl-date: 2023-05-05T20:57:16+00:00; +7h58m26s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  msrpc         Microsoft Windows RPC
49700/tcp open  msrpc         Microsoft Windows RPC
49704/tcp open  msrpc         Microsoft Windows RPC
58673/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h58m26s, deviation: 0s, median: 7h58m25s
| smb2-time: 
|   date: 2023-05-05T20:57:11
|_  start_date: N/A

```

Enumerated UDP ports:

```
```

Notes:

* Added IP and coder.htb to /etc/hosts

## Enumeration

### Port 80 - HTTP (Windows IIS)

#### Directory Bruteforce

```
dirsearch -u http://coder.htb
-- No results
```

#### Subdomain Enumeration

```
ffuf -u http://coder.htb -H "Host: FUZZ.coder.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

then had to add -fs 703 for proper filtering of results

ffuf -u http://coder.htb -H "Host: FUZZ.coder.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 703
```

### Port 445 - SMB

Enum4linux:

```
enum4linux -a coder.htb
```

#### Null Authentication

CME:

```
crackmapexec smb coder.htb -u '' -p '' --shares 
```

<figure><img src="../../../.gitbook/assets/image (65) (1).png" alt=""><figcaption></figcaption></figure>

smbclient:

```
smbclient //coder.htb/Development
Password for [WORKGROUP\pocydon]:
Try "help" to get a list of possible commands.
smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> cd Migrations\
smb: \Migrations\> mget *
```

* This will take some time
* I tried to grab the other directory ( \Temporary Projects) but it didn't let me (Object\_Not\_Found)
* There are a ton of files, my guess is this box is going to require some sort of static code analysis

#### Files grabbed from NULL SMB Share

{% code overflow="wrap" %}
```
adcs_reporting/  bootstrap-template-master/  Cachet-2.4/  kimchi-master/  teamcity_test_repo/
```
{% endcode %}

#### Notes

* Possible version number <mark style="color:yellow;">Cachet-2.4</mark>
* Basic enumeration of kimchi shows that it was last updated in 2016

adcs\_reporting:

Get-ADCS\_Report.ps1

Note: This is a snippet. I still need to analyze the file further.

```powershell
#requires -version 2
<#
.SYNOPSIS
  Collect ADCS information, export results into a HTML report and send it by mail

.DESCRIPTION
  Collect ADCS information locally, and export results into a HTML report (failed and pending requests, plus expired certificates) and send it by mail

.INPUTS
  <None>

.OUTPUTS
   Create HTML  report, similar to $ScriptDir\[SCRIPTNAME]_[YYYY_MM_DD]_[HHhMMmSSs].html
   
   
.NOTES
  Version:        1.1
  Author:         ALBERT Jean-Marc
  Creation Date:  01/09/2015
  Purpose/Change: 1.0 - 2015.09.01 - ALBERT Jean-Marc - Initial script development
                  1.1 - 2016.10.31 - ALBERT Jean-Marc - Replace alias used instead of command name, and " per ', and multiple enhancement


  
.SOURCES
  https://www.shellandco.net/monitor-certificate-expiration/
  http://pspki.codeplex.com/
  http://stackoverflow.com/questions/16542729/send-multiple-outputs-to-the-same-html-file-in-powershell

$emailFrom = 'pkiadmins@coder.htb'
$emailCC = 'e.black@coder.htb'
$emailTo = 'itsupport@coder.htb'
$smtpServer = 'smtp.coder.htb'

```

We can see that we have obtained some users here:

users.txt:

```
pkiadmins
e.black
itsupport
```

Going to attempt AS-REP roasting as a low-hanging fruit.

#### Impacket-GetNPUsers

```
impacket-GetNPUsers coder.htb/ -dc-ip 10.129.229.10 -no-pass -usersfile users.txt
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User e.black doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
```

The user e.black is a valid user, but does not have Kerberos Pre-Authentication Disabled.

Moving on to a different file to analyze.

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

