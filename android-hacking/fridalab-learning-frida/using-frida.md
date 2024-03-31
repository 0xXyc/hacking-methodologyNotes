---
description: The fun part!!!
cover: https://redfoxsec.com/wp-content/uploads/2022/04/root-bypass-new-1.png
coverY: 72
---

# 🧨 Using Frida

## Introduction: Frida

### How Does Frida Work?

Frida utilizes the Java Virtual Machine (JVM) to write code directly into process memory.

Frida is also split into two main components, following the client/server model:

* Frida (on your attacking device)
* `frida-server` (installed on the target device)

When attaching Frida to an already running process, `ptrace` is used in the background to hijack the thread (A.K.A. _**ptrace-thread process injection**_).

A bootstrapper then populates this thread and starts a new one. This bootstrapper is then used to connect to the Frida-Server agent on the target device.

Simultaneously, a dynamic loaded library w/ the Frida agent is loaded within our instrumentation code.

The hijacked thread is then restored to its original state and resumed;  starting the process "normally".

{% embed url="https://medium.com/@pranav.s.paranjpe/introduction-to-frida-tool-b0b926ad3f59" %}
Medium Blog: Introduction to Frida
{% endembed %}

### Setting up Frida

{% embed url="https://frida.re/docs/installation/" %}
Installing Frida
{% endembed %}

The official Frida website is insanely in depth and offers lots of code examples, insight, and documentation on the [API](https://frida.re/docs/javascript-api/). Be sure to use it to your advantage when pwning Android apps!

After installed, we can confirm our version of Frida as well as confirm successful installation with:

<pre><code><strong>frida --version
</strong>16.2.1
</code></pre>

Since our Genymotion device is running on x86 (32-bit) architecture, we need to download this version of `frida-server` via direct [download](https://github.com/frida/frida/releases/download/16.2.1/frida-server-16.2.1-android-x86.xz). Frida source code and frida-server releases can be found [here](https://github.com/frida/frida/releases).

Before we get started, we will need to prepare `frida-server` before we send it over to our target device.

**Download w/ `wget`:**

```
wget https://github.com/frida/frida/releases/download/16.2.1/frida-server-16.2.1-android-x86.xz
```

**Decompressing and modifying permissions:**

```
xz -d frida-server-16.2.1-android-x86.xz

chmod +x frida-server-16.2.1-android-x86.xz
```

**Rename this binary to `frida-server`:**

```
mv frida-server-16.2.1-android-x86.xz frida-server
```

**Install Android Debug Bridge (ADB):**

We will utilize ADB to be able to communicate with our Genymotion device, transfer files, drop a shell, etc.

```
sudo apt install adb
```

**Transport the binary using ADB:**

```
adb push frida-server /data/local/tmp
```

**Start the `frida-server` binary:**

```
adb shell /data/local/tmp/frida-server &
<PID_pops_up_here>
```

Upon seeing the PID, you know you have successfully started `frida-server` on the target device.

