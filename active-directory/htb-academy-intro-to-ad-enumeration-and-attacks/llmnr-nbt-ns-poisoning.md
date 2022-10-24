# LLMNR/NBT-NS Poisoning

## Introduction

### Password Spraying and Poisoning

* The goal of these attacks are to acquire a valid set of credentials for a domain user account
* In return, we will be granted a foothold in the domain and we can begin the next phase of enumeration with credentials!

## LLMNR & NBT-NS

LLMNR: Link-Local Multicast Name Resolution

NBT-NS: NetBIOS Name Service

* Both are ways that serve as alternate methods of host identification when DNS fails
* We can attack these protocols by using the Man-in-The-Middle technique&#x20;
* <mark style="color:yellow;">The reason that LLMNR and NBT-NS are such a target is because any host on the network can reply to name resolution requests which is why we can poison these requests with Responder</mark>
* The poisoning works because the attacker pretends to know the location of the requested host
* We can then capture their NTLM hash and take it offline and crack it or attempt to Pass The Hash if the NTLM version is NTLMv1 and not NTLMv2
* The captured authentication request can also be relayed (Impacket-ntlmrelayx) to access another host or to be used against a different protocol such as LDAP on the same host

### Overall

* A mixture of LLMNR & NBT-NS in an environment will lead to a bad day
* Especially when those two network protocols are associated with a lack of SMB signing
* This can lead to SMB relay attacks; granting the attacker administrative access on the hosts in a domain
