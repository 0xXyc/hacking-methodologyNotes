---
description: 09-24-22
---

# Intelligence (Medium)

### Improved skills

* Skill 1
* Skill 2

### Used tools

* Nmap -- NmapAutomator.sh

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-server-header: Microsoft-IIS/10.0
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-09-25 03:44:15Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         Microsoft Windows RPC
49707/tcp open  msrpc         Microsoft Windows RPC
49714/tcp open  msrpc         Microsoft Windows RPC
53972/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Enumerated UDP ports:

```
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
389/udp open  ldap
```

Notes:

* 53/DNS - Added <mark style="color:yellow;">intelligence.htb</mark> to <mark style="color:yellow;">/etc/hosts</mark>. Attempted zone transfer with no luck.
* 80/HTTP - Visual Inspection
* 88/Kerberos -&#x20;
* 135/RPC -&#x20;
* 389/LDAP - Nothing
* 445/SMB - Nothing null, nothing guest
* 5985/WinRM - <mark style="color:yellow;">Initial entry point? -- Creds needed</mark>

### Visual Inspection

* Input field for email subscription
* Email of contact@intelligence.htb -- only works with an email
  * Intercept w/ burp
  * There is no request when sending an email for the subscription
* There are images on the site as well
  * Two PDFs -- run exiftool

## Enumeration

### Port 80 (IIS 10.0)

Dirsearch:

Wordlists used:&#x20;

* Default dirsearch wordlist
* common.txt

```
[16:48:57] 301 -  157B  - /documents  ->  http://intelligence.htb/documents/
[16:49:00] 200 -    7KB - /index.html
```

* <mark style="color:yellow;">/Documents</mark> gives us a 403 forbidden
  * Attempt 403 bypass
  * None of my bypass attempts worked

Source Code:

* Nothing seemed to be in the source code

### Enum4Linux

* I was unable to obtain anything interesting from this

```
Domain Sid: S-1-5-21-4210132550-3389855604-3437519686
```

### SMB&#x20;

smbmap:

```
smbmap -H 10.129.95.154
```

* Nothing

smbclient:

```
smbclient -L "//10.129.95.154/" -U "guest"%
```

* Status: Account\_Disabled

### LDAP

* Attempted to do recon on LDAP, however, I could not communicate with the LDAP server

### VHOST/Subdomain Bruteforce

gobuster:

```
gobuster vhost -u http://10.129.95.154/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 30 | grep 200
```

* Unable to find anything

### Downloaded Documents

* 2020-01-01-upload.pdf
  * <mark style="color:yellow;">William.Lee</mark>
* 2020-12-15-upload.pdf
  * <mark style="color:yellow;">Jose.Williams</mark>

We find potentially useful metadata in here

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
