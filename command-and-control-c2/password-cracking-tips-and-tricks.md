---
description: 09/09/2025
---

# Password Cracking Tips & Tricks

## Introduction

As we were able to see within the [obtaining-credentials.md](obtaining-credentials.md "mention")section, we can obtain credentials in the forms of hashes for a user, but sadly, <mark style="color:$danger;">this is not always in the form of a plaintext password</mark>.

Instead, <mark style="color:yellow;">it is more common to retrieve various hashes, rather than plaintext passwords</mark>.&#x20;

**These could be:**

* NTLM
* NetNTLM
* SHA
* Kerberos Tickets

Some hashes such as NTLM can be utilized as they are (they don't always need to be cracked offline).&#x20;

This means that we can use a technique such as _**Pass The Hash**_, but others are not so useful unless we can crack them to recover an original plaintext password.

Regardless of the type of hash, there are generic password cracking methodologies that will be covered in this section.

### Cracking Applications

**Two common applications to crack hashes are:**

1. [hashcat](https://hashcat.net/hashcat/) (both CPU and GPU-optimized, but GPU is more optimized)
2. [John the Ripper](https://www.openwall.com/john/) (both, but is a more efficient CPU password-cracking utility)

So, depending on which hardware component in your rig is the best, choose the right and most efficient/optimized tool that aligns with your setup.

{% hint style="success" %}
For fun and practice, you should take the hashes that we obtain throughout and crack them yourselves, just make sure you have a decent GPU/CPU.
{% endhint %}

## Wordlists

A wordlist or dictionary attack is the easiest mode of password cracking.

This simply iterates through a list of plaintext password candidates and attempts to try each password one-by-one.

There are many popular lists out there, including the venerable `rockyou` list.

The [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Passwords) repo also have an expansive collection for different applications.

```
hashcat.exe -a 0 -m 1000 ntlm.txt rockyou.txt

58a478135a93ac3bf058a5ea0e8fdb71:Password123
```

**Where:**

* `-a 0` specifies the wordlist attack mode
* `-m 1000` specifies that the hash is NTLM
* `ntlm.txt` is a text file containing the NTLM hash to crack
* `rockyou.txt` is the wordlist

{% hint style="info" %}
Use `hashcat.exe --help` to get a complete list of attack mode and hash types.
{% endhint %}

This cracks practically instantly because `Password123` is present in the wordlist:

```powershell
PS C:\> Select-String -Pattern "^Password123$" -Path rockyou.txt -CaseSensitive

rockyou.txt:33523:Password123
```

It's fast, it's not very flexible, since if the password is not in the list, we won't crack it.

### Wordlist Rules

