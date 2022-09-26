---
description: >-
  Domain Name Service (DNS) is a protocol that allows systems to resolve
  hostnames to IP addresses. It an be used to get information about subdomains
---

# 🔁 53/DNS

## Zone Transfer

```python
dig axfr @<DNS_IP> #Try zone transfer without domain
dig axfr @<DNS_IP> <DOMAIN> #Try zone transfer guessing the domain
fierce --domain <DOMAIN> --dns-servers <DNS_IP> #Will try toperform a zone transfer against every authoritative name server and if this doesn'twork, will launch a dictionary attack
```

#### More Info:

```python
**dig ANY @<DNS_IP> <DOMAIN>     #Any information
dig A @<DNS_IP> <DOMAIN>       #Regular DNS request
dig AAAA @<DNS_IP> <DOMAIN>    #IPv6 DNS request
dig TXT @<DNS_IP> <DOMAIN>     #Information
dig MX @<DNS_IP> <DOMAIN>      #Emails related
dig NS @<DNS_IP> <DOMAIN>      #DNS that resolves that name
dig -x 192.168.0.2 @<DNS_IP>   #Reverse lookup
dig -x 2a00:1450:400c:c06::93 @<DNS_IP> #reverse IPv6 lookup

#Use [-p PORT]  or  -6 (to use ivp6 address of dns)**
```

### DIG

```python
dig axfr cronos.htb @10.129.227.211
```

### DNSENUM:

```python
dnsenum zonetransfer.me
```

### NSLookup

```python
nslookup
> Default server: 127.0.0.1
Address: 127.0.0.1#53
> server 10.129.227.211
Default server: 10.129.227.211
Address:10.129.227.211#53
> 10.129.227.211
211.227.129.10.in-addr.arpa     name = ns1.cronos.htb
```

## Brute Forcing Sub Domains

```python
locate subdomains-1000.txt
dnsrecon -D subdomains-1000.txt -d <DOMAIN> -n <IP_DNS>
dnsrecon -d example.com
dnscan -d <domain> -r -w subdomains-1000.txt #Bruteforce subdomains in recursive way, <https://github.com/rbsec/dnscan>
```

## IPv6

```python
dnsdict6 -s -t <domain>
```

## Tools

NSLOOKUP

DIG

Dnsenum

dnsdict6

dnsrecon
