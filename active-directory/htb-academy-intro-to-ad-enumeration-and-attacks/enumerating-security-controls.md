# Enumerating Security Controls

## Windows Defender

PowerShell Cmdlet: <mark style="color:yellow;">Get-MpComputerStatus</mark>

<figure><img src="../../.gitbook/assets/image (5) (1) (3).png" alt=""><figcaption></figcaption></figure>

* According to the output, we see that the <mark style="color:yellow;">RealTimeProtectionEnabled</mark> parameter is set to <mark style="color:yellow;">True</mark>
  * This means that Defender is on the system and active

## AppLocker

* This is <mark style="color:yellow;">Microsoft's application whitelisting utility</mark> that gives administrators control over which applications and files users can run
* It provides granular control over executables, scripts, Windows installer files, DLLs, packaged apps, and packaged app installers
* It is very common for organizations to block cmd.exe, powershell.exe, and write access to certain directories
* However, this can all be bypassed

For example:

* Organizations will remember to block the PowerShell.exe executable but forget there are other locations where PowerShell can be executed from
  * Such as: <mark style="color:yellow;">%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe</mark>

### PowerShell Cmdlet for AppLocker Policy

```
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

PathConditions      : {%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 3d57af4a-6cf8-4e5b-acfc-c2c2956061fa
Name                : Block PowerShell
Description         : Blocks Domain Users from using PowerShell on workstations
UserOrGroupSid      : S-1-5-21-2974783224-3764228556-2640795941-513
Action              : Deny

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 921cc481-6e17-4653-8f75-050b80acca20
Name                : (Default Rule) All files located in the Program Files folder
Description         : Allows members of the Everyone group to run applications that are located in the Program Files folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : a61c8b2c-a319-4cd0-9690-d2177cad7b51
Name                : (Default Rule) All files located in the Windows folder
Description         : Allows members of the Everyone group to run applications that are located in the Windows folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : fd686d83-a829-4351-8ff4-27c7de5755d2
Name                : (Default Rule) All files
Description         : Allows members of the local Administrators group to run all applications.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow
```

## Local Administrator Password Solution (LAPS)

<mark style="color:yellow;">This technology is used to randomize and rotate local administrator passwords on Windows hosts and prevent lateral movement</mark>.&#x20;

We can <mark style="color:yellow;">enumerate what domain users can read the LAPS password set for machines with LAPS installed and what machines do NOT have LAPS installed</mark>.

We can use the [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) to parse EntendedRIghts for all computers with LAPS enabled.

This will show groups delegated to read LAPS passwords which are often users in protected groups.

<mark style="color:yellow;">An account that has joined a computer to a domain receives all Extended Rights over that host and this right gives the account the ability to READ PASSWORDS</mark>!

### Using Find-LAPSDelegatedGroups

```
PS C:\ Find-LAPSDelegatedGroups
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Find-AdmPwdExtendedRights

* <mark style="color:yellow;">This checks the rights on each computer with LAPS installed</mark>
* Users with "All Extended Rights" can read LAPS passwords and may be less protected than users in delegated groups
* So, definitely check this out

```
PC C:\ Find-AdmPwdExtendedRights
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

### Get-LAPSComputers

* After <mark style="color:yellow;">Find-AdmPwdExtendedRights</mark>, we can use <mark style="color:yellow;">Get-LAPSComputers</mark> to <mark style="color:yellow;">search for computers that have LAPS enabled</mark> when passwords expire, and even <mark style="color:yellow;">obtain the randomized passwords in cleartext if our user has access</mark>

```
PS C:\ Get-LAPSComputers
```

<figure><img src="../../.gitbook/assets/image (4) (9).png" alt=""><figcaption></figcaption></figure>
