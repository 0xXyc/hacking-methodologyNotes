---
description: 02/26/2026
---

# "Evil Twin" Attack

## Introduction

An "Evil Twin" Attack, A.K.A. a Rogue Access Point (AP), is a technique where an attacker will "clone" and masquerade as a legitimate access point to control traffic, harvest credentials, etc.

### How-to

There are plenty of frameworks that can allow us to perform "Evil Twin" attacks, but the one I like to use is [`wifiphisher`](https://github.com/wifiphisher/wifiphisher).

### Hardware

I am using an [ALFA Wireless AWUS036NH](https://www.google.com/url?sa=t\&source=web\&rct=j\&url=https%3A%2F%2Fwww.alfa.com.tw%2Fproducts%2Fawus036nh-eol_1\&ved=0CBoQjhxqFwoTCOC62tDr-JIDFQAAAAAdAAAAABAI\&opi=89978449).

<figure><img src="../.gitbook/assets/image (370).png" alt="" width="240"><figcaption></figcaption></figure>

This is necessary as it supports both monitor and injection modes.

### Download Additional Phishing Pages

Check out [here](https://github.com/wifiphisher/extra-phishing-pages) for additional phishing pages. This is what allowed me to serve a malicious Google login page form upon authenticating to my `StarbucksFreeWifi` ESSID.

### Usage

```
sudo wifiphisher --essid "StarbucksFreeWifi" -p google-login -pPD ~/evil-twin/extra-phishing-pages/ --nodeauth
```

<figure><img src="../.gitbook/assets/image (372).png" alt=""><figcaption></figcaption></figure>
