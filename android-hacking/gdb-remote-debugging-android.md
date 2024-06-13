---
description: 06/10/2024
---

# GDB Remote Debugging (Android)

## Introduction

Want that extra extensibility and comfort from your own command line but your target's running on a remote device? Look no further.

{% hint style="info" %}
**Be sure to have `gdbserver` on the target device.**
{% endhint %}

### **Obtain PID of target application:**

```
adb shell ps -A | grep <name_here>

# Or if already in an adb shell session -- Specify -I if

ps -A | grep <name_here>
```

### **Kick off `gdbserver` on the target host:**

```
./<path_to_gdb_server> <ip_of_target_host>:5045 --attach <PID_of_app_to_be_debugged>
```

### **Connect to remote debugging session:**

```
gdb

target remote <ip_of_target_host>:5045

# or w/ gef

gef-remote <ip_of_target_host>:5045
```
