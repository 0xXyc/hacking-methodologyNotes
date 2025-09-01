---
description: 08/31/2025
---

# Payload Generation

## Motivation

Payloads can come in a variety of sizes and different formats. All suited for a variety of different situations; thus, use cases.

These can all be accessed within the **Payloads** menu.

<figure><img src="../.gitbook/assets/image (262).png" alt=""><figcaption></figcaption></figure>

## HTML Application

This produces an `.hta` file (which is typically delivered via a browser — client-side attack involving social engineering). This uses embedded VBScript to run the payload. It only generates payloads for egress listeners and is limited to x86.&#x20;

## MS Office Macro

This produces a VBA script that can be injected within a macro-enabled MS Word/Excel document.&#x20;

* This only generates payloads via egress listeners but is compatible for both x86/x64 Office.

## "Stager" Payload Generator

This is a payload "stager" that comes in a variety of languages including C, C#, PowerShell, Python, and VBA.

These are useful when building your own custom payloads/exploits.

* Only generates payloads egress listeners but supports x86/x64.

## "Stageless" Payload Generator

This generates "stageless" payloads rather than "stagers". It has a slight tweak to outputted formats (e.g. no PowerShell).&#x20;

It features options of specifying an `EXIT` function for process or threads. It can also generate payloads for P2P listeners.

## Windows "Stager" Payload

This produces a pre-compiled stager as an `.exe` or `.dll`.

## Windows "Stageless" Payload

Outputs a pre-compiled stageless payload as an `.exe`, service, `.dll`, shellcode, or PowerShell.&#x20;

_<mark style="color:yellow;">**This is the only means of generating payloads for P2P listeners.**</mark>_

## Windows "Stageless" Generate All Payloads

This produces every "stageless" payload variant for every listener in x86/x64.

## Need more information?

Check out this [blog](https://buffered.io/posts/staged-vs-stageless-handlers/).
