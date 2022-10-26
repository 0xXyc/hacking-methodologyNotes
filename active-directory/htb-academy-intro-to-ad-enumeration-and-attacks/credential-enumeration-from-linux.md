# Credential Enumeration - From Linux

## CrackMapExec

### Domain <mark style="color:yellow;">User</mark> Enumeration

* Point CME at the DC and use the credentials for a particular user to retrive a list of domain users
* BadPwdCount will be included

```
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users
```

### Domain <mark style="color:yellow;">Group</mark> Enumeration

* You can also obtain a listing of domain groups

```
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups
```

Pay attention to groups like:

* <mark style="color:yellow;">Backup Operators</mark>
* <mark style="color:yellow;">Administrators</mark>
* <mark style="color:yellow;">Domain Admins</mark>
* <mark style="color:yellow;">Executives</mark>

### CME - Logged On Users

*
