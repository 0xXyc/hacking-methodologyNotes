---
description: 05-05-2023
---

# Absolute (Insane)

<figure><img src="../../../.gitbook/assets/Absolute.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Absolute
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-05-05 22:10:43Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.absolute.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.absolute.htb
| Issuer: commonName=absolute-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T08:14:24
| Not valid after:  2023-06-09T08:14:24
| MD5:   bfc067aca80d4a43c76770e3daac4089
|_SHA-1: d2020dbd811c7e36ad9e120be6eba1108695f3f7
|_ssl-date: 2023-05-05T22:11:48+00:00; +7h00m00s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-05-05T22:11:47+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=dc.absolute.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.absolute.htb
| Issuer: commonName=absolute-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T08:14:24
| Not valid after:  2023-06-09T08:14:24
| MD5:   bfc067aca80d4a43c76770e3daac4089
|_SHA-1: d2020dbd811c7e36ad9e120be6eba1108695f3f7
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.absolute.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.absolute.htb
| Issuer: commonName=absolute-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T08:14:24
| Not valid after:  2023-06-09T08:14:24
| MD5:   bfc067aca80d4a43c76770e3daac4089
|_SHA-1: d2020dbd811c7e36ad9e120be6eba1108695f3f7
|_ssl-date: 2023-05-05T22:11:48+00:00; +7h00m00s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: absolute.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-05-05T22:11:47+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=dc.absolute.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.absolute.htb
| Issuer: commonName=absolute-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-06-09T08:14:24
| Not valid after:  2023-06-09T08:14:24
| MD5:   bfc067aca80d4a43c76770e3daac4089
|_SHA-1: d2020dbd811c7e36ad9e120be6eba1108695f3f7
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49698/tcp open  msrpc         Microsoft Windows RPC
49701/tcp open  msrpc         Microsoft Windows RPC
49706/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-05-05T22:11:41
|_  start_date: N/A
|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m59s
```

Enumerated UDP ports:

```
```

Notes:

* Added absolute.htb to /etc/hosts

## Enumeration

### Port 80 - HTTP (IIS)

#### Visual Inspection

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

* Analyzed source code

#### Directory Bruteforce

```
dirsearch -u http://absolute.htb
```

* No results

#### Subdomain Enumeration

```
ffuf -u http://absolute.htb -H "Host: FUZZ.absolute.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 2909
```

* No results

### Port 445 - SMB

#### Enum4Linux

```
enum4linux -a absolute.htb
```

* Only result was the SMB NULL auth was possible

#### CrackMapExec

```
crackmapexec smb absolute.htb -u '' -p '' --shares

SMB         absolute.htb    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:absolute.htb) (signing:True) (SMBv1:False)
SMB         absolute.htb    445    DC               [+] absolute.htb\: 
SMB         absolute.htb    445    DC               [-] Error enumerating shares: STATUS_ACCESS_DENIED
```

Boom, we see that we have NULL access to this SMB share.

#### smbclient

```
smbclient -L //absolute.htb/ -U '' -N

	Sharename       Type      Comment
	---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to absolute.htb failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

smbclient \\\\absolute.htb\\public -U 10.129.228.64 -N
session setup failed: NT_STATUS_LOGON_FAILURE
```

Unfortunately, I was unable to get anything with this for some reason. Paranoid that I was messing up, I confirmed this finding with a few more tools.

#### smbmap

```
smbmap -H absolute.htb -d absolute.htb -L
```

Still nothing.

### RPC

#### rpcclient

```
rpcclient -U '' -N absolute.htb
```

I was unable to obtain any information from RPC without creds.

### LDAP

#### ldapsearch

```
ldapsearch -LLL -x -H ldap://absolute.htb -b '' -s base '(objectclass=\*)'
dn:
domainFunctionality: 7
forestFunctionality: 7
domainControllerFunctionality: 7
rootDomainNamingContext: DC=absolute,DC=htb
ldapServiceName: absolute.htb:dc$@ABSOLUTE.HTB
isGlobalCatalogReady: TRUE
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: DIGEST-MD

--- cut --- 

subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=absolute,DC=htb
serverName: CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configurat
 ion,DC=absolute,DC=htb
schemaNamingContext: CN=Schema,CN=Configuration,DC=absolute,DC=htb
namingContexts: DC=absolute,DC=htb
namingContexts: CN=Configuration,DC=absolute,DC=htb
namingContexts: CN=Schema,CN=Configuration,DC=absolute,DC=htb
namingContexts: DC=DomainDnsZones,DC=absolute,DC=htb
namingContexts: DC=ForestDnsZones,DC=absolute,DC=htb
isSynchronized: TRUE
highestCommittedUSN: 168078
dsServiceName: CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN
 =Sites,CN=Configuration,DC=absolute,DC=htb
dnsHostName: dc.absolute.htb
defaultNamingContext: DC=absolute,DC=htb
currentTime: 20230505224841.0Z
configurationNamingContext: CN=Configuration,DC=absolute,DC=htb
```

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

