---
description: 11-09-2022
---

# Nickel (Intermediate)

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp    open  ssh           OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 8684fdd5432705cfa7f2e9e27570d5f3 (RSA)
|   256 9c93cf48a94e70f460dee1a9c2c0b6ff (ECDSA)
|_  256 004ed73b0f9fe3744d04990bb18bdea5 (ED25519)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2022-11-10T02:03:11+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=nickel
| Issuer: commonName=nickel
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-11-09T01:58:19
| Not valid after:  2023-05-11T01:58:19
| MD5:   ae87078529cc1047117f9699a5364c3c
|_SHA-1: 254de154bbf9a8bcc47973866f5187715b4341bc
| rdp-ntlm-info: 
|   Target_Name: NICKEL
|   NetBIOS_Domain_Name: NICKEL
|   NetBIOS_Computer_Name: NICKEL
|   DNS_Domain_Name: nickel
|   DNS_Computer_Name: nickel
|   Product_Version: 10.0.18362
|_  System_Time: 2022-11-10T02:02:04+00:00
8089/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
| http-methods: 
|_  Supported Methods: GET
|_http-favicon: Unknown favicon MD5: 9D1EAD73E678FA2F51A70A933B0BF017
33333/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-favicon: Unknown favicon MD5: 76C5844B4ABE20F72AA23CBE15B2494E
| http-methods: 
|_  Supported Methods: GET POST
|_http-title: Site doesn't have a title.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb2-time: ERROR: Script execution failed (use -d to debug)
|_smb2-security-mode: SMB: Couldn't find a NetBIOS name that works for the server. Sorry!
```

Enumerated UDP ports:

```
1000 open|filtered udp ports (no-response)
```

Notes:

* 21/FTP - HTA attack or other client-side attack? Permissions? Version
* 3389/RDP - <mark style="color:yellow;">Product\_Version: 10.0.18362</mark>
* 8089/HTTP - DevOps Dashboard
* 33333/HTTP - Blank as of discovery; "invalid token"

## Enumeration

### Port 8089 - HTTP "DevOps Dashboard"

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

* Let's check out List Running Processes as it seems the most interesting

### Unable to Connect

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Let's change the IP to the target's and try again:

<figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

* Interesting, the server is reporting that it cannot "GET" the directory
* What if we sent it a POST?

POST Request:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### SSH Enumeration

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Credentials:

User: ariah

Password: Tm93aXNlU2xvb3BUaGVvcnkxMzkK

* Base64 encoding?

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">ariah:NowiseSloopTheory139</mark>

Connecting to host via SSH:

```
ssh ariah@192.168.81.99
password: NowiseSloopTheory139
```

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

## Exploitation

### Sensitive Information Disclosure

* The list-running-procs function within the poorly configured web app pointed me to SSH credentials which granted me initial foothold of the victim machine

## Privilege Escalation

### Local enumeration

* Interesting file in <mark style="color:yellow;">C:/ftp/infrastructure.pdf</mark>
* <mark style="color:yellow;">Unable to utilize SMB or HTTP for file transfer, remembered SSH was open and utilized SCP for file transfer</mark>

### SCP File Transfer

```
scp ariah@192.168.81.99:C:/ftp/infrastructure.pdf .
ariah@192.168.81.99's password: NowiseSloopTheory139

Infrastructure.pdf                                                     100%   45KB 240.7KB/s   00:00 
```

* We can now access Infrastructure.pdf from our Kali box

Opening PDF:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* However, when I'm trying to open it, we find out that it is encrypted
  * Sounds like a <mark style="color:yellow;">pdf2john</mark> kind of mission

### pdf2john -- infrastructure.pdf

```
pdf2john Infrastructure.pdf > Infrastructure.pdf.hash
Infrastructure.pdf:$pdf$4*4*128*-1060*1*16*14350d814f7c974db9234e3e719e360b*32*6aa1a24681b93038947f76796470dbb100000000000000000000000000000000*32*d9363dc61ac080ac4b9dad4f036888567a2d468a6703faf6216af1eb307921b0
```

### PrivEsc vector

## Proof
