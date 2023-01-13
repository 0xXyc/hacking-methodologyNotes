---
description: Preventing MITM's one pin at a time
---

# SSL Certificate Pinning

## What is certpinning?

Certificate pinning is a security technique that was originally devised as a means to mitigate Man-in-The-Middle (MiTM) attacks. The application accepts only authorized or "pinned" certificates for authentication of client-server connections.

Any attempted secure connection requests utilizing non-pinned certificates are refused.

This technique is usually only found in very expensive, lucrative applications or other pieces of technology because of how expensive it is to implement this technique.&#x20;

## How does it work?

The idea is actually very simple; it is accomplished by correlating a host, from which connections will be sent, with the predetermined certificate.
