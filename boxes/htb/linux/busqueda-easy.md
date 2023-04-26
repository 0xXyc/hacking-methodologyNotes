---
description: 04-26-2023
---

# Busqueda (Easy)

<figure><img src="../../../.gitbook/assets/Busqueda.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4fe3a667a227f9118dc30ed773a02c28 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIzAFurw3qLK4OEzrjFarOhWslRrQ3K/MDVL2opfXQLI+zYXSwqofxsf8v2MEZuIGj6540YrzldnPf8CTFSW2rk=
|   256 816e78766b8aea7d1babd436b7f8ecc4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPTtbUicaITwpKjAQWp8Dkq1glFodwroxhLwJo6hRBUK
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://searcher.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
```

Notes:

* 80 is open
* 22 is open
* Navigated to http://busqueda.htb and the page failed to load and I noticed <mark style="color:yellow;">searcher.htb</mark> in the URL
* Let's add this to <mark style="color:yellow;">/etc/hosts</mark>

## Enumeration

### Port 80 - HTTP (Apache httpd 2.4.52)

#### Visual Inspection

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption><p>Flask and Searchor 2.4.0</p></figcaption></figure>

* At the bottom of the page, my eyes are immediately drawn to the Searchor 2.4.0 repo is search of more information
* Let's check out the **Releases** page

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

As it turns out, we get some pretty interesting information from the **Releases** page.

<mark style="color:yellow;">Since we know the target is running version 2.4.0, this means that it is vulnerable to this vulnerability that was later discovered in a later version</mark>.

#### Vulnerability in v2.4.2

{% embed url="https://github.com/ArjunSharda/Searchor/pull/130" %}



## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
