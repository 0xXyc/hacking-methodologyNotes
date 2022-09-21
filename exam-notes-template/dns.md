---
description: DNS Enumeration workflow -- can we gain an increased attack surface?
---

# DNS

## Methodology

1. Web server running? Visit IP address in browser; does it give you a subdomain? If so, add to <mark style="color:yellow;">/etc/hosts</mark>
2. Attempt zone transfer
3. dnsrecon
4. subdomain enumeration
5. vhost enumeration

## Enumeration

