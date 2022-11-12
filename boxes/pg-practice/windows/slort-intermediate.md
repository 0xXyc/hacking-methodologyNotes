---
description: 11/11/2022
---

# Slort (Intermediate)

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql?
| fingerprint-strings: 
|   Kerberos, LDAPSearchReq, LPDString, NULL: 
|_    Host '192.168.49.81' is not allowed to connect to this MariaDB server
4443/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.81.53:4443/dashboard/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8080/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.81.53:8080/dashboard/
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.93%I=7%D=11/11%Time=636E9022%P=x86_64-pc-linux-gnu%r(N
SF:ULL,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20al
SF:lowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(Kerberos,
SF:4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(LPDString,4C,
SF:"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20allowed\x
SF:20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(LDAPSearchReq,4C
SF:,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.81'\x20is\x20not\x20allowed\
SF:x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-11-11T18:13:24
|_  start_date: N/A
```

Enumerated UDP ports:

```
PORT     STATE         SERVICE
123/udp  open|filtered ntp
137/udp  open|filtered netbios-ns
138/udp  open|filtered netbios-dgm
500/udp  open|filtered isakmp
1900/udp open|filtered upnp
4500/udp open|filtered nat-t-ike
5050/udp open|filtered mmcc
5353/udp open|filtered zeroconf
5355/udp open|filtered llmnr
```

Notes:



## Enumeration

### Port 4443 - HTTP (XAMPP)

* VHOST enumeration
* Directory enumeration
* Source code analysis (static)
* Visual Inspection of site sources -- passwords, images, etc.

VHOST Enumeration:

```
gobuster vhost -u http://192.168.81.53:4443 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 100 | grep Status: 200
```

Directory Enumeration:

```
gobuster dir -u http://192.168.81.53:4443 -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.81.53:4443
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2022/11/12 11:14:43 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 1045]
/.htpasswd            (Status: 403) [Size: 1045]
/.hta                 (Status: 403) [Size: 1045]
/aux                  (Status: 403) [Size: 1045]
/cgi-bin/             (Status: 403) [Size: 1059]
/com1                 (Status: 403) [Size: 1045]
/com2                 (Status: 403) [Size: 1045]
/com4                 (Status: 403) [Size: 1045]
/com3                 (Status: 403) [Size: 1045]
/con                  (Status: 403) [Size: 1045]
/dashboard            (Status: 301) [Size: 349] [--> http://192.168.81.53:4443/dashboard/]
/favicon.ico          (Status: 200) [Size: 30894]
/examples             (Status: 503) [Size: 1059]
/img                  (Status: 301) [Size: 343] [--> http://192.168.81.53:4443/img/]
/index.php            (Status: 302) [Size: 0] [--> http://192.168.81.53:4443/dashboard/]
/licenses             (Status: 403) [Size: 1204]
/lpt1                 (Status: 403) [Size: 1045]
/lpt2                 (Status: 403) [Size: 1045]
/nul                  (Status: 403) [Size: 1045]
/phpmyadmin           (Status: 403) [Size: 1204]
/prn                  (Status: 403) [Size: 1045]
/server-status        (Status: 403) [Size: 1204]
/server-info          (Status: 403) [Size: 1204]
/site                 (Status: 301) [Size: 344] [--> http://192.168.81.53:4443/site/]
/webalizer            (Status: 403) [Size: 1045]
Progress: 4627 / 4714 (98.15%)===============================================================
2022/11/12 11:15:05 Finished
```

* /site looks interesting
* Found file inclusion within source code on /site

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### Port 8080 - HTTP (XAMPP)

mailto:postmaster@localhost

Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6

## Exploitation

### Local File Inclusion (LFI)

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
