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

## Service Accounts

* Domain accounts running services are often local admins
* If not, they are typically highly privileged domain accounts
* Always be sure to identify what privileges are granted across multiple servers and hosts on the domain
* Finding SPNs associated with high privileged accounts in Windows is very common
* Retrieving a Kerberos ticket for an account with a SPN does not by itself allow you to execute commands in the context of that account
* With that said, the TGS-REP or ticket, is encrypted with the NTLM hash&#x20;
  * This means that it can possibly be taken offline and cracked with Hashcat&#x20;

## Performing Kerberoasting

This attack can be performed in a number of ways depending on your position in the network:

* &#x20;

