---
description: 12/10/2023
---

# Networked (Easy)

<figure><img src="../../../.gitbook/assets/Networked.png" alt="" width="563"><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
Nmap scan report for networked.htb (10.129.61.220)
Host is up (0.062s latency).
Not shown: 65377 filtered tcp ports (no-response), 155 filtered tcp ports (host-prohibited)
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 2275d7a74f81a7af5266e52744b1015b (RSA)
|   256 2d6328fca299c7d435b9459a4b38f9c8 (ECDSA)
|_  256 73cda05b84107da71c7c611df554cfc4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
443/tcp closed https
```

Enumerated UDP ports:

```
```

Notes:

Added `networked.htb` to `/etc/hosts`.

Attack entry point is likely going to be stemming from port 80, Apache webserver.

## Enumeration

### Port 80 - HTTP (Apache 2.4.6)

### Dirsearch

```
[16:53:43] 301 -  236B  - /backup  ->  http://networked.htb/backup/
[16:53:43] 200 -  885B  - /backup/
[16:53:44] 403 -  210B  - /cgi-bin/
[16:53:58] 200 -    1KB - /photos.php
[16:54:08] 200 -  169B  - /upload.php
[16:54:08] 301 -  237B  - /uploads  ->  http://networked.htb/uploads/
[16:54:08] 200 -    2B  - /uploads/
```

`/backup` contains a file: `backup.tar`:

* `index.php`
* `lib.php`
* `photos.php`
* `upload.php`

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
