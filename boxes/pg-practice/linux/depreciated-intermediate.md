---
description: 12/05/2022
---

# Depreciated (Intermediate)

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1994b952225ed0f8520d363b448bbcf (RSA)
|   256 0f448badad95b8226af036ac19d00ef3 (ECDSA)
|_  256 32e12a6ccc7ce63e23f4808d33ce9b3a (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET OPTIONS HEAD
|_http-title: Under Maintainence
|_http-server-header: nginx/1.18.0 (Ubuntu)
5132/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, NULL: 
|     Enter Username:
|   GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     Enter Username: Enter OTP: Incorrect username or password
8433/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.8.10)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD
|_http-server-header: Werkzeug/2.0.2 Python/3.8.10
```

Enumerated UDP ports:

```
```

Notes:



## Enumeration

### Port 80 - HTTP (Nginx 1.18.0)

* Supposedly the main site is under maintenance and is operational on 5132

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption><p>Under maintenance</p></figcaption></figure>

### Port 5132 - (Nginx 1.18.0)

* Enter Username: Enter OTP: Incorrect username or password

<figure><img src="../../../.gitbook/assets/image (3) (2).png" alt=""><figcaption><p>Enter Username</p></figcaption></figure>

### Port 8433 - (Werkzeug httpd 2.0.2 - Python 3.8.10)

* API working, check documentation for usage

<figure><img src="../../../.gitbook/assets/image (3) (3).png" alt=""><figcaption><p>API working</p></figcaption></figure>

#### Directory Enumeration:

```
dirsearch -u 192.168.66.170:8433
```

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption><p>Dirsearch</p></figcaption></figure>

* We discover a graphql application running

### Graphql

<figure><img src="../../../.gitbook/assets/image (11) (4).png" alt=""><figcaption><p>graphql</p></figcaption></figure>

### Login

* Upon visiting, we get a Method not allowed error
* <mark style="color:yellow;">I then simply curl it with a POST request and we see that the authentication has failed due to a wrong username or password</mark>

<figure><img src="../../../.gitbook/assets/image (1) (11).png" alt=""><figcaption><p>POST command via Curl</p></figcaption></figure>

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
