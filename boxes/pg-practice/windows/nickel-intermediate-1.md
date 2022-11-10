---
description: 11-09-2022
---

# Nickel (Intermediate)

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

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

### Port 80 - HTTP (Apache)

random text here

## Exploitation

### Name of the technique

## Privilege Escalation

### Local enumeration

### PrivEsc vector

## Proof
