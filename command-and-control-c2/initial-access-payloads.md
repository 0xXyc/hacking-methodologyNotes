---
description: 09/01/2025
---

# Initial Access Payloads

## Initial Access Payload Approach/Classification&#x20;

Sending a payload to the phished user(s) is a direct way of gaining a foothold as the payload will be directly executed on their system(s).

<mark style="color:yellow;">**There are two options for delivering a payload (at a high-level)**</mark>**:**

1. **Send a URL where a payload can be downloaded**
2. **Attach the payload to the phishing email**

## Zone Identification

Any file downloaded via a browser (outside of a trusted zone) will be tainted with the _**"Mark of the Web"**_ (MOTW).

This means that this particular data stream gets embedded into the file which says that its file origins come from an untrusted location.&#x20;

**Of course, we can obtain this value and read it using PowerShell:**

```
gc .\test.txt -Stream Zone.Identifier
[ZoneTransfer]
ZoneId=3
HostUrl=http://nickelviper.com/test.txt
```

### Possible Zones

* `0` = Local Computer
* `1` = Local Intranet
* `2` = Trusted Sites
* `3` = Internet
* `4` = Restricted Sites

## Considerations

_**Files marked MOTW are handled with additional security scrutiny.**_

As a result, you may be familiar with Both Windows _**SmartScreen**_ and _**Office Protected View**_.

Although they _**often**_ do not stop files from being opened/executed, but require the user to click through additional warnings; overall lowering the likelihood of your phish being successful.

<mark style="color:yellow;">Files that are emailed "internally" via a compromised Exchange mailbox are</mark> <mark style="color:yellow;"></mark>_<mark style="color:yellow;">**not**</mark>_ <mark style="color:yellow;"></mark><mark style="color:yellow;">tagged with a Zone Identifier!</mark>

#### MS Office Considerations

If MS Office "block macros downloaded from the Internet" option is enabled, a user cannot run a macro-enabled document even if they wanted to.

This will soon become a default setting (might already be).
