---
description: 09/17/2025
---

# Data Protection API (DPAPI)

## Motivation/What is it?

The Data Protection API (DPAPI) is a component built into Windows that provides a means for encrypting and decrypting data "blobs".

It uses cryptographic keys that are tied to either a specific user or computer and allows both native Windows functionality and third-party applications to protect/unprotect data transparently to the user.

DPAPI is used by the Windows Credential Manager to store saved secrets such as RDP credentials, and by third-party applications like Google Chrome to store website credentials.

## Credential Manager

Upon initially reading up about the Windows Credential Manager may end up being a bit confusing at first.

You're going to find both terms "Vaults" and "Credentials". So, what's the difference?

### Vaults

It essentially holds records of encrypted credentials and a reference to the encrypted blobs.

Windows has two vaults:

1. Web Credentials (for storing browser creds)
2. Windows Credentials (for storing credentials saved by `mstsc`, etc.)

### Credentials

This is the actual encrypted blob(s).

### Enumerating User's Vaults

To achieve this, we can use the <mark style="color:yellow;">`vaultcmd`</mark> tool.

```
beacon> run vaultcmd /list

Currently loaded vaults:
	Vault: Web Credentials
	Vault Guid:4BF4C442-9B8A-41A0-B380-DD4A704DDB28
	Location: C:\Users\bfarmer\AppData\Local\Microsoft\Vault\4BF4C442-9B8A-41A0-B380-DD4A704DDB28

	Vault: Windows Credentials
	Vault Guid:77BC582B-F0A6-4E15-4E80-61736B6F3B29
	Location: C:\Users\bfarmer\AppData\Local\Microsoft\Vault

beacon> run vaultcmd /listcreds:"Windows Credentials" /all

Credentials in vault: Windows Credentials

Credential schema: Windows Domain Password Credential
Resource: Domain:target=TERMSRV/sql-2.dev.cyberbotic.io
Identity: SQL-2\Administrator
Hidden: No
Roaming: No
Property (schema element id,value): (100,2)
```

Another is to use <mark style="color:yellow;">`seatbelt`</mark> along with the <mark style="color:yellow;">`-group=user`</mark> parameter, or more specifically, the <mark style="color:yellow;">`WindowsVault`</mark> parameter!

```
beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsVault

====== WindowsVault ======

  Vault GUID     : 4bf4c442-9b8a-41a0-b380-dd4a704ddb28
  Vault Type     : Web Credentials
  Item count     : 0

  Vault GUID     : 77bc582b-f0a6-4e15-4e80-61736b6f3b29
  Vault Type     : Windows Credentials
  Item count     : 1
      SchemaGuid   : 3e0e35be-1b77-43e7-b873-aed901b6275b
      Resource     : String: Domain:target=TERMSRV/sql-2.dev.cyberbotic.io
      Identity     : String: SQL-2\Administrator
      PackageSid   : (null)
      Credential   : 
      LastModified : 9/6/2022 9:12:12 AM
```

Based on this, we now know that the user has saved credentials for the local administrator account on <mark style="color:yellow;">`SQL-2`</mark>. The encrypted credentials themselves are stored in the users' <mark style="color:yellow;">`\Credentials`</mark> directory.

```
beacon> ls C:\Users\bfarmer\AppData\Local\Microsoft\Credentials
[*] Listing: C:\Users\bfarmer\AppData\Local\Microsoft\Credentials\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 468b     fil     09/06/2022 10:34:22   6C33AC85D0C4DCEAB186B3B2E5B1AC7C
 10kb     fil     08/30/2022 08:42:59   DFBE70A7E5CC19A398EBF1B96859CE5D
```

<mark style="color:yellow;">`seatbelt`</mark> can also enumerate them using the <mark style="color:yellow;">`WindowsCredentialFiles`</mark> parameter.

```
beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsCredentialFiles

====== WindowsCredentialFiles ======

  Folder : C:\Users\bfarmer\AppData\Local\Microsoft\Credentials\

    FileName     : 6C33AC85D0C4DCEAB186B3B2E5B1AC7C
    Description  : Local Credential Data

    MasterKey    : bfc5090d-22fe-4058-8953-47f6882f549e
    Accessed     : 9/6/2022 10:50:44 AM
    Modified     : 9/6/2022 10:50:44 AM
    Size         : 468

    FileName     : DFBE70A7E5CC19A398EBF1B96859CE5D
    Description  : Local Credential Data

    MasterKey    : bfc5090d-22fe-4058-8953-47f6882f549e
    Accessed     : 9/6/2022 10:50:44 AM
    Modified     : 9/6/2022 10:50:44 AM
    Size         : 11036
```

{% hint style="info" %}
As of 09/17/2025, as far as I'm aware, there is no way to know which credential blob belong to which vault; and by extension, if a vault has multiple entries, which credential blob corresponds to which. This just ends with us having to decrypt each one in turn.
{% endhint %}

<mark style="color:yellow;">`seatbelt`</mark> also provides the GUID of the master key used to encrypt the credentials.

The master keys are stored in the users' roaming "Protect" directory. But guess what... they're also encrypted.

```
beacon> ls C:\Users\bfarmer\AppData\Roaming\Microsoft\Protect\S-1-5-21-569305411-121244042-2357301523-1104
[*] Listing: C:\Users\bfarmer\AppData\Roaming\Microsoft\Protect\S-1-5-21-569305411-121244042-2357301523-1104\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 740b     fil     08/15/2022 17:34:31   bfc5090d-22fe-4058-8953-47f6882f549e
 928b     fil     08/15/2022 17:34:31   BK-DEV
 24b      fil     08/15/2022 17:34:31   Preferred
```

_<mark style="color:yellow;">**So we must decrypt the master key first to obtain the actual AES128/256 encryption key and then use that key to decrypt the credentialed blob**</mark>_.&#x20;

**There are two ways of doing this:**

1. _**The first is only possible is you have local admin access on the machine and if the key is cached in LSASS**_. _<mark style="color:red;">**It will not be in the cache if the user has not recently accessed/decrypted the credential**</mark>_.

