---
description: 10/01/2025
---

# Group Policy

## Introduction: Abusing Group Policy

Group Policy is the central repository in a forest or domain that controls the configuration of computers and users.

Group Policy Objects (GPOs) are sets of configurations that are applied to Organizational Units (OUs).

By default, only Domain Admins can create GPOs and link them to OUs, but it is common practice to delegate those rights to other teams.

This delegation is typically assigned to domain groups. For example, a "Workstation Admins" group may have rights to manage GPOs that apply to a "Workstation" OU.

{% hint style="success" %}
These can create privilege escalation opportunities by allowing users to apply malicious GPOs to domain users or their computers.&#x20;
{% endhint %}

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Modify Existing GPO

#### Goal

Find GPOs modifiable by non-standard principals, see what OUs/computers they affect, then modify the GPO (SYSVOL or tool) to deploy persistence.

### Find GPOs with Modify ACLs

**Find GPOs and filter ACLs for&#x20;**_**Modify Rights**_**&#x20;(`CreateChild`, `WriteProperty`, and `GenericWrite`) and non-standard SIDs:**

```
beacon> powershell Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ActiveDirectoryRights -match "CreateChild|WriteProperty" -and $_.SecurityIdentifier -match "S-1-5-21-569305411-121244042-2357301523-[\d]{4,10}" }

AceType               : AccessAllowed
ObjectDN              : CN={5059FAC1-5E94-4361-95D3-3BB235A23928},CN=Policies,CN=System,DC=dev,DC=cyberbotic,DC=io
ActiveDirectoryRights : CreateChild, DeleteChild, ReadProperty, WriteProperty, GenericExecute
OpaqueLength          : 0
ObjectSID             : 
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-569305411-121244042-2357301523-1107
AccessMask            : 131127
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

This returns ACE(s) showing GPO GUID and principal SID.

