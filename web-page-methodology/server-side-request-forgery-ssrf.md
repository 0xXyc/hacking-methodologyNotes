---
cover: https://i.ytimg.com/vi/ih5R_c16bKc/maxresdefault.jpg
coverY: 0
---

# Server-Side Request Forgery (SSRF)

## Introduction

* <mark style="color:yellow;">An attacker can expose internal network configurations and have the ability to reveal secrets and other information from internal network enumeration stemming from a vulnerable web app</mark>
* In some cases, you may be able to <mark style="color:yellow;">access other internal systems</mark>

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Fundamentals

* Basically, a <mark style="color:yellow;">SSRF</mark> is when an attacker will forge a <mark style="color:yellow;">request that will act as a proxy</mark>
* This proxy will point to EITHER a <mark style="color:yellow;">local or remote source</mark>
* The RESPONSE will contain the data resulting from the request

## What can you do with SSRF?

* Local or remote port scanning
* Local file read using file://
* Interact with internal apps/service/network
* RCE by chaining services on the internal network
* Reflected XSS or CSRF

## Local/Remote Port Scanning

This technique will allow you to be able to map out what applications/services are running internally.

This is especially important for interacting nand querying internal applications/services.

Port scanning can be done with:

* HTTP
* HTTPs
* GOPHER
* DICT Protocols

## What does SSRF look like?

<figure><img src="../.gitbook/assets/image (5) (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

However, if there is a <mark style="color:yellow;">blind SSRF</mark> present, you can <mark style="color:yellow;">pay attention to the content-length, response time, or HTTP status code</mark>

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

* Be sure to look for other unusal elements in the request as this can lead you to possible SSRF

## Local File Read

* SSRF can only access or read files using the file:/// protocol

<mark style="color:red;">Local File Read Examples:</mark>&#x20;

<mark style="color:yellow;">`GET /?url=file:///etc/passwd HTTP/1.1`</mark>

* Notice the 3 /'s&#x20;

<mark style="color:yellow;">`GET /?url=file:/PathToFile`</mark>

<mark style="color:yellow;">`GET /?url=file://hostname/PathToFile`</mark>

<figure><img src="../.gitbook/assets/image (1) (7).png" alt=""><figcaption></figcaption></figure>

If the web application is deployed on a <mark style="color:yellow;">Windows server</mark>, you can access files using the file:/// protocol as well! You just need to use file:///\<drive_letter/path-to-file_&#x20;

Example: <mark style="color:yellow;">file:///d:/hello.txt</mark>

## Attack Vectors

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery#tools" %}
