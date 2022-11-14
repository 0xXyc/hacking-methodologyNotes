---
description: 09-14-22
---

# Billyboss (Intermediate) - Incomplete

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-phpmyadmin-dir-traversal: 
|   VULNERABLE:
|   phpMyAdmin grab_globals.lib.php subform Parameter Traversal Local File Inclusion
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2005-3299
|       PHP file inclusion vulnerability in grab_globals.lib.php in phpMyAdmin 2.6.4 and 2.6.4-pl1 allows remote attackers to include local files via the $__redirect parameter, possibly involving the subform array.
|       
|     Disclosure date: 2005-10-nil
|     Extra information:
|       ../../../../../etc/passwd not found.
|   
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2005-3299
|_      http://www.exploit-db.com/exploits/1244/
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-majordomo2-dir-traversal: ERROR: Script execution failed (use -d to debug)
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
8081/tcp  open  http          Jetty 9.4.18.v20190429
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-fileupload-exploiter: 
|   
|_    Couldn't find a file-type field.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-server-header: Nexus/3.21.0-05 (OSS)
| http-enum: 
|_  /robots.txt: Robots file
|_http-csrf: Couldn't find any CSRF vulnerabilities.
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_samba-vuln-cve-2012-1182: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
|_smb-vuln-ms10-061: Could not negotiate a connection:SMB: Failed to receive bytes: ERROR
|_smb-vuln-ms10-054: false
```

Enumerated UDP ports:

```
```

### Nikto

```
Starting nikto scan
                                                                                               
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.75.61
+ Target Hostname:    192.168.75.61
+ Target Port:        8081
+ Start Time:         2022-09-14 21:56:10 (GMT-4)
---------------------------------------------------------------------------
+ Server: Nexus/3.21.0-05 (OSS)
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ "robots.txt" contains 3 entries which should be manually viewed.
+ Server banner has changed from 'Nexus/3.21.0-05 (OSS)' to 'Jetty(9.4.18.v20190429)' which may suggest a WAF, load balancer or proxy is in place
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ 7919 requests: 0 error(s) and 2 item(s) reported on remote host
+ End Time:           2022-09-14 22:14:00 (GMT-4) (1070 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

Finished nikto scan
```

Notes:



## Enumeration

### Port 80 - HTTP (Apache)

random text here

## Exploitation

### Name of the technique

This is the exploit

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proofs
