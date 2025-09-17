---
description: 09/17/2025
---

# Data Protection API (DPAPI)

## Motivation/What is it?

The Data Protection API (DPAPI) is a component built into Windows that provides a means for encrypting and decrypting data "blobs".

It uses cryptographic keys that are tied to either a specific user or computer and allows both native Windows functionality and third-party applications to protect/unprotect data transparently to the user.

DPAPI is used by the Windows Credential Manager to store saved secrets such as RDP credentials, and by third-party applications like Google Chrome to store website credentials.

## Credential Manager

