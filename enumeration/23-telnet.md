---
description: >-
  Telnet is an insecure-in-nature, remote communications protocol. The
  insecurity is due to it being implemented without the use of encryption. It
  was later replaced by SSH.
---

# 🙀 23/Telnet

## Banner Grabbing:

```python
nc -nv <IP> 23
```

## Nmap Enumeration:

```python
nmap -n -sV -Pn --script "*telnet* and safe" -p 23 <IP>
```
