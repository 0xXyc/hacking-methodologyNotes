# Credentialed Enumeration - From Windows

## Introduction

* <mark style="color:yellow;">Here we will cover SharpHound/BloodHound, PowerView/SharpView, Group2, Snaffler, and some built-in tools useful for AD enumeration</mark>

## Active Directory PowerShell Module

Get-Module:

* This will <mark style="color:yellow;">list all available modules</mark>, version, and their potential commands for use

```
Get-Module
```

### Load AD Module

```
Import-Module ActiveDirectory
```

### Get Domain Info

```
Get-ADDomain
```

Get-ADUser

```
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

* Notice the <mark style="color:yellow;">ServicePrincipalName</mark> at the end of the cmdlet
* This will show a list of user accounts that are potentially susceptible to to <mark style="color:yellow;">Kerberoasting</mark>

### Checking For Trust Relationships

```
Get-ADTrust -Filter *
```

* This cmdlet will <mark style="color:yellow;">print out any trust relationships the domain has</mark>
* We can determine if they are trusts within our forest or with domains in other forests
* We can also determine the type of trust, direction of the trust, and the name of the domain the relationship is with

Why is this <mark style="color:yellow;">important</mark>?

* This will be helpful when <mark style="color:yellow;">taking advantage of child-to-parent trust relationships and attacking across forest trusts</mark>

### Group Enumeration

```
Get-ADGroup -Filter * | select name
```

* We can then take the <mark style="color:yellow;">results of this cmdlet and feed interesting names back into the cmdlet</mark> to get more detailed information about a particular group

### Detailed Group Info

```
Get-ADGroup -Identity "Backup Operators"
```

<figure><img src="../../.gitbook/assets/image (3) (1) (6).png" alt=""><figcaption></figcaption></figure>

* We can now see additional information about the group

### Group Membership

* With Backup Operators being enumerated, we can now list the members of this group

```
Get-ADGroupMember -Identity "Backup Operators"
```

<figure><img src="../../.gitbook/assets/image (9) (6).png" alt=""><figcaption></figcaption></figure>

## PowerView

<mark style="color:yellow;">PowerView is a tool that is written in PowerShell to help us gain situational awareness in an AD domain.</mark>

* <mark style="color:yellow;">It is essentially the CLI-version of BloodHound</mark>

PowerView Functions:

<figure><img src="../../.gitbook/assets/image (2) (3) (4) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (11).png" alt=""><figcaption></figcaption></figure>

### Domain User Information

Import PowerView:

```
ipmo PowerView.ps1
```

```
Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol
```

<figure><img src="../../.gitbook/assets/image (1) (2) (2).png" alt=""><figcaption></figcaption></figure>

### Recursive

```
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

### Trust Enumeration

```
Get-DomainTrustMapping
```

### Testing for Local Admin Access

```
Test-AdminAccess -ComputerName ACADEMY-EA-MS01
```

### Finding Users with SPN Set

```
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

## SharpView

{% embed url="https://github.com/BC-SECURITY/Empire/blob/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1" %}

General Usage:

```
.\SharpView.exe Get-DomainUser -Help
```

<mark style="color:yellow;">Enumerate</mark> information about a <mark style="color:yellow;">specific user</mark>:

```
.\SharpView.exe Get-DomainUser -Identity forend
```

## Snaffler

{% embed url="https://github.com/SnaffCon/Snaffler" %}

* <mark style="color:yellow;">This is similar to Responder, but the Windows version</mark>

General Usage:

```
.\Snaffler.exe -d INLANEFREIGHT.LOCAL -s -v data

 .::::::.:::.    :::.  :::.    .-:::::'.-:::::':::    .,:::::: :::::::..
;;;`    ``;;;;,  `;;;  ;;`;;   ;;;'''' ;;;'''' ;;;    ;;;;'''' ;;;;``;;;;
'[==/[[[[, [[[[[. '[[ ,[[ '[[, [[[,,== [[[,,== [[[     [[cccc   [[[,/[[['
  '''    $ $$$ 'Y$c$$c$$$cc$$$c`$$$'`` `$$$'`` $$'     $$""   $$$$$$c
 88b    dP 888    Y88 888   888,888     888   o88oo,.__888oo,__ 888b '88bo,
  'YMmMY'  MMM     YM YMM   ''` 'MM,    'MM,  ''''YUMMM''''YUMMMMMMM   'W'
                         by l0ss and Sh3r4 - github.com/SnaffCon/Snaffler

2022-03-31 12:16:54 -07:00 [Share] {Black}(\\ACADEMY-EA-MS01.INLANEFREIGHT.LOCAL\ADMIN$)
2022-03-31 12:16:54 -07:00 [Share] {Black}(\\ACADEMY-EA-MS01.INLANEFREIGHT.LOCAL\C$)
2022-03-31 12:16:54 -07:00 [Share] {Green}(\\ACADEMY-EA-MX01.INLANEFREIGHT.LOCAL\address)
2022-03-31 12:16:54 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares)
2022-03-31 12:16:54 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\User Shares)
2022-03-31 12:16:54 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\ZZZ_archive)
2022-03-31 12:17:18 -07:00 [Share] {Green}(\\ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL\CertEnroll)
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.kdb$|289B|3/31/2022 12:09:22 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\GroupBackup.kdb) .kdb
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.key$|299B|3/31/2022 12:05:33 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\ShowReset.key) .key
2022-03-31 12:17:19 -07:00 [Share] {Green}(\\ACADEMY-EA-FILE.INLANEFREIGHT.LOCAL\UpdateServicesPackages)
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.kwallet$|302B|3/31/2022 12:04:45 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\WriteUse.kwallet) .kwallet
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.key$|298B|3/31/2022 12:05:10 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\ProtectStep.key) .key
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.ppk$|275B|3/31/2022 12:04:40 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\StopTrace.ppk) .ppk
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.key$|301B|3/31/2022 12:09:17 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\WaitClear.key) .key
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.sqldump$|312B|3/31/2022 12:05:30 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\DenyRedo.sqldump) .sqldump
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.sqldump$|310B|3/31/2022 12:05:02 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\AddPublish.sqldump) .sqldump
2022-03-31 12:17:19 -07:00 [Share] {Green}(\\ACADEMY-EA-FILE.INLANEFREIGHT.LOCAL\WsusContent)
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.keychain$|295B|3/31/2022 12:08:42 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\SetStep.keychain) .keychain
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.tblk$|279B|3/31/2022 12:05:25 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\FindConnect.tblk) .tblk
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.psafe3$|301B|3/31/2022 12:09:33 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\GetUpdate.psafe3) .psafe3
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.keypair$|278B|3/31/2022 12:09:09 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\UnprotectConvertTo.keypair) .keypair
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.tblk$|280B|3/31/2022 12:05:17 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\ExportJoin.tblk) .tblk
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.mdf$|305B|3/31/2022 12:09:27 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\FormatShow.mdf) .mdf
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.mdf$|299B|3/31/2022 12:09:14 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\LockConfirm.mdf) .mdf

```

* It is possible for us to find passwords, SSH keys, configuration files, or other data that can be used to further our access
