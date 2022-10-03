---
description: Passive and Active recon
---

# Hunting Subdomains

<mark style="color:yellow;">The goal here is to increase our attack surface.</mark>

Can we find lucrative subdomains that will assist in leveraging our attacks?

Example:&#x20;

If we are targetting apple.com, what if there is a vulnerable web app on dev.apple.com?

## Sublist3r

Installation (apt):

```
apt install sublist3r -y
```

Syntax Usage:

```
sublist3r -d apple.com
```

## Certificate Fingerprinting

{% embed url="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjTyJjbhMT6AhUBElkFHaBsC_wQFnoECAwQAQ&url=https%3A%2F%2Fcrt.sh%2F&usg=AOvVaw0x-O9bg7JBcCIcp4fEMZV1" %}

On this site, the % is a wildcard that is placed before the domain.

Example:

```
%.apple.com
```

## OWASP Amass

This is a <mark style="color:yellow;">DNS enumeration tool</mark> that assists an attacker in mapping the attack surface of a target through means of external asset discovery.

GitHub Repo:

{% embed url="https://github.com/OWASP/Amass" %}

{% embed url="https://owasp.org/www-project-amass/" %}
