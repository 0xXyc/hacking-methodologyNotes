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

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* We can now see additional information about the group

### Group Membership

* With Backup Operators being enumerated, we can now list the members of this group

```
Get-ADGroupMember -Identity "Backup Operators"
```

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
