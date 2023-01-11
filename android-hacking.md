---
description: Venturing into new worlds... as always
---

# 🤖 Android Hacking

## Frida

* Frida is a dynamic code instrumentation toolkit
* It allows you to inject code directly during run time

{% embed url="https://github.com/frida" %}
Official GitHub Repository
{% endembed %}

{% embed url="https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/" %}
SSL Cert Pinning Bypass Script
{% endembed %}

### Basic Operations

```
frida-ls-devices

frida-ps-Ua

frida-trace -i "*" -U -f com.example.app

open/send/recv
```

{% embed url="https://erev0s.com/blog/frida-code-snippets-for-android/" %}
Cheat Sheet
{% endembed %}

## RootAVD

* This is an extremely useful script for loading APKs and rooting Android Virtual Devices (AVD's)
* This is what I've used on numerous projects and it is SOLID

{% embed url="https://github.com/newbit1/rootAVD" %}
Official GitHub Repository
{% endembed %}

## Android Debugging Bridge (ADB)

The ultimate toolkit for interfacing with Android devices (works with physical devices and emulation).

#### Installation:

{% embed url="https://www.xda-developers.com/install-adb-windows-macos-linux/" %}
Guide 1
{% endembed %}

{% embed url="https://developer.android.com/studio/releases/platform-tools" %}
Guide 2
{% endembed %}

## Android Command-Line Tools (Mac -- Homebrew)

`brew install --cask android-sdk`

#### Troubleshooting

adb push error permission denied:

Run `adb root`

Starting/Stopping ADB Server:

Run `adb kill-server`

Then, `adb start-server`
