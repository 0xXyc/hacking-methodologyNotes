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

## Responder

Responder is capable of attacking the following networking protocols:

* LLMNR
* DNS
* MDNS
* NBNS
* DHCP
* ICMP
* HTTP
* HTTPS
* SMB
* LDAP
* WPAD
* WebDAV

### In Action

Responder Help Page:

```
responder -h
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.0.6.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

Usage: responder -I eth0 -w -r -f
or:
responder -I eth0 -wrf

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -A, --analyze         Analyze mode. This option allows you to see NBT-NS,
                        BROWSER, LLMNR requests without responding.
  -I eth0, --interface=eth0
                        Network interface to use, you can use 'ALL' as a
                        wildcard for all interfaces
  -i 10.0.0.21, --ip=10.0.0.21
                        Local IP to use (only for OSX)
  -e 10.0.0.22, --externalip=10.0.0.22
                        Poison all requests with another IP address than
                        Responder's one.
  -b, --basic           Return a Basic HTTP authentication. Default: NTLM
  -r, --wredir          Enable answers for netbios wredir suffix queries.
                        Answering to wredir will likely break stuff on the
                        network. Default: False
  -d, --NBTNSdomain     Enable answers for netbios domain suffix queries.
                        Answering to domain suffixes will likely break stuff
                        on the network. Default: False
  -f, --fingerprint     This option allows you to fingerprint a host that
                        issued an NBT-NS or LLMNR query.
  -w, --wpad            Start the WPAD rogue proxy server. Default value is
                        False
  -u UPSTREAM_PROXY, --upstream-proxy=UPSTREAM_PROXY
                        Upstream HTTP proxy used by the rogue WPAD Proxy for
                        outgoing requests (format: host:port)
  -F, --ForceWpadAuth   Force NTLM/Basic authentication on wpad.dat file
                        retrieval. This may cause a login prompt. Default:
                        False
  -P, --ProxyAuth       Force NTLM (transparently)/Basic (prompt)
                        authentication for the proxy. WPAD doesn't need to be
                        ON. This option is highly effective when combined with
                        -r. Default: False
  --lm                  Force LM hashing downgrade for Windows XP/2003 and
                        earlier. Default: False
  -v, --verbose         Increase verbosity.
```
