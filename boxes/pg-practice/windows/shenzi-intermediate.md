# Shenzi (Intermediate)

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
80/tcp   open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.211.55/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 56F7C04657931F2D0B79371B2D6E9820
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:   a0a44cc99e84b26f9e639f9ed229dee0
|_SHA-1: b0238c547a905bfa119c4e8baccaeacf36491ff6
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
| http-title: Welcome to XAMPP
|_Requested resource was https://192.168.211.55/dashboard/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
445/tcp  open  microsoft-ds?
3306/tcp open  mysql?
| fingerprint-strings: 
|   NULL: 
|_    Host '192.168.49.211' is not allowed to connect to this MariaDB server
5040/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.93%I=7%D=12/15%Time=639B9B71%P=x86_64-pc-linux-gnu%r(N
SF:ULL,4D,"I\0\0\x01\xffj\x04Host\x20'192\.168\.49\.211'\x20is\x20not\x20a
SF:llowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-12-15T22:13:39
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
```

Enumerated UDP ports:

```
Filtered
```

Notes:

<mark style="color:yellow;">21/FTP - No anonymous access</mark>

<mark style="color:yellow;">80/Apache 2.4.43 - XAMPP 7.4.6</mark>

<mark style="color:yellow;">445/SMB - Enum4Linux no null authentication allowed</mark>

## Enumeration

### Port 80 - HTTP (Apache 2.4.43)

Dirsearch (Directory Bruteforce):

<figure><img src="../../../.gitbook/assets/image (5) (12).png" alt=""><figcaption><p>Dirsearch Output</p></figcaption></figure>

* /Webalizer
  * <mark style="color:yellow;">OpenSSL/1.1.1g PHP/7.4.6</mark>
* <mark style="color:red;">postmaster</mark> -- possible user
* <mark style="color:red;">shenzi</mark> -- possible user
* <mark style="color:red;">john</mark> -- possible user
* /phpmyadmin
  * New XAMPP Security concept: "Access to the requested directory is only available from the local network. This setting can be configured in the file 'httpd-xampp.conf'".
* /phpinfo.php
  * I could not find anything sensitive in here
* /dashboard/docs/backup-restore-mysql.html
  * I see a potential username/password for the mysql database

<figure><img src="../../../.gitbook/assets/image (5) (3) (1).png" alt=""><figcaption><p>Interesting find</p></figcaption></figure>

* Well, this box really did not make me very happy once I found out all my hard work instantly went to waste after 3 hours of nothing
* <mark style="color:yellow;">I had to take a hint to find out that /shenzi is a directory that should have been guessed?</mark>
  * <mark style="color:yellow;">What if we didn't have the box's name? Lol</mark>

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
