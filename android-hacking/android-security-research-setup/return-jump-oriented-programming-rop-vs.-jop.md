---
description: 06/18/2024
cover: ../../.gitbook/assets/Screenshot 2024-06-18 at 6.11.33 PM.png
coverY: 613.5963508771929
---

# 🤾‍♂️ Return/Jump-Oriented Programming -- ROP vs. JOP

## Introduction

ROP/JOP is a technique that enables attackers to bypass exploit mitigations. Such as the No-eXecute (NX) bit.&#x20;

[no-execute-nx.md](../../binary-exploitation/memory-protections/no-execute-nx.md "mention")

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>ROP/JOP Monster (AI-gen)</p></figcaption></figure>

ROP/JOP gadgets are short sequences of machine instructions (machine code) that end in return or branch instructions.&#x20;

We can obtain gadgets from either the binary (program) itself, or from the dynamically-loaded library, such as `libc.so`.

Various gadgets will either end in a `ret` (return) or a `blr` ( branch & link register) instruction.&#x20;

Gadgets ending with `ret` instructions will be ROP gadgets.&#x20;

Gadgets ending in `blr` instructions will be JOP gadgets.&#x20;

### References

{% embed url="https://tyagi-bhaumik.medium.com/return-oriented-programming-vs-jump-oriented-programming-f5571c44b910" %}

{% embed url="https://developer.arm.com/documentation/102433/latest/Jump-oriented-programming" %}

### Why?

Why you might ask... well, gadgets can be "chained" together in order to leverage already existent code within the binary.&#x20;

They are essentially a "program within a program".

### Examples

**Be sure to check out my older blog on ROP chains.**

[rop-chains-101.md](../../binary-exploitation/return-oriented-programming-rop/rop-chains-101.md "mention")

{% hint style="info" %}
**Note: We can use a ROP chain to fulfill register values to obtain reverse shells or some other cyber effect on your target.**
{% endhint %}

## Obtaining Gadgets

We can utilize `gdb` to manually obtain ROP/JOP gadgets.

We can also utilize a tool known as [`ropper`](https://github.com/sashs/Ropper).&#x20;



### GDB

We can utilize `gdb` to dump memory and scan for ROP gadgets!

Load up your program in `gdb`.

Perform the following:

```
set logging enabled on

info sharedlibrary libc.so    # Copy libc base address (top-left)

x/30000i <library_base_address>     # Paste libc base address

q

more gdb.txt | grep ldp
```

Essentially, this is an alternative for `ropper`!

### Utilizing Ropper

```
ropper --help-examples

ropper -f </path/to/libc.so> --search "ldp x0"

# Search for "pop x0" for system(x0 (first parameter) -> ("/bin/sh"))
```
