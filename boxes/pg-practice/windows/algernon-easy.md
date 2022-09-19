---
description: 09-14-22
---

# Algernon (Easy)

### Improved skills

* <mark style="color:yellow;">I learned a VERY valuable skill here; if you enumerated THE ABSOLUTE HELL out of a box, check for public exploits</mark>.

### Used tools

* Nmap -- NmapAutomator.sh

## Information Gathering

Scanned all TCP ports:

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
9998/tcp  open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
17001/tcp open  remoting      MS .NET Remoting services
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Enumerated UDP ports:

```
```

Notes:

* I could not find anything for the longest time
* I attempted to enumerate the anonymous FTP but there was nothing lucrative

## Enumeration

### Port 9998

## Exploitation

### SmarterMail Exploit&#x20;

This is the exploit that I used to completely root the machine in one shot.

{% embed url="https://www.exploit-db.com/exploits/49216" %}
ExploitDB PoC
{% endembed %}

1. Modify the script (change LHOST, LPORT, RPORT, and HOST)
2. Start Netcat listener
3. Execute exploit
4. Shell!!

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption><p>PWND</p></figcaption></figure>

## Privilege Escalation

### - No privilege escalation was required.

## Proofs

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>
