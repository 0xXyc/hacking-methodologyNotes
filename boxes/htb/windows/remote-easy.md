---
description: 09-26-22
---

# Remote (Easy)

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /blog/: Blog
|   /home.aspx: Possible admin folder
|   /contact/: Potentially interesting folder
|   /home/: Potentially interesting folder
|_  /intranet/: Potentially interesting folder
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
111/tcp   open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2049/tcp  open  mountd        1-3 (RPC #100005)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb-vuln-ms10-054: false
|_samba-vuln-cve-2012-1182: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
|_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
```

Enumerated UDP ports:

```
PORT     STATE SERVICE
111/udp  open  rpcbind
2049/udp open  nfs
```

Notes:

* 21/FTP - <mark style="color:yellow;">Attempted anonymous login regardless of nmap report and it allowed me to authenticate via anonymous</mark>.
  * Enumerated, and I found nothing
  * Can we place a file? -- No
* 80/HTTP - <mark style="color:yellow;">Microsoft Ajax Content Delivery Network -- jQuery 1.13.1 JS (Library)</mark>
  * Ran directory bruteforce x
  * Nikto scan
  * Vhost enumeration x
  * Subdomain enumeration x
  * Source code analysis -- I see all sorts of interesting stuff here
    * Wile E. Coyote
    * /cg16 x
    * /codegarden x
    * /umbraco
    * /great x

Script:

```
<script src="https://code.jquery.com/jquery-3.1.0.min.js" integrity="sha256-cCueBR6CsyA4/9szpPfrX3s49M9vUU5BgtiJj06wt/s=" crossorigin="anonymous"></script>
```

* Let's check out the rest of the site...
* /products x
* /People -- We get a list of people! x
  * Jan Skovgaard
  * Matt Brailsford
  * Lee Kelleher
  * Jeavon Leopold
  * Jeroen Breuer
* /umbraco looks the most suspicious

## Enumeration

### Port 80 - HTTP (MS Ajax)

Dirsearch:

```
Target: http://10.129.227.150/

[20:52:08] Starting: 
[20:52:12] 200 -    7KB - /Home                                            
[20:52:13] 200 -    5KB - /Products                                        
[20:52:13] 200 -    5KB - /Blog                                            
[20:52:15] 200 -    7KB - /People                                           
[20:52:15] 200 -    8KB - /Contact                                          
[20:52:19] 200 -    5KB - /blog                                             
[20:52:25] 200 -    8KB - /contact                                          
[20:52:36] 200 -    7KB - /home                                             
[20:52:37] 302 -  126B  - /install  ->  /umbraco/                           
[20:52:37] 200 -    3KB - /intranet                                         
[20:52:40] 500 -    3KB - /master                                           
[20:52:44] 200 -    7KB - /people                                           
[20:52:46] 200 -    3KB - /person                                           
[20:52:47] 200 -    5KB - /products                                         
[20:52:49] 500 -    3KB - /product                                          
[20:52:49] 400 -    3KB - /render/https://www.google.com                    
[20:52:56] 200 -    4KB - /umbraco 
```

* /umbraco looks to be the most interesting as it is a login page
* http://10.129.227.150/about-us/todo-list-for-the-starter-kit/ looks interesting as well. It has a to-do list in it

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
