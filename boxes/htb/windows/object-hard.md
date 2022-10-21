---
description: 10-21-2022
---

# Object (Hard)

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
8080/tcp open  http    Jetty 9.4.43.v20210629
|_http-server-header: Jetty(9.4.43.v20210629)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Enumerated UDP ports:

```
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
```

Notes:

* HTTP running on 80/8080
  * <mark style="color:yellow;">Jetty 9.4.43</mark>&#x20;
* <mark style="color:yellow;">5985/WinRM</mark> -- Can we find valid creds to Evil-WinRM into the machine?
* UDP -- 53/DNS & 123/NTP
  * Attempt zone transfer

## Enumeration

### Port 80 - HTTP (IIS 10.0)

Visual Inspection:

* The site appears to be looking for automation technology solutions
* I notice a potential DNS name here

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* Add to /etc/hosts
  * I attempted a zone transfer in an attempt to broaden our overall attack surface with no luck

Nikto:&#x20;



## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
