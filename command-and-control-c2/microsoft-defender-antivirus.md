---
description: 10/04/2025
---

# Microsoft Defender Antivirus

## Introduction

Microsoft Defender is an antivirus product that comes pre-installed with Windows (both desktop and server editions).

In previous years, Defender was seen as a bit of a joke, but has since evolved into a rather formidable defence.

**There are three facets to its detection capability that we shall explore in this chapter:**

1. **On-disk**
2. **In-memory**
3. **Behavioral**

### How Payloads are Generated in Cobalt Strike

Before diving into Defender, it's useful to explain how payloads are generated in Cobalt Strike since there are only a few individual parts.

Beacon itself is written as a _**Reflective DLL**_ based on Steven Fewer's [work](https://github.com/stephenfewer/ReflectiveDLLInjection).

**So, there's the:**

1. Core Beacon DLL (everything that make Beacon actually function)&#x20;
2. Plus a reflective loader component.

These are then converted into Position Independent Shellcode, which when injected and executed, calls the reflective loader entry point.

The reflective loader then loads the Beacon DLL into memory and kicks off a new thread to run it.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Settings from Malleable C2 profile, such as the callback addresses are stomped into the DLL at the time your payloads are generated.

When you generate payloads artifacts such as the EXE's, DLL's, and PowerShell, this Beacon shellcode is generated, XOR'd and then stomped into them.

The artifacts themselves act as "simple" shellcode injectors.&#x20;

They all, with the exception of the service binary, inject Beacon shellcode into themselves.&#x20;

**That flow looks something like this:**

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

The service binary is identical except that it spawns another process and performs remote injection instead.

The complication when trying to bypass AV signatures is knowing which "part" of the payload they apply to - the core Beacon, the reflective loader, or the artifact.

## Artifact Kit

