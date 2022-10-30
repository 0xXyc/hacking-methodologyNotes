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

