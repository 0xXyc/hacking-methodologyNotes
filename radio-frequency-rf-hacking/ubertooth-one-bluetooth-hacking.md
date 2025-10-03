---
description: 07/21/2024
---

# Ubertooth One Bluetooth Hacking

## Installation (MacOS)

```
brew install ubertooth
```

## RTFM

{% embed url="https://ubertooth.readthedocs.io/en/latest/ubertooth_one.html" %}

## Quick n' Dirty Configuration and Setup

### Sniffing Bluetooth Traffic

First, we will need to utilize `mkfifo` to make a `pipe` for a temporary storage solution to store our traffic to be written for later analysis:

```
mkfifo /tmp/pipe
```

{% hint style="info" %}
If you are unfamiliar with pipes in computing, please check out this link:

[https://www.techtarget.com/whatis/definition/pipe](https://www.techtarget.com/whatis/definition/pipe)
{% endhint %}

Next, we will need to open up Wireshark to read the traffic in a GUI friendly manner:

```
wireshark
```

* Capture -> Options -> Manage Interfaces -> Pipes -> Click "+"
* Add `/tmp/pipe`
* Click "Ok" and then "Start"

You are now good to go, simply start sniffing on that pipe. You will see `/tmp/pipe` as a valid option within the interfaces list. Select it (double-click).

<figure><img src="../.gitbook/assets/image (215).png" alt=""><figcaption><p>Showcasing <code>/tmp/pipe</code> option</p></figcaption></figure>

Lastly, we will need to utilize `ubertooth-btle` in order to begin the capture:

```
ubertooth-btle -f -c /tmp/pipe
```

As long as you have ran this command and started the Wireshark capture using the pipe, you will be able to see nearby traffic!

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Wireshark Capture Example</p></figcaption></figure>
