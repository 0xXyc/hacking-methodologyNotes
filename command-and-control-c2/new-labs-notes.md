---
description: 02/16/2026
---

# 🆕 New Labs Notes

**Pulling the band-aid off, we're finally taking this exam.**

## Beacon Listener Config

**Configuring Beacon Listeners:**

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Defense Evasion

Defense evasion from AV/EDR typically boils down to two capabilities we will need to keep in mind and bypass:

1. Signature (static detection) - detects the second payloads are dropped to disk, loaded into memory before/after execution.
2. Heuristics-based (behavioral)&#x20;

#### Goals

* The Artifact Kit to remove known indicators from the compiled artifacts (the _`.exe`_ and _`.dll`_).
* The Resource Kit to remove known indicators from the script artifacts (the _`.ps1`_).
* Malleable C2 to modify how the prepended loader loads Beacon into memory.
* Malleable C2 to modify post-exploitation behaviors.

### Artifact Kit

**The artifact kit what is it? The numbers, Mason, what do they mean??!?**

The &#x20;

**Default executables payloads templates:**

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

As we know, CS generates Beacon-specific shellcode that is then patched into one of these templates. It is then patched into one of these templates depending on what type of payload the operator asked for.&#x20;

* So essentially, the templates are nothing more than shellcode injectors
* They take that Beacon shellcode and inject it into memory

