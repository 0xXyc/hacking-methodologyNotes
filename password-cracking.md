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

## Hydra

### Bruteforcing

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.78.56 ftp
```









