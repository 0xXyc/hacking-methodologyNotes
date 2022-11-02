---
description: It's like roasting marshmallows on a fire... except we're using SPN's instead
---

# Kerberoasting - From Linux

## Introduction

Kerberoasting is a <mark style="color:yellow;">lateral movement/privilege escalation technique</mark> in Active Directory environments.

* It should be highly noted that <mark style="color:yellow;">this attack targets Service Princial Names (SPNs) Accounts</mark>

### What do you need to perform this?

* An <mark style="color:yellow;">account's cleartext password or NTLM hash</mark>
* A shell from a domain user
* <mark style="color:yellow;">SYSTEM</mark> level access on a domain-joined account

### Service Accounts

* Domain accounts running services are often local admins
* If not, they are typically highly privileged domain accounts
* Always be sure to identify what privileges are granted across multiple servers and hosts on the domain
* Finding SPNs associated with high privileged accounts in Windows is very common
* Retrieving a Kerberos ticket for an account with a SPN does not by itself allow you to execute commands in the context of that account
* With that said, the TGS-REP or ticket, is encrypted with the NTLM hash&#x20;
  * This means that it can possibly be taken offline and cracked with Hashcat&#x20;

\-----------------------------------------------------------------------------------------------------

## Performing Kerberoasting

This attack can be performed with the following tools:

* Impacket's <mark style="color:yellow;">GetUserSPNs</mark> from a non-domain joined Linux host
* A combination of <mark style="color:yellow;">setspn.exe</mark>, <mark style="color:yellow;">powershell</mark>, and <mark style="color:yellow;">Mimikatz</mark>
* From Windows, we can use <mark style="color:yellow;">PowerView</mark>, <mark style="color:yellow;">Rubeus</mark>, and other PowerShell scripts

### Reality

<mark style="color:yellow;">Obtaining a TGS ticket from a Kerberoast attack will NOT guarantee you a set of valid credentials and the ticket must still be taken offline and cracked with Hashcat to obtain the cleartext password. TGS tickets take longer to crack than NTLM hashes!</mark>

Also, Kerberoasting can be a great way to move laterally or vertically in a domain but it does not guarantee us any level of access.

* There are times where you can perform the attack, retrieve the TGS keys, crack them offline, and we obtain Domain Admin access or obtain credentials that will aid us in our path there!

\-----------------------------------------------------------------------------------------------------

## Kerberoasting with GetUserSPNs

Before we start, we need to make sure that we have the following:

* Valid domain credentials (cleartext or NTLM hash)
* A shell in the context of a domain user, or account such as SYSTEM
*
