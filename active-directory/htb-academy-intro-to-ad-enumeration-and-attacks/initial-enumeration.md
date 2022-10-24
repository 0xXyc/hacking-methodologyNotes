# Initial Enumeration

## Introduction

* This test will be simulated on an "<mark style="color:yellow;">assumed breach</mark>" pentest where the attacker is already inside the network
* This is a common way that a client prefers an assessment

Others include:

* SSH'ing into their environment via bastion/jump host
* Physically present in the office with a laptop plugged into an ethernet port
* VDI

## Tasks

* Enumerate the internal network
* Identify hosts
* Critical services
* Potential opportunities for a foothold

Note: <mark style="color:yellow;">This can include active and passive measures to identify users, hosts, and vulnerabilities we may be able to take advantage of to further our access</mark>

* <mark style="color:yellow;">Be sure to document all findings for later use!!!</mark>
* We will start from the Linux attack host without domain user credentials (it is common to start off this way because it provides a more realistic approach)
* <mark style="color:red;">It is common for the customer to provide you with access to a domain-joined host or a set of credentials for the network to expedite testing and allow us to cover as much ground as possible</mark>

## What to look for

<mark style="color:yellow;">AS Users</mark>:&#x20;

* We are trying to enumerate VALID user accounts we can target for password spraying

<mark style="color:yellow;">AD Joined Computers</mark>:&#x20;

* Identify Domain Controllers, File Servers, SQL Servers, Web Servers, Exchange Mail Servers, DB Servers, etc.&#x20;

<mark style="color:yellow;">Key Services</mark>:&#x20;

* Kerberos, NetBIOS, LDAP, and DNS

<mark style="color:yellow;">Vulnerable Hosts & Services</mark>:

* Can any of our findings result in a quick win?
* A.K.A. Low-hanging fruit

## Enumerating AD: Plan of Attack

<mark style="color:yellow;">Phase 1</mark>: <mark style="color:green;">Passive</mark> identification of hosts

<mark style="color:yellow;">Phase 2</mark>: <mark style="color:green;">Active validation</mark> of the results to learn more about these hosts

* What services are running, names, vulnerabilities, etc.

Once we know what hosts exist, we can probe them and attempt to extract useful information from them

* Hopefully, we can find a pair of credentials somewhere in order to leverage our access in the network

### Identifying Hosts - Wireshark and TCPdump PCAP

* <mark style="color:yellow;">What can we see on the network?</mark>

Tools: <mark style="color:red;">Wireshark</mark> or <mark style="color:red;">TCPdump</mark>

TCPdump:

```
sudo tcpdump -i tun0
```

* We can see lots of ARP traffic from existing hosts in the network

### Responder

* <mark style="color:yellow;">We can now use Responder to listen, analyze, and poison LLMNR, NBT-NS, and MDNS requests and responses</mark>

Syntax:

```
sudo responder -I ens224 -A
```

* Be sure to record all hosts that are detected in these outputs!

### ICMP Ping Sweep with FPing

* <mark style="color:yellow;">FPing is the same as the ping utility (still utilizes ICMP) to verify connectivity between hosts</mark>
* Except for one exception, it can provide that ICMP request to numerous IP addresses or a list

Active Hosts:

FPing Flags:

* \-a: Show alive targets
* \-s: Print stats at the end of the scan
* \-g: Generate a target list from the CIDR network
* \-q: do not show per-target results; quiet mode

Syntax:

```
fping -asgq 172.16.5.0/23
```

### Scanning Multiple Hosts

1. Gather list of live hosts with FPing

```
fping -asgq 172.16.5.0/23
```

&#x20; 2\. Place live hosts in hosts.txt

&#x20; 3\. Nmap hosts.txt

```
sudo nmap -sV -sC -iL hosts.txt -oN host-enum
```

### Kerbrute

{% embed url="https://github.com/ropnop/kerbrute/releases/tag/v1.0.3" %}
Pre-compiled binaries
{% endembed %}

1. wget one of the precompiled binaries
2. Add the tool to your path variable

```
echo $PATH

sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

* You can now type kerbrute and it will execute from anywhere on the system

Enumerating Users with Kerbrute:

```
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```
