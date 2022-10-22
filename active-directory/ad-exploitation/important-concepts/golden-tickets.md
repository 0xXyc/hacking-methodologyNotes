---
description: God-Mode Activated
---

# Golden Tickets

## Golden Ticket Attacks

This will cover how they work and how to defend against them.

### Impact

A <mark style="color:yellow;">successful</mark> Golden Ticket attack will <mark style="color:yellow;">give the attacker access to an organization's entire Active Directory domain</mark>. This includes access to all computers, files, folders, shares, and Domain Controllers. <mark style="color:red;">The attacker can impersonate anyone</mark>.

## How it works

The Golden Ticket attack stems from a vulnerability in Kerberos.

When a user authenticates to use services or access file servers they do not authenticate directly through themselves. Instead, the <mark style="color:yellow;">Kerberos Key Distribution Center (KDC)</mark> functions as a trusted authentication service. Every Domain Controller in an AD domain runs a KDC service.

At the time of authentication, the KDC will issue a <mark style="color:yellow;">Ticket Granting Ticket (TGT</mark>), which will include a unique key when the user needs access to resources. Before sending a TGT, the KDC encrypts it using the password hash for a special account which is the <mark style="color:yellow;">KRBTGT</mark> account.

* TGT's are always encrypted with the KRBTGT password hash

## Exploitation Process

Attackers can <mark style="color:yellow;">bypass the KDC and create TGT's at will</mark> in order to get access to anything desired.&#x20;

In order to do this, you need the following:

* <mark style="color:yellow;">The FQDN of the domain</mark>
* <mark style="color:yellow;">The SID of the domain</mark>
* <mark style="color:yellow;">The username of the account to be impersonated</mark>
* <mark style="color:yellow;">The KRBTGT password hash</mark>

How do you get the KRBTGT hash? You can do this in a couple ways:

* <mark style="color:yellow;">Stealing the NTDS.DIT file</mark>: This file is a database that stores all AD data that includes the password hashes for all users in the domain. This file is stored in C:\Windows\NTDS\NTDS.DIT
* <mark style="color:yellow;">Compromising a workstation</mark>: Gain a foothold on a domain endpoint and you can attempt to find valid credentials in memory or on the disk for use.
* <mark style="color:yellow;">Mimikatz</mark>: This is a powerful tool that allows an attacker to steal credentials from Windows systems.
* <mark style="color:yellow;">DCSync Attack</mark>: This is an attack where a user can have DCSync permissions and you can pretend to be a DC and request password hashes from a legitimate DC. This can also be done with Mimikatz.

#### Main Point:

<mark style="color:yellow;">If you obtain the KRBTGT hash, you can do anything you want, it is indeed a Golden Ticket.</mark>

## Mitigations

* Change the KRBTGT password regularly.
  * Secure Technical Implementation Guide (STIG) recommends changing it every 180 days.
* Apply Principle of Least Privilege (PoLP) in your domain. Ensure that you are minimizing the number of accounts that can access the KRBTGT hash.
