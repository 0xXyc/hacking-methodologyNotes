---
description: Preventing MITM's one pin at a time
---

# 📍 SSL Certificate Pinning

## What is certpinning?

Certificate pinning is a security technique that was originally devised as a means to mitigate Man-in-The-Middle (MiTM) attacks. The application accepts only authorized or "pinned" certificates for authentication of client-server connections.

Any attempted secure connection requests utilizing non-pinned certificates are refused.

This technique is usually only found in very expensive, lucrative applications or other pieces of technology because of how expensive it is to implement this technique.&#x20;

## How does it work?

The idea is actually very simple; it is accomplished by correlating a host, from which connections will be sent, with the predetermined certificate.

Once the association has been established between the host and the certificate, the relationship is formed, and the certificate has been pinned to the host.

Pinning adds an extra layer of security by making it more difficult for a would-be attacker to compromise a pin by placing themselves in the middle of the communication (hence Man-in-The-Middle). This can be especially important during the early development phase. The pinning process can also take place once an application first attempts to connect. This is known as "key continuity".
