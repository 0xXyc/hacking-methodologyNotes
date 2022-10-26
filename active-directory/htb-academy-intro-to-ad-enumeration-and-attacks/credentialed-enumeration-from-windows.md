# Credentialed Enumeration - From Windows

## Introduction

* Here we will cover SharpHound/BloodHound, PowerView/SharpView, Group2, Snaffler, and some built-in tools useful for AD enumeration

## Active Directory PowerShell Module

Get-Module:

* This will list all available modules, version, and their potential commands for use

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

* Notice the <mark style="color:yellow;">ServicePrincipalName</mark> at the end of the command
* This will show a list of user accounts that are potentially susceptible to to <mark style="color:yellow;">Kerberoasting</mark>
