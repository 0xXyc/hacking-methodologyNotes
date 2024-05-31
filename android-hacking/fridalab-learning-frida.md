---
description: 03/28/2024
cover: https://vicarius-marketing.s3.amazonaws.com/frida.png
coverY: 0
---

# 🌶️ FridaLab (Learning Frida)

## Introduction

Lately, I've been involved in efforts that require heavy usage of Frida. In this writeup, I'm going to be covering the walkthrough for FridaLab which is a vulnerable APK that you can practice your Frida skills on and can be found [here](https://rossmarks.uk/blog/fridalab/).

For emulation software, I am using [Genymotion](https://www.genymotion.com/product-desktop/download/).&#x20;

I highly recommend checking out Dark Wolf Solutions Android Security Research Playbook ([ASRP](https://github.com/DarkWolf-Labs/playbooks/blob/main/Android-Security-Research-Playbook.pdf)) for a reference guide for performing Android security research. This will be the foundation of my methodology going through this challenge!

Be sure to configure your Android device to communicate on a bridged, virtual network interface so you  can communicate with your security research VM.

### Getting Started

First, start off by downloading the APK from the source site above and install it to your virtual Android device.&#x20;

**You can automatically install the APK via:**

```
adb install FridaLab.apk
```

Then, navigate to the applications menu inside of your emulated device and you will see FridaLab. Go ahead and click on it.

<figure><img src="../.gitbook/assets/image (5) (1).png" alt="" width="302"><figcaption><p>FridaLab home screen</p></figcaption></figure>

We can now see that it features eight different challenges for us to get into and sharpen our skills.

Now we're able to begin the static analysis portion of our research on our target.&#x20;

**Click below to jump to the next section!**

{% content-ref url="fridalab-learning-frida/static-analysis.md" %}
[static-analysis.md](fridalab-learning-frida/static-analysis.md)
{% endcontent-ref %}
