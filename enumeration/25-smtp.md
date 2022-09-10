---
description: >-
  Simple Mail Transfer Protocol (SMTP) is a mailing protocol that allows hosts
  to send/receive emails.
---

# 📧 25/SMTP

## Banner Grabbing:

```python
nc -vn <IP> 25
```

## Nmap Enumeration:

```python
nmap -p25 --script smtp-commands <IP>
```

## SMTP User Enumeration Bruteforce

```
smtp-user-enum
```

## Reference

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp" %}
