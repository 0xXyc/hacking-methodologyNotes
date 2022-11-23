---
description: 11/13/2022
---

# "Malbec" (Intermediate) - Incomplete

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74ba2023899262029fe73d3b83d4d96c (RSA)
|   256 548f79555ab03a695ad5723964fd074e (ECDSA)
|_  256 7f5d102762ba75e9bcc84fe27287d4e2 (ED25519)
2121/tcp open  ftp     pyftpdlib 1.5.6
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx   1 carlos   carlos     108304 Jan 25  2021 malbec.exe [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.81.129:2121
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
7138/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port7138-TCP:V=7.93%I=7%D=11/13%Time=63718396%P=x86_64-pc-linux-gnu%r(G
SF:enericLines,4,"\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
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
