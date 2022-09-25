---
description: 09-24-22
---

# Traverxec

### Improved skills

* Skill 1
* Skill 2

### Used tools

* Nmap -- Autorecon

## Information Gathering

Scanned all TCP ports:

```
```

Enumerated UDP ports:

```
```

Notes:

* 22/SSH
* 80/HTTP

### Visual Inspection of Web Server

* We have a Contact Form
* There is a note on the site (embedded picture) that has names on it
  * John
  * Mark
  * Accountant

### Source Code Analysis

<mark style="color:yellow;">Interesting finds</mark>

* I notice that there is a header with information on it that I did not see on the site during visual inspection
* It says "Hello, my name is **David White**. I create for the web."
* Upon going back to the site, I notice that it is there, it just was not adhearing to the site's colors and matched the same color as the background... as you can see below!

Initial:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

After Discovery:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* I simply highlighted the text on the site to make this appear this way!&#x20;
* Did we just gain another potential user?

### Possible Users?

* David White
* John
* Mark
* Accountant

## Enumeration

### Port 80 - HTTP (Apache)

random text here

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
