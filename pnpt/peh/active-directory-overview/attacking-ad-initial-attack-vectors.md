---
description: The fun part
---

# Attacking AD: Initial Attack Vectors

## Great Resource

{% embed url="https://adam-toscher.medium.com/top-five-ways-i-got-domain-admin-on-your-internal-network-before-lunch-2018-edition-82259ab73aaa" %}

## LLMNR Poisoning

* <mark style="color:yellow;">Link Local Multicast Name Resolution</mark>
* This is a protocol that is used to <mark style="color:yellow;">identify hosts when DNS fails to do so</mark>
* Previously NBT-NS
* The <mark style="color:yellow;">flaw</mark> is that the services utilize a <mark style="color:yellow;">user's username and NTLMv2 when appropriately responded to</mark>

Example:

Victim and Server

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### How to exploit

Step 1: Run Responder

```
sudo responder -I tun0 -rdwv
```

* NOTE: This tool should seriously be the first thing that you run on an engagement
  * Especially during the morning hours when users are authenticating for the first time that day
  * Lunch is also a great time
* You should theoretically receive a hash once an even occurs

Step 2: Wait

Step 3: Obtain hash and proceed to Pass-The-Hash or take offline and crack!

Step 4: Password Cracking

```
hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt
```

* Run these on your host for fast speeds

Finding Hashcat format:

```
hashcat --help | grep NTLM
```
