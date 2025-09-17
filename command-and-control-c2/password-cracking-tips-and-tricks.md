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

### What are rules?

Rules are a means of extending or manipulating _base_ words in a wordlist in ways that are common habits for users.

These manipulations can include toggling cases (e.g. `a` to `A`), character replacement (`a` to `@`), and prepending/appending characters (e.g. `password` to `password!`).

<mark style="color:$success;">This allows our wordlists to be overall smaller in size (because we don't have to store every permutation)</mark>, <mark style="color:yellow;">but with the drawback of a slightly slower cracking time</mark>.

```
hashcat.exe -a 0 -m 1000 ntlm.txt rockyou.txt -r rules\add-year.rule

acbfc03df96e93cf7294a01a6abbda33:Summer2020
```

**Where:**

* <mark style="color:yellow;">`-r rules\add-year.rule`</mark>: is our custom rule file

The rockyou list does not contain `Summer2020`, but it does contain the base word, `Summer`.

```
PS C:\> Select-String -Pattern "^Summer2020$" -Path rockyou.txt -CaseSensitive
PS C:\> Select-String -Pattern "^Summer$" -Path rockyou.txt -CaseSensitive

rockyou.txt:16573:Summer
```

The [hashcat wiki](https://hashcat.net/wiki/doku.php?id=rule_based_attack) contains all the information we need to write a custom rule that will append the year 2020 to each word in rockyou.  We can see that to append a character, we use `$X` - therefore to append "2020", we just need `$2$0$2$0`.

```
PS C:\> cat hashcat\rules\add-year.rule
$2$0$2$0
```

{% hint style="info" %}
`Hashcat` also ships with lots of rule files in the rules directory that you can use.
{% endhint %}

## Mask Attacks

A _**brute-force**_ is where we try all combinations from a given keyspace - for lowercase alphanumeric (of 3 characters), that would mean trying `aaa`, `aab`, `aac`... all the way to `zzz`. This is incredibly time consuming and not efficient.

&#x20;A mask, allows us to attack a password pattern in a much more efficient and strategic way. For instance, instead of using the full keyspace on the fist character, we can limit ourselves to uppercase only and likewise with the other positions.

```
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt ?u?l?l?l?l?l?l?l?d

64f12cddaa88057e06a81b54e73b949b:Password1
```

**Where:**

* `-a` `3` specifies the mask attack.
* `?u?l?l?l?l?l?l?l?d` is the mask.

**`hashcat --help` will show the charsets and are as follows:**

```
? | Charset
===+=========
l | abcdefghijklmnopqrstuvwxyz
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
d | 0123456789
h | 0123456789abcdef
H | 0123456789ABCDEF
s | !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
a | ?l?u?d?s
b | 0x00 - 0xff
```

You can combine these charsets within your mask for even more flexibility. It's also common for password to end with a special (such as `!`) rather than a number, but we can specify both in a mask.

```
hashcat.exe -a 3 -m 1000 ntlm.txt -1 ?d?s ?u?l?l?l?l?l?l?l?1

fbdcd5041c96ddbd82224270b57f11fc:Password!
```

**Where:**

* `-1 ?d?s` defines a custom charset (digits and specials).
* `?u?l?l?l?l?l?l?l?1` is the mask, where `?1` is the custom charset.

## Mask Length & Mask File

By default, this mask attack sets a static password length - `?u?l?l?l?l?l?l?l?1` defines nine characters. This means we can only crack a nine character password. To crack passwords of different lengths, we will need to adjust the mask accordingly.

**Hashcat mask files make this process a lot easier for custom masks that you use often:**

```
PS C:\> cat example.hcmask
?d?s,?u?l?l?l?l?1
?d?s,?u?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?l?1

hashcat.exe -a 3 -m 1000 ntlm.txt example.hcmask
hashcat (v6.1.1) starting...

Status...........: Exhausted
Guess.Mask.......: ?u?l?l?l?l?1 [6]

[...snip...]

Guess.Mask.......: ?u?l?l?l?l?l?1 [7]

820be3700dfcfc49e6eb6ef88d765d01:Chimney!
```

**Masks can even have static strings defined, such as a company name or other keyword you suspect are being used in passwords:**

```
ZeroPointSecurity?d
ZeroPointSecurity?d?d
ZeroPointSecurity?d?d?d
ZeroPointSecurity?d?d?d?d
```

```
hashcat.exe -a 3 -m 1000 ntlm.txt example2.hcmask

f63ebb17e157149b6dfde5d0cc32803c:ZeroPointSecurity1234
```

## Combinator Attack

This combines the entries from two dictionaries into a single-word candidate.&#x20;

**Take the following:**

```
PS C:\> cat list1.txt
purple

PS C:\> cat list2.txt
monkey
dishwasher
```

The combinator will produce "purplemonkey" and "purpledishwasher" as candidates.

You can also apply a rule to each word on the left or right-hand side using the options `-j` and `-k`.

For instance, `-j $-` and `-k $!` would produce `purple-monkey!`.

```
hashcat.exe -a 1 -m 1000 ntlm.txt list1.txt list2.txt -j $- -k $!

ef81b5ffcbb0d030874022e8fb7e4229:purple-monkey!
```

{% hint style="warning" %}
&#x20;If running in Linux, shells (`sh`, `bash`, `zsh`, `fish`, etc) will have their own behavior when the `$` character is used on the command line.  They may need to be quoted.
{% endhint %}

## Hashcat Modes: "Hybrid"

Hashcat modes 6 and 7 are hybrid's based on wordlists, masks, and the combinator.

You must specify both a wordlist and a mask and the mask is appended or prepended to the words within the list.

For example, if you have a dictionary that contains the word `Password`, then `-a 6 [...] list.txt ?d?d?d?d` will produce `Password0000` to `Password9999`.

```
hashcat.exe -a 6 -m 1000 ntlm.txt list.txt ?d?d?d?d

be4c5fb0b163f3cc57bd390cdc495bb9:Password5555
```

**Where:**

* `-a 6` specifies the hybrid wordlist + mask mode.
* `?d?d?d?d` is the mask.

&#x20;

The hybrid mask + wordlist mode (`-a 7`) is practically identical, where the mask comes first.

```
hashcat.exe -a 7 -m 1000 ntlm.txt ?d?d?d?d list.txt

28a3b8f54a6661f15007fca23beccc9c:5555Password
```

## kwprocessor

There is a number of external utilities that are separate from the main hashcat application.

Here is one called [`kwprocessor`](https://github.com/hashcat/kwprocessor).

This is a utility for generating key-walk passwords, which are based on adjacent keys such as `qwerty`, `1q2w3e4r`, `6yHnMjU7` and so on. To humans, these can look rather random and secure (uppers, lowers, numbers & specials), but in reality they're easy to generate programmatically.

**`kwprocessor` has three main components:**

1. **Base characters** - the alphabet of the target language.
2. **Keymaps** - the keyboard layout.
3. **Routes** - the directions to walk in.

There are several examples provided in the **basechars**, **keymaps** and **routes** directory in the `kwprocessor` download.

```
kwp64.exe basechars\custom.base keymaps\uk.keymap routes\2-to-10-max-3-direction-changes.route -o keywalk.txt

PS C:\> Select-String -Pattern "^qwerty$" -Path keywalk.txt -CaseSensitive

D:\Tools\keywalk.txt:759:qwerty
D:\Tools\keywalk.txt:926:qwerty
D:\Tools\keywalk.txt:931:qwerty
D:\Tools\keywalk.txt:943:qwerty
D:\Tools\keywalk.txt:946:qwerty
```

Some candidates will get generated multiple times, so you'll want to de-dup the list before using it for maximum efficiency.  This wordlist can then be used like any other dictionary in hashcat.

&#x20; Use `kwp64.exe --help` to see customisable options such as toggling the shift key.
