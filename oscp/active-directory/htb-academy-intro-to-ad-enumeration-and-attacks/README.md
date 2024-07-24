---
description: 'Start Date: 10-23-22'
---

# HTB Academy: Intro to AD Enumeration & Attacks

Remember: "AD is immense, always think outside the box and think wild."

## Active Directory Structure

AD is built on the x.500 and LDAP protocols

* <mark style="color:yellow;">AD allows for centralized management of an organization's resources that include users, computers, groups, network devices, file shares, group policies, devices, and trusts.</mark>&#x20;
* As a result, AD is essentially a database
* It provides <mark style="color:yellow;">Authentication</mark>, <mark style="color:yellow;">Accounting</mark>, and <mark style="color:yellow;">Authorization</mark>

## So, What is the Importance of AD Security?

* Essentially, AD is EVERYWHERE
  * It does not look like it is going anywhere soon
* Just within the last two years (2022 currently), Microsoft has had over 2,000 different vulnerabilities tied to a CVE

### Why is it so hard to protect?

* AD has <mark style="color:yellow;">many services</mark>
* <mark style="color:yellow;">The main purpose is to make information easy to find and access</mark>
  * This is a dream for hackers
  * This makes it <mark style="color:red;">very difficult to correctly manage and harden</mark>

### What will we cover?

* How to identify, enumerate, and exploit weaknesses associated with AD

AD tools covered:

* Sysinternals
* WMI
* DNS
* etc.

Attack Vectors:

* Password Spraying
* Kerberoasting
* AS-REProasting
* Responder
* Kerbrute
* BloodHound
* etc.

### What is the best way to compromise AD?

* You may often find yourself in a network that provides no clear path to a foothold through a remote exploit&#x20;
* <mark style="color:yellow;">However, we are in an AD network which has many ways that could lead to exploitation and give us a foothold</mark>
* You must be <mark style="color:yellow;">confident with living off of the land</mark>&#x20;

## Examples of Foothold to Full Compromise

### Scenario 1: Waiting on an Admin to Access a Share

#### Compromise

* You were able to gain access to a single host and gain local administrative access
* <mark style="color:yellow;">Since this host was domain-joined, you now have access to enumerate the domain</mark>
* <mark style="color:yellow;">Since you have credentials</mark> and there are <mark style="color:yellow;">Service Principal Names (SPNs)</mark> on the network, you can <mark style="color:yellow;">Kerberoast</mark>
* <mark style="color:yellow;">Successful Kerberoast attacks will lead to retrieval of Ticket Granting Service (TGS) tickets for associated accounts.</mark>
  * This will <mark style="color:yellow;">dump the NTLM hash associated with that account</mark>
* You can then <mark style="color:yellow;">crack this hash if the password is weak enough</mark> or if you have it in your password list
* <mark style="color:red;">This hash was able to be cracked with Hashcat and it gives you access to a file share</mark>
* With new access to this file share, you can authenticate to it and sit in the middle of the transmissions with Responder

#### Low-privileged user -> Domain Admin

* <mark style="color:red;">After awhile, you may notice that you can get NTLM hashes v2 or v1 hashes</mark>
  * Remember v2 you cannot pass the hash but v1, you can!
* <mark style="color:red;">The hash of this user was the admin's!</mark>

### Scenario 2: Spraying and Praying

#### Compromise

* <mark style="color:yellow;">Password spraying is a very effective way of gaining a foothold in a domain</mark>
* However, you must be <mark style="color:yellow;">conscious of lockout periods and thresholds</mark>
* This attack stems from an <mark style="color:yellow;">SMB NULL session that can be enumerated using the Enum4Linux</mark> tool&#x20;
  * This will <mark style="color:yellow;">retrieve all users on the domain as well as the password policy</mark>
* <mark style="color:red;">Equipped with this information, you can ensure a proper way of password spraying without locking the accounts out!</mark>

#### Low-privileged user -> Domain Admin

* <mark style="color:red;">After finding a valid password for the list of users, you can then use a tool known as Rubeus and extract a Kerberos Ticket Granting Ticket (TGT) for that domain user</mark>
* Then, you can perform a <mark style="color:yellow;">pass-the-ticket (PTT)</mark> and <mark style="color:red;">authenticate as this domain admin user</mark>!
