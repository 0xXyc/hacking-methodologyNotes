---
description: 12/1/2022
---

# Twiggy

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 447d1a569b68aef53bf6381773165d75 (RSA)
|   256 1c789d838152f4b01d8e3203cba61893 (ECDSA)
|_  256 08c912d97b9898c8b3997a19822ea3ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-favicon: Unknown favicon MD5: 11FB4799192313DD5474A343D9CC0A17
|_http-server-header: nginx/1.16.1
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Home | Mezzanine
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.16.1
|_http-title: Site doesn't have a title (application/json)
```

Enumerated UDP ports:

```
```

Notes:

* 22/SSH
* 53/DNS -- Attempted zone transfer and other enumeration
* 80/HTTP nginx 1.16.1 no exploits -- <mark style="color:yellow;">Mezzanine</mark>
* <mark style="color:yellow;">Confirmed XSS via 40799.txt</mark>
  * admin page
  * blog page
  * Search bar
  * Contact form
* 8000/HTTP -- cherrypy 5.6.0

## Enumeration

### Port 80 - HTTP (Mezzanine/HTTP)

Searchsploit:

{% code overflow="wrap" %}
```
searchsploit mezz
 Exploit Title                                                                                                                             
Mezzanine 4.2.0 - Cross-Site Scripting                                                                                                      | python/webapps/40799.txt
```
{% endcode %}

## Exploitation

{% embed url="https://www.exploit-db.com/raw/48421" %}
