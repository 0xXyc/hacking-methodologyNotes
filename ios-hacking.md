---
description: 07/22/2024
---

# iOS Hacking

## Introduction

Have an old iPhone lying around? Want to break into that sucker?

{% hint style="info" %}
<mark style="color:yellow;">**Note**</mark>: This is a very entry-level introduction into iOS instrumentation and hacking. Treat this as a get-started, definitely not an all inclusive guide. I will be performing this research on an Apple Silicon Mac.
{% endhint %}

## Connecting Your Phone

Connect your iOS device to your attacking/research device via USB-A or USB-C.&#x20;

We can utilize a media player such as <mark style="color:yellow;">**QuickTime Player**</mark> in order to view the screen of our iPhone.&#x20;

After that, we can then select **File -> New Movie Recording -> Click the drop-down arrow and select your phone under the "screen", subsection.**

<figure><img src=".gitbook/assets/image (216).png" alt="" width="272"><figcaption><p>You will then be able to see your device!</p></figcaption></figure>

## "Jailbreaking" Your iOS Device

I cannot begin to harp this enough, do NOT do this on your actual phone, especially, if you are "security" conscious. This should make you extremely paranoid and it's best if you do NOT have any personal data on the phone while performing research on it.

### Check This Video out

{% embed url="https://www.youtube.com/watch?v=OqxWrLxxJps" %}
Great Video
{% endembed %}

### The Checkm8 Jailbreak Exploit

This is a permanent, BootROM exploit. This is a physical access exploit that requires, physical, tangible, access and DFU mode in order to trigger the exploit. This is a tethered exploit, meaning it must be re-exploited upon each boot.&#x20;

{% hint style="info" %}
**Note:** This does not impact the Secure Enclave on your device. Meaning, biometrics cannot be pwned from this exploit.
{% endhint %}

### Palera1n Jailbreak

{% embed url="https://palera.in/download/?tab=macos" %}
Safe, reliable, checkm8 exploit&#x20;
{% endembed %}

#### Installation

**Simply run the following on your Mac's terminal:**

```
sudo /bin/sh -c "$(curl -fsSL https://static.palera.in/scripts/install.sh)"
```

**Then run:**

```
source ~/.zshrc
```

**You can now run:**

```
palera1n --help
```

### Jailbreaking the iOS Device

**With your phone connected to your device, we can now run:**

```
palera1n -l
```

<figure><img src=".gitbook/assets/image (218).png" alt=""><figcaption><p>Our device is now waiting to be placed in DFU mode</p></figcaption></figure>

Press <mark style="color:blue;">**Enter**</mark>, once ready.

Next, follow the on-screen directions. There will be a countdown for both directions to begin the exploit:

1. <mark style="color:yellow;">Hold the volume down button + the side</mark> (lock button) -> Hold until the timer runs out
2. Next, <mark style="color:yellow;">hold the volume down button, releasing the side</mark> (lock button) -> Hold until the timer runs out

<figure><img src=".gitbook/assets/image (219).png" alt=""><figcaption><p>Successful Exploitation within DFU Mode</p></figcaption></figure>

This should result in a successful Checkm8 exploit and a tethered, jailbroken phone!

{% hint style="info" %}
**Note:** Remember, you will need to do this again with each reboot of the device since this is a tethered exploit!
{% endhint %}

<figure><img src=".gitbook/assets/image (220).png" alt=""><figcaption><p>Success!</p></figcaption></figure>

## Accessing New Jailbreak

If you are an OG "Jailbreaker", you definitely remember Cydia. This is a universal and notorious package manager and was essentially the jailbroken appstore.

We now have <mark style="color:yellow;">palera1n</mark>, which will grant us access to both <mark style="color:yellow;">Sileo</mark> and <mark style="color:yellow;">Zebra</mark> package managers.&#x20;

### Installing Sileo

**Open up the palera1n app and under "Install", select Sileo, and select Install:**

<figure><img src=".gitbook/assets/image (222).png" alt="" width="269"><figcaption><p>Installing Sileo -> If already installed, it will look similar to above</p></figcaption></figure>

It will then ask you to create a new `sudo` password. This password is important and will allow you to `ssh` into your device and escalate privileges as needed. Be sure to remember it!

## Sileo Usage

### Installing Frida

Now that we have Sileo, we can install `OpenSSH` and `Frida`.&#x20;

For Frida, we will need to go to Sources and select "+", in the top-right corner.

Here, we will need to add the "Source", `https://build.frida.re`.&#x20;

After that, select Frida and go to "Installed Packages". Then select Frida and install all dependencies.&#x20;

### Installing OpenSSH

Simply go to "Search" in the bottom-right and search for OpenSSH and follow the same directions as above to install.&#x20;

## SSH'ing Into iOS Device

We can now obtain our iOS device's IP address by going into the Settings app.

Wi-Fi -> Select blue "i" with a circle -> Scroll down and obtain the IP Address

This will be your device's private and unique IP address for your local network in which you will need in order to `ssh` into your device.

```
ssh root@<ip_here>
password: alpine
```

## Installing Frida on MacOS

```
brew install pipx

pipx install frida-tools

pipx install frida

pipx ensurepath

# Ensure successful installation
frida --version
16.4.6 # Version at the time of this post
```
