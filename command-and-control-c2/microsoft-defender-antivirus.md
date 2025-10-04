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

Settings from **Malleable C2 profile**, such as the callback addresses are stomped into the DLL at the time your payloads are generated.

When you generate payloads artifacts such as the EXE's, DLL's, and PowerShell, this Beacon shellcode is generated, XOR'd and then stomped into them.

The artifacts themselves act as "simple" shellcode injectors.&#x20;

They all, with the exception of the service binary, inject Beacon shellcode into themselves.&#x20;

**That flow looks something like this:**

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

The service binary is identical except that it spawns another process and performs remote injection instead.

The complication when trying to bypass AV signatures is knowing which "part" of the payload they apply to - the core Beacon, the reflective loader, or the artifact.

## Artifact Kit

As we've all learned by now, dropping files to disk has a pretty bad rap.&#x20;

There are instances where it is pretty much unavoidable if we want to deploy certain tactics against our target(s).

For instance, we have access to the File Server, but we can't <mark style="color:yellow;">`PsExec`</mark> to it because the default binary payload is protect by Defender.

**Here's what happens if we try with Defender enabled:**

```
beacon> ls \\fs.dev.cyberbotic.io\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     09/14/2022 15:44:51   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     05/08/2021 08:20:24   PerfLogs
          dir     09/14/2022 15:55:16   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     09/14/2022 15:59:23   ProgramData
          dir     09/14/2022 15:25:23   Recovery
          dir     09/14/2022 15:25:04   System Volume Information
          dir     09/14/2022 15:26:47   Users
          dir     09/14/2022 15:25:15   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 12kb     fil     09/14/2022 16:00:25   DumpStack.log.tmp
 1gb      fil     09/14/2022 16:00:25   pagefile.sys

beacon> jump psexec64 fs.dev.cyberbotic.io smb
[-] Could not start service 633af16 on fs.dev.cyberbotic.io: 225
```

**We can view that message (`225`) by using:**

```
PS C:\Users\Attacker> net helpmsg 225

Operation did not complete successfully because the file contains a virus or potentially unwanted software.
```

and verify that Windows Defender indeed blocked the attempt.

**If we copy the payload to our local desktop and check the associated log, we can see that the "file" was detected:**

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

These Cobalt Strike "artifacts" are nothing more than shellcode runners that inject Beacon shellcode when executed.&#x20;

As a _**rule of thumb**_, they inject the shellcode into themselves (e.g. using the `VirtualAlloc()` & `CreateThread()` pattern).

The service binary is the one exception, as it spawns a new process and injects the shellcode into that instead.

This is done so that when moving laterally with <mark style="color:yellow;">`PsExec`</mark>, the artifact can be deleted from disk immediately.

The Artifact Kit contains the source code for these artifacts and is designed to facilitate the development the development of sandbox-safe injectors. The idea is to develop artifacts that inject Beacon shellcode in a way that cannot be emulated by AV engines.

There are several bypass techniques provided with the kit which you can modify, or you can implement entirely new ones yourself.

Where the Artifact Kit does not help is making Beacon resilient to detection once it's running in memory (e.g. from memory scanners).
