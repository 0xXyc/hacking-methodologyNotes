---
description: 11/01/2023
---

# Adjusting Binary's Base Address to Match Ghidra During Debugging

## Introduction

Did you ever have to both reverse and debug a binary simultaneously but you found that switching back and forth addresses were a pain because they were not matching?

<mark style="color:yellow;">This is the fix for that</mark>!

## How-to

Fire up `gdb` and be sure to <mark style="color:yellow;">run your binary and break at a specific point</mark>, it doesn't matter where.

Run `vmmap` to list out all memory regions within the binary:

<figure><img src="../.gitbook/assets/image (176).png" alt=""><figcaption><p><code>0x555555554000</code> is our binary's base address</p></figcaption></figure>

Great, now that we have established that `0x555555554000` is our base address for our binary, we can add that information to Ghidra and align our memory addresses accordingly.

With your binary in Ghidra, select the "RAM" button in the menu bar:

<figure><img src="../.gitbook/assets/image (178).png" alt=""><figcaption><p>Memory Map</p></figcaption></figure>

Once inside, select the "home" button:

<figure><img src="../.gitbook/assets/image (179).png" alt=""><figcaption><p>Base Image</p></figcaption></figure>

Once inside, assign the base address the according value obtained from `gdb`:

<figure><img src="../.gitbook/assets/image (180).png" alt=""><figcaption><p>Base Image Address Established</p></figcaption></figure>

We can now see that our instructions and addresses are aligned accordingly! Ultimately, making our efforts way easier.

<figure><img src="../.gitbook/assets/image (181).png" alt=""><figcaption><p><code>gdb</code> disassembly</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (183).png" alt=""><figcaption><p>Ghidra Disassembly</p></figcaption></figure>
