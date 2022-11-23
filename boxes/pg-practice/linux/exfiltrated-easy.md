---
description: 11/22/2021
---

# Exfiltrated (Easy)

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1994b952225ed0f8520d363b448bbcf (RSA)
|   256 0f448badad95b8226af036ac19d00ef3 (ECDSA)
|_  256 32e12a6ccc7ce63e23f4808d33ce9b3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
|_http-favicon: Unknown favicon MD5: 09BDDB30D6AE11E854BFF82ED638542B
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
```

Notes:

* 80/HTTP - <mark style="color:yellow;">added exfiltrated.offsec to /etc/hosts</mark>
  * CMS- Subrion -- Can we enumerate the version? -- Yes, <mark style="color:yellow;">v4.2.1</mark>
  * Apache 2.4.41
* Perform quick win attempt with searchsploit on both services above

## Enumeration

### Port 80 - HTTP (Apache) -- exfiltrated.offsec

Visual Inspection:

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Wappalyzer:

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Searchsploit results of Subrion CMS:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Enumerated Subrion Version:

* Found via selecting the "Go to admin dashboard" button on the main site

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

* Version has been enumerated <mark style="color:yellow;">Subrion v4.2.1</mark>
* We have 4 exploits that are targeted towards this version

### Subrion Default Credentials

A simple Google search for Subrion Default Creds gives us this first site with lucrative and guessable information

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">Attempted admin/admin login and it worked!</mark>

### Subrion Admin Panel

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>



## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
