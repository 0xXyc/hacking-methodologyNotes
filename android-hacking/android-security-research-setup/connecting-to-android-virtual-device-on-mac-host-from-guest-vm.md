---
description: 07/07/2024
---

# Connecting to Android Virtual Device on Mac Host From Guest VM

Solving typical unfortunate Mac obstacles, one at a time...&#x20;

## Introduction

So, to make a long story short, I was facing the issue of actually wanting to maintain a secure environment on my Mac while performing Android security research from the safety and comfort of my Ubuntu security research VM. Unfortunately, there is no native method to access a hypervisor from the VM itself and Android Studio AVD's as well as Genymotion Desktop is not available or possible from AARCH64 at the time of this blog post. Here is my work around for it!

I believe this will work for Android Studio, but decided to stick with Genymotion for this one. Each Genymotion Android device will be hosted on `127.0.0.1:6555`.&#x20;

## Fixing This Issue

We will need to perform <mark style="color:yellow;">SSH port forwarding</mark> in order to get around this!

We will need to first enable remote access on your Mac before this will work.&#x20;

{% hint style="info" %}
Now, if you're paranoid like me, it's probably best to turn this on/off as needed. You know, for obvious reasons.&#x20;
{% endhint %}

### Enabling Remote Access on Your Mac

**Open up settings, search for "sharing" and enable "Remote Login":**

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Illustration of above instructions</p></figcaption></figure>

### Authenticating

We can now go into our Ubuntu VM and SSH into our Mac using `user@Mac_IP_Here`.

**Once we have done that, we can now forward the connection via:**

```
ssh -L localhost:15555:127.0.0.1:6555 user@Mac_IP_Here
```

**Awesome, we can now connect to our device via:**

```
adb connect localhost:15555

adb devices

adb shell
```

Congrats! Enjoy your research!
