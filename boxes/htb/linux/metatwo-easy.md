---
description: 04-26-2023
---

# MetaTwo (Easy)

<figure><img src="../../../.gitbook/assets/MetaTwo.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp?    syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c4b44617d2102d8fec1dc927fecd79ee (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDPp9LmBKMOuXu2ZOpw8JorL5ah0sU0kIBXvJB8LX26rpbOhw+1MPdhx6ptZzXwQ8wkQc88xu5h+oB8NGkeHLYhvRqtZmvkTpOsyJiMm+0Udbg+IJCENPiKGSC5J+0tt4QPj92xtTe/f7WV4hbBLDQust46D1xVJVOCNfaloIC40BtWoMWIoEFWnk7U3kwXcM5336LuUnhm69XApDB4y/dt5CgXFoWlDQi45WLLQGbanCNAlT9XwyPnpIyqQdF7mRJ5yRXUOXGeGmoO9+JALVQIEJ/7Ljxts6QuV633wFefpxnmvTu7XX9W8vxUcmInIEIQCmunR5YH4ZgWRclT+6rzwRQw1DH1z/ZYui5Bjn82neoJunhweTJXQcotBp8glpvq3X/rQgZASSyYrOJghBlNVZDqPzp4vBC78gn6TyZyuJXhDxw+lHxF82IMT2fatp240InLVvoWrTWlXlEyPiHraKC0okOVtul6T0VRxsuT+QsyU7pdNFkn2wDVvC25AW8=
|   256 2aea2fcb23e8c529409cab866dcd4411 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBB1ZmNogWBUF8MwkNsezebQ+0/yPq7RX3/j9s4Qh8jbGlmvAcN0Z/aIBrzbEuTRf3/cHehtaNf9qrF2ehQAeM94=
|   256 fd78c0b0e22016fa050debd83f12a4ab (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOP4kxBr9kumAjfplon8fXJpuqhdMJy2rpd3FM7+mGw2
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0
|_http-title: Did not follow redirect to http://metapress.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
Awaiting results
```

Notes:

* Port 21 is open
* Port 22 is open
* Port 80 is open
  * Upon visiting metapress.htb, I could not connect to the site because the URL says metapress.htb
  * Let's add this to the /etc/hosts file
  * Viewed root page source and found nothing

## Enumeration

### Port 80 - HTTP (Nginx 1.18.0) CMS: Wordpress 5.6.2

#### Visual Inspection

<figure><img src="../../../.gitbook/assets/image (9) (3) (5) (1).png" alt=""><figcaption><p>Root page</p></figcaption></figure>

Upon clicking on search, we see a PHP parameter pop up in the URL. Is it possible to inject here?



<figure><img src="../../../.gitbook/assets/image (8) (3) (2).png" alt=""><figcaption><p>Can we inject here?</p></figcaption></figure>

#### Directory Bruteforce

Dirsearch:

```
Target: http://metapress.htb/

[19:30:15] Starting: 
[19:30:22] 200 -  633B  - /.htaccess
[19:30:42] 301 -    0B  - /0  ->  http://metapress.htb/0/
[19:30:49] 301 -    0B  - /About  ->  http://metapress.htb/about-us/
[19:30:57] 301 -    0B  - /New%20folder%20(2)  ->  http://metapress.htb/New%20folder%20(2
[19:31:12] 301 -    0B  - /A  ->  http://metapress.htb/about-us/
[19:31:12] 301 -    0B  - /a  ->  http://metapress.htb/about-us/
[19:31:13] 301 -    0B  - /ab/  ->  http://metapress.htb/about-us/
[19:31:13] 301 -    0B  - /about  ->  http://metapress.htb/about-us/
[19:31:13] 301 -    0B  - /about-us  ->  http://metapress.htb/about-us/
[19:31:16] 301 -    0B  - /actuator/events  ->  http://metapress.htb/events/
[19:31:19] 302 -    0B  - /admin  ->  http://metapress.htb/wp-admin/
[19:31:20] 301 -    0B  - /admin.  ->  http://metapress.htb/admin
[19:31:21] 302 -    0B  - /admin/  ->  http://metapress.htb/wp-admin/
[19:31:51] 301 -    0B  - /asset..  ->  http://metapress.htb/asset
[19:31:52] 301 -    0B  - /atom  ->  http://metapress.htb/feed/atom/
[19:32:01] 301 -    0B  - /c  ->  http://metapress.htb/cancel-appointment/
[19:32:16] 302 -    0B  - /dashboard  ->  http://metapress.htb/wp-admin/
[19:32:25] 301 -    0B  - /e  ->  http://metapress.htb/events/
[19:32:27] 301 -    0B  - /engine/classes/swfupload//swfupload_f9.swf  ->  http://metapress.htb/engine/classes/swfupload/swfupload_f9.swf
[19:32:27] 301 -    0B  - /engine/classes/swfupload//swfupload.swf  ->  http://metapress.htb/engine/classes/swfupload/swfupload.swf
[19:32:29] 301 -    0B  - /events  ->  http://metapress.htb/events/
[19:32:31] 301 -    0B  - /extjs/resources//charts.swf  ->  http://metapress.htb/extjs/resources/charts.swf
[19:32:32] 301 -    0B  - /feed  ->  http://metapress.htb/feed/
[19:32:38] 301 -    0B  - /h  ->  http://metapress.htb/hello-world/
[19:32:39] 301 -    0B  - /hello  ->  http://metapress.htb/hello-world/
[19:32:40] 301 -    0B  - /html/js/misc/swfupload//swfupload.swf  ->  http://metapress.htb/html/js/misc/swfupload/swfupload.swf
[19:32:44] 301 -    0B  - /index.php  ->  http://metapress.htb/
[19:32:44] 301 -    0B  - /index.php/login/  ->  http://metapress.htb/login/
[19:32:52] 200 -   19KB - /license.txt
[19:32:55] 302 -    0B  - /login  ->  http://metapress.htb/wp-login.php
[19:32:55] 301 -    0B  - /login.wdm%20  ->  http://metapress.htb/login.wdm
[19:32:55] 301 -    0B  - /login.wdm%2e  ->  http://metapress.htb/login.wdm
[19:32:56] 302 -    0B  - /login/  ->  http://metapress.htb/wp-login.php
[19:33:19] 301 -    0B  - /phpmyadmin!!  ->  http://metapress.htb/phpmyadmin
[19:33:31] 301 -    0B  - /public..  ->  http://metapress.htb/public
[19:33:32] 200 -    7KB - /readme.html
[19:33:33] 301 -    0B  - /rating_over.  ->  http://metapress.htb/rating_over
[19:33:36] 200 -  113B  - /robots.txt
[19:33:37] 301 -    0B  - /rss  ->  http://metapress.htb/feed/
[19:33:37] 301 -    0B  - /s  ->  http://metapress.htb/sample-page/
[19:33:37] 301 -    0B  - /sample  ->  http://metapress.htb/sample-page/
[19:33:41] 301 -    0B  - /servlet/hello  ->  http://metapress.htb/hello-world/
[19:33:46] 302 -    0B  - /sitemap.xml  ->  http://metapress.htb/wp-sitemap.xml
[19:33:50] 301 -    0B  - /static..  ->  http://metapress.htb/static
[19:33:55] 301 -    0B  - /t  ->  http://metapress.htb/thank-you/
[19:34:12] 301 -  169B  - /wp-admin  ->  http://metapress.htb/wp-admin/
[19:34:12] 301 -  169B  - /wp-content  ->  http://metapress.htb/wp-content/
[19:34:13] 302 -    0B  - /wp-admin/  ->  http://metapress.htb/wp-login.php?redirect_to=http%3A%2F%2Fmetapress.htb%2Fwp-admin%2F&reauth=1
[19:34:13] 400 -    1B  - /wp-admin/admin-ajax.php
[19:34:13] 409 -    3KB - /wp-admin/setup-config.php
[19:34:13] 200 -    1KB - /wp-admin/install.php
[19:34:13] 200 -    0B  - /wp-config.php
[19:34:13] 200 -    0B  - /wp-content/
[19:34:13] 403 -  555B  - /wp-content/uploads/
[19:34:13] 403 -  555B  - /wp-content/upgrade/
[19:34:13] 301 -  169B  - /wp-includes  ->  http://metapress.htb/wp-includes/
[19:34:13] 403 -  555B  - /wp-includes/
[19:34:14] 200 -    0B  - /wp-cron.php
[19:34:14] 200 -    0B  - /wp-includes/rss-functions.php
[19:34:14] 200 -  578B  - /wp-json/wp/v2/users/
[19:34:14] 200 -    7KB - /wp-login.php
[19:34:14] 302 -    0B  - /wp-signup.php  ->  http://metapress.htb/wp-login.php?action=register
[19:34:14] 200 -   91KB - /wp-json/
[19:34:16] 405 -   42B  - /xmlrpc.php
```

Attempted to bypass authentication found on the admin login page using SQLi, however, it did not work.

#### whatweb

```
PHP 8.0.24

WordPress 5.6.2

nginx/1.18.0
```

#### ffuf Subdomain Scan

```
ffuf -u http://metapress.htb -H "Host: FUZZ.metapress.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 145
```

No results found.

#### wpscan

```
[+] robots.txt found: http://metapress.htb/robots.txt X

[+] The external WP-Cron seems to be enabled: http://metapress.htb/wp-cron.php X

[+] WordPress readme found: http://metapress.htb/readme.html X

[i] User(s) Identified: X

[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://metapress.htb/wp-json/wp/v2/users/?per_page=100&page=1
 |  Rss Generator (Aggressive Detection)
 |  Author Sitemap (Aggressive Detection)
 |   - http://metapress.htb/wp-sitemap-users-1.xml
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] manager
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

Possible Users:

```
admin
manager
```

#### robots.txt

```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php

Sitemap: http://metapress.htb/wp-sitemap.xml
```

#### XML Sitemap

<figure><img src="../../../.gitbook/assets/image (9) (3) (5).png" alt=""><figcaption></figcaption></figure>

Enumerated these and did not end up finding anything. Rabbithole.

#### OSINT

PHP

{% embed url="https://github.com/advisories/GHSA-c43m-486j-j32p" %}

Wordpress

Wordpress 5.6.2 SQLi

{% embed url="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwidwd-K6sj-AhWqTjABHdUxBD4QFnoECAsQAQ&url=https%3A%2F%2Fwww.exploit-db.com%2Fexploits%2F50663&usg=AOvVaw3JKVpkcT3oQEzjegNRgSLy" %}



#### Attempted to signup for an account...

http://metapress.htb/wp-json/wp/v2/users/

<figure><img src="../../../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>

Registration is currently not allowed.

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
