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

* We can use CME to tell how many users are currently logged in
* Can we steal a local admin's credentials or impersonate them?

```
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
```

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

* We can also use <mark style="color:yellow;">BloodHound</mark> and <mark style="color:yellow;">PowerView</mark> to achieve a similar goal of hunting for user sessions

### CME Share Searching

* You can use the <mark style="color:yellow;">--shares</mark> flag to enumerate available shares on the remote host
* You can also see if we have read/write access on that share

```
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares
```

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### Spider\_plus (CME)

* We can spider into files recursively with the spider\_plus option in CME
* This will write the results in /tmp/cme_spider\_plus/\<ip of host>_&#x20;
* We should be looking for passwords and other sensitive information in here

```
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```

View the first 10 lines of the json file with the head command:

```
head -n 10 /tmp/cme_spider_plus/172.16.5.5.json
```

## SMBMap

* This is a tool used for enumerating SMB shares from a Linux host
  * shares
  * permissions
  * share contents if possible
  * Shell can be obtained and you can download, upload files, and execute remote commands

### SMBMap to Check or Map Access

* I like to use the term "<mark style="color:yellow;">map</mark>" hence SMBMap
* You can see that we are simply mapping out what we can see before we go digging into it

```
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5
```

<figure><img src="../../.gitbook/assets/image (7) (6).png" alt=""><figcaption></figcaption></figure>

### Recursive List of all Directories

* We can also recursively list directories with easy
* This is done with the <mark style="color:yellow;">-R argument</mark>

<mark style="color:yellow;">Recursively List Directories</mark>:

```
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```

* However, if we do NOT want to see directories, and we want to see everything, we simply remove <mark style="color:yellow;">--dir-only</mark>

Recursively Listing Every File in a Share:

```
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares'
```

## rpcclient

* This is a handy tool to provide functionality with MS-RPC
* It is <mark style="color:yellow;">capable of enumerating, adding, changing, and even removing objects from AD</mark>

SMB Null Session:

```
rpcclient -U "" -N 172.16.5.5

rpcclient $>
```

<figure><img src="../../.gitbook/assets/image (5) (6).png" alt=""><figcaption></figcaption></figure>

### rpcclient Enumeration

* There are different types of identifiers that you will notice here
* <mark style="color:yellow;">Relative Identifier (RID)</mark>, is a unique identifier utilized by Windows to track and identify objects

Example:

<mark style="color:yellow;">SID</mark> for INLANEFREIGHT.LOCAL is: `S-1-5-21-3842939050-3880317879-2865463114`

<mark style="color:yellow;">RID</mark> for domain user htb-student is: \[0x457]

RID for Administrator is: Hex 0x1f4 or decimal 500

### Queryuser

Querying the Administrator user in rpcclient:

```
rpcclient $> queryuser 0x1f4
```

<figure><img src="../../.gitbook/assets/image (5) (1) (4).png" alt=""><figcaption></figcaption></figure>

### Enumdomusers

```
rpcclient $> enumdomusers
```

* This will enumerate all users on the domain

<figure><img src="../../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>
