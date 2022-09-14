---
description: 09-14-22
---

# Squid (Easy)

### Improved skills

* Skill 1
* Skill 2

### Used tools

* Nmap -- NmapAutomator.sh

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE    VERSION
3128/tcp open  http-proxy Squid http proxy 4.14
|_http-server-header: squid/4.14
```

Enumerated UDP ports:

```
```

Notes:

Why is it taking so long to scan this host? Why are we barely getting any information back?

* <mark style="color:yellow;">We were in front of a proxy that's why</mark>

### Squid Proxy Research

Squid is a caching and forwarding HTTP web proxy. It has a ton of uses such as speeding up a web server by caching requests, caching web, DNS, and other lookups.

In other words, it can make it very difficult to see what is actually going on behind the proxy without particular configurations. Luckily, in our case, I found a tool called "spose" that allows us to detect open ports behind the proxy.

{% embed url="https://github.com/aancw/spose" %}
GitHub Repository
{% endembed %}

## Enumeration

### Port 3128 - HTTP (Squid 4.14)

Visual Inspection

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

### Version Number Exploit Enumeration

* Nothing came back for <mark style="color:red;">Searchsploit</mark>
* Google provided me a lot more results
  * However, this was a rabbit hole

### "Spose"

Syntax

```
python3 spose.py --proxy http://192.168.75.189:3128/ --target 192.168.75.189                                                                                  Wed 14 Sep 2022 04:53:28 AM EDT

Using proxy address http://192.168.75.189:3128/
192.168.75.189 3306 seems OPEN 
192.168.75.189 8080 seems OPEN
```

* We see that by using the proxy port scanner, we get 2 additional ports open that we can detect
* This is great, however, we still do not have a way of natively visiting these ports

### Proxy Foxy

By adding the following configuration to Proxy Foxy, we can navigate to the 8080 port

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption><p>Proxy Foxy Configuration</p></figcaption></figure>

### Port 8080 - HTTP Wampserver&#x20;

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>Wampserver</p></figcaption></figure>

* After adding the proxy configuration, we are able to navigate to the site

### Dirsearch (Behind Proxy)

```
dirsearch --proxy http://192.168.75.189:3128 -u 192.168.75.189:8080                                                                                     156ms  Wed 14 Sep 2022 05:03:43 AM EDT

  _|. _ _  _  _  _ _|_    v0.4.2                                                                                                                                                                                  
 (_||| _) (/_(_|| (_| )                                                                                                                                                                                           
                                                                                                                                                                                                                  
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/xyconix/.dirsearch/reports/8080_22-09-14_05-03-55.txt

Error Log: /home/xyconix/.dirsearch/logs/errors-22-09-14_05-03-55.log

Target: http://192.168.75.189:8080/

[05:03:55] Starting: 
[05:03:57] 403 -  291B  - /%C0%AE%C0%AE%C0%AF                              
[05:03:57] 403 -  291B  - /%3f/                                            
[05:03:57] 403 -  291B  - /%ff                                             
[05:04:00] 403 -  291B  - /.htaccess.orig                                  
[05:04:00] 403 -  291B  - /.htaccess.sample
[05:04:00] 403 -  291B  - /.htaccess.bak1
[05:04:00] 403 -  291B  - /.htaccess_orig
[05:04:00] 403 -  291B  - /.htaccess_extra
[05:04:00] 403 -  291B  - /.htaccess_sc
[05:04:00] 403 -  291B  - /.htaccessBAK                                    
[05:04:00] 403 -  291B  - /.htaccess.save
[05:04:00] 403 -  291B  - /.ht_wsr.txt
[05:04:00] 403 -  291B  - /.html                                           
[05:04:00] 403 -  291B  - /.htm
[05:04:00] 403 -  291B  - /.htaccessOLD
[05:04:00] 403 -  291B  - /.htaccessOLD2                                   
[05:04:00] 403 -  291B  - /.htpasswds
[05:04:00] 403 -  291B  - /.htpasswd_test
[05:04:00] 403 -  291B  - /.httr-oauth
[05:04:07] 403 -  291B  - /Trace.axd::$DATA                                 
[05:04:15] 200 -    4KB - /adminer/                                         
[05:04:20] 403 -  291B  - /cgi-bin/                                         
[05:04:21] 500 -  634B  - /cgi-bin/printenv.pl                              
[05:04:28] 200 -  198KB - /favicon.ico                                      
[05:04:30] 200 -    6KB - /index                                            
[05:04:30] 200 -    6KB - /index.php                                        
[05:04:30] 403 -  291B  - /index.php::$DATA                                 
[05:04:30] 200 -    6KB - /index.pHp                                        
[05:04:30] 200 -    6KB - /index.php/login/
[05:04:30] 200 -    6KB - /index.php.
[05:04:38] 200 -    1KB - /phpmyadmin/README                                
[05:04:38] 200 -   15KB - /phpmyadmin/doc/html/index.html                   
[05:04:38] 200 -   27KB - /phpmyadmin/ChangeLog                             
[05:04:38] 301 -  338B  - /phpmyadmin  ->  http://192.168.75.189:8080/phpmyadmin/
[05:04:39] 200 -    2KB - /phpsysinfo/                                      
[05:04:39] 200 -   15KB - /phpmyadmin/                                      
[05:04:40] 200 -   15KB - /phpmyadmin/index.php
[05:04:43] 200 -    2KB - /shell                                            
[05:04:43] 200 -    2KB - /shell.php
[05:04:43] 200 -    2KB - /shell/
```

* Notice how I had to use the -proxy flag



## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
