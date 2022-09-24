---
description: 09-23-22
---

# Nickel (Intermediate)

### Improved skills <a href="#improved-skills" id="improved-skills"></a>

*

Skill 1

*

Skill 2

### Used tools <a href="#used-tools" id="used-tools"></a>

*

Nmap -- NmapAutomator.sh

## Information Gathering <a href="#information-gathering" id="information-gathering"></a>

Scanned all TCP ports:​

```
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           FileZilla ftpd
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp   open  ssh           OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 86:84:fd:d5:43:27:05:cf:a7:f2:e9:e2:75:70:d5:f3 (RSA)
|   256 9c:93:cf:48:a9:4e:70:f4:60:de:e1:a9:c2:c0:b6:ff (ECDSA)
|_  256 00:4e:d7:3b:0f:9f:e3:74:4d:04:99:0b:b1:8b:de:a5 (ED25519)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=nickel
| Not valid before: 2022-09-22T23:14:25
|_Not valid after:  2023-03-24T23:14:25
|_ssl-date: 2022-09-23T23:18:55+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: NICKEL
|   NetBIOS_Domain_Name: NICKEL
|   NetBIOS_Computer_Name: NICKEL
|   DNS_Domain_Name: nickel
|   DNS_Computer_Name: nickel
|   Product_Version: 10.0.18362
|_  System_Time: 2022-09-23T23:17:49+00:00
8089/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
33333/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb2-security-mode: SMB: Couldn't find a NetBIOS name that works for the server. Sorry!
|_smb2-time: ERROR: Script execution failed (use -d to debug)
```

Enumerated UDP ports:

```
No UDP ports
```

​Notes:​

* FTP
  * No anon login, we need valid creds
* SSH
* RPC
* NetBIOS
* RDP
* HTTP on 8089
  * We can see a DevOps Dashboard and appears to be an API
  * However, upon clicking on some of the embeded links, we get not found errors
* HTTP on 33333
  * Upon accessing, we see "Invalid Token"
  * Intercepted request in burp and changed the GET request to a POST and we see "Not Implemented"

## Enumeration <a href="#enumeration" id="enumeration"></a>

### Port 21- FTP (FileZilla ftpd) <a href="#port-80-http-apache" id="port-80-http-apache"></a>

random text here

## Exploitation <a href="#exploitation" id="exploitation"></a>

### Name of the technique <a href="#name-of-the-technique" id="name-of-the-technique"></a>

This is the exploit

## Privilege Escalation <a href="#privilege-escalation" id="privilege-escalation"></a>

### Local enumeration <a href="#local-enumeration" id="local-enumeration"></a>

### PrivEsc vector <a href="#privesc-vector" id="privesc-vector"></a>

## Proofs <a href="#proofs" id="proofs"></a>
