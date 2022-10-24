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

### Identifying Hosts

