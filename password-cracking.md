---
description: The art of password cracking with John, Hashcat, and Hydra.
---

# 🔓 Password Cracking

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

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.78.56 ftp
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
