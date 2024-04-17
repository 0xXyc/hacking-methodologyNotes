---
description: Venturing into new worlds... as always
---

# 🤖 Android Hacking

## Introduction

**Check out my team's playbook for getting into and performing Android Security research:**

{% embed url="https://github.com/DarkWolf-Labs/playbooks/blob/main/Android-Security-Research-Playbook.pdf" %}

## Android Ecosystem

The Android ecosystem is extremely fragmented in a multitude of ways. Mainly due to a wide variety of form factors, different operating system flavors (Linux-based), and manufacturers.&#x20;

With over three billion Android devices operational in the entire world, making up 71% of the global mobile operating system market share, understanding Android security in the world of cyber security has become unbelievably important. As a result, having a foundational knowledge base of vulnerability classes and exploitation in this area has become more paramount than ever.&#x20;

## Frida

* Frida is a dynamic code instrumentation toolkit
* It allows you to inject code directly during run time

### Installation

```
pipx install frida-tools
```

{% embed url="https://github.com/frida" %}
Official GitHub Repository
{% endembed %}

{% embed url="https://frida.re/docs/installation/" %}
Fridaaaaa
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

## Android Command-Line Tools (Ubuntu/Linux-- `apt`)

```
sudo apt-get install libc6-dev-i386 lib32z1 openjdk-8-jdk
```
