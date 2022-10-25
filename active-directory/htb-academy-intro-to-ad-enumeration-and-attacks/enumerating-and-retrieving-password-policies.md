# Enumerating & Retrieving Password Policies

## Introduction

* You can pull the domain password policy in several ways
* This all depends on how the domain is configured and whether or not we have valid domain credentials or not
* <mark style="color:yellow;">With credentials is obviously the most effective way, but SMB NULL sessions, or LDAP anonymous bind can be a way to get this information as well</mark>
* The password policy can also be obtained remotely via <mark style="color:yellow;">CrackMapExec, Enum4Linux,</mark> or <mark style="color:yellow;">rpcclient</mark>

## Enumerating Password Policy (Credentialed)

### CME: Obtaining Password Policy

```
crackmapexec smb <ip> -u <username> -p <password> --pass-pol
```

## Rpcclient (NULL)

Using SMB NULL Authentication and Rpcclient:

```
rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## Enum4Linux

Enum4Linux combines all these tools in one package:

* <mark style="color:yellow;">nmblookup</mark>
* <mark style="color:yellow;">net</mark>
* <mark style="color:yellow;">rpcclient</mark>
* <mark style="color:yellow;">smbclient</mark> -- to use for enumeration of Windows hosts and domains

Syntax:

```
enum4linux -P 172.16.5.5
```

