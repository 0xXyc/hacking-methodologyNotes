---
description: Secure Shell (SSH) is an encrypted remote communications protocol.
---

# 🔐 22/SSH

## Checklist

* [ ] Attempt password checking with found credentials.
  * Note: This is CRUCIAL please check for password reuse as an easy win.
  * Brute force will 99.9% of the time NOT be the way in.

Banner Grabbing:

```python
nc -vn <IP> 22
```

Config Files to look out for:

```python
ssh_config
sshd_config
authorized_keys
ssh_known_hosts
known_hosts
id_rsa
```
