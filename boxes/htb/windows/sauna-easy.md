---
description: 09-26-22
---

# Sauna (Easy)

### Improved skills

* Skill 1
* Skill 2

### Used tools

* Nmap -- NmapAutomator.sh

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

* We are able to grab some potential usernames from the content of the site

### Port 389 - LDAP&#x20;

Enum4Linux

```
enum4linux -a 10.129.148.212
```

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
