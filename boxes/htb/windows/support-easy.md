---
description: 10-22-22
---

# Support (Easy)

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

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

* 53/DNS -- DNS Zone Transfer Failed -- support.htb -- TXT record: <mark style="color:yellow;">dc.support.htb hostmaster.support.htb</mark>
* 88/Kerberos -- <mark style="color:yellow;">Can we enumerate users and try AS-REPRoasting or Kerberoasting</mark>
* 445/SMB -- <mark style="color:yellow;">SMBMAP and SMBCLIENT</mark>
* 3268/LDAP -- <mark style="color:yellow;">Ldap enumeration -- JXplorer?</mark>
* 5985/WinRM -- If we have valid creds, we will be able to WinRM
* 123/udp NTP -- <mark style="color:yellow;">NTP v3</mark>???

## Enumeration

### Port 53 - DNS&#x20;

* Zone Transfer Attempted -- FAILED
  * /etc/hosts -- <mark style="color:yellow;">support.htb</mark>

Dig:

<pre><code><strong>dig axfr 10.129.92.248
</strong>
dig axfr support.htb</code></pre>

Dig TXT Query:

```
dig TXT @10.129.92.248 support.htb

; <<>> DiG 9.18.7-1-Debian <<>> TXT @10.129.92.248 support.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16728
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;support.htb.                   IN      TXT

;; AUTHORITY SECTION:
support.htb.            3600    IN      SOA     dc.support.htb. hostmaster.support.htb. 133 900 600 86400 3600

```

Two new subdomains?

* <mark style="color:yellow;">dc.support.htb</mark>
* <mark style="color:yellow;">hostmaster.support.htb</mark>
* Added to /etc/hosts

I can also achieve the same with:

```
dig @10.129.92.248 +short support.htb any

10.129.92.248
dc.support.htb.
dc.support.htb. hostmaster.support.htb. 134 900 600 86400 3600
```

dnsrecon:

```
dnsrecon -d support.htb

dnsrecon -d dc.support.htb

dnsrecon -d hostmaster.support.htb
```

### Port 88 - Kerberos

###

### 445 - SMB&#x20;

Map out shares with **smbmap (Null User):**

```
smbmap -H 10.129.92.248
[+] IP: 10.129.92.248:445       Name: support.htb
```

* We confirmed that we have access as a null user

List out shares with **smbclient:**

```
smbclient --no-pass -L //10.129.92.248

Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        support-tools   Disk      support staff tools
        SYSVOL          Disk      Logon server share
```

* \-L will list out the shares
* We notice <mark style="color:yellow;">support-tools</mark> -- This is not a standard share and is definitely something that we want to check out.
* Support staff tools

Getting an SMB session:&#x20;

```
smbclient --no-pass //10.129.92.248/support-tools

Try "help" to get a list of possible commands.
smb: \>
```

Enumerating Share:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">UserInfo</mark> looks the most interesting

Found something in UserInfo.exe:

* <mark style="color:yellow;">$5a280d0b-9fd0-4701-8f96-82e2f1ea9dfb</mark>

Found a public key token in UserInfo.exe.config:

* <mark style="color:yellow;">publicKeyToken="b03f5f7f11d50a3a"</mark>

Found interesting strings in <mark style="color:yellow;">System.Runtime.CompilerServices.Unsafe.dll</mark>

* Some strings resembled passwords

It appears that the support team is using an outdated version of Notepad++ version 3 (came out in 2007)



## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
