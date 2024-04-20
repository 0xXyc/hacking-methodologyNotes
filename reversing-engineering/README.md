---
description: >-
  Reverse engineering is the process in which an artificial object is
  deconstructed to reveal its designs, architecture, code, or knowledge of the
  object.
---

# ⏪ Reversing Engineering

**Go from zero-to-strong intermediate reverse engineer!**

This will cover <mark style="color:yellow;">x86, x64, and 32-bit/64-bit ARM architectures!</mark>

Time to get after it!!!

## Building/Installing Ghidra on Mac w/ Apple Silicon (M1-M3)

I highly recommend using VM's for any/all reversing efforts, see below this section, or continue if you aren't hyper paranoid like me.

**Clone repo:**

```
git clone 
```

Now, `cd` into the `ghidra` master directory.

Use brew to install Gradle:

```
brew install gradle
```

**Install dependencies w/ Gradle:**

```
gradle -I gradle/support/fetchDependencies.gradle init
```

Build w/ Gradle:

```
gradle buildGhidra
```

**Obtain build:**

```
cd build/dist
unzip ghidra_11.1_DEV_20240420_mac_arm_64.zip # unzip 
```

**Run Ghidra:**

```
./ghidraRun
```

## Building/Installing Ghidra on Ubuntu AARM64 (MAC w/ Apple Silicon)

This absolutely sucked and took hours to debug... I used multiple Gradle versions and the only one that seemed to work for me was <mark style="color:yellow;">Gradle-7.6.2</mark>.

We do this because we want to be smart and segment our device from threats as much as possible, right?

Download and install Gradle from the official Gradle site.

**Be sure to move Gradle to /opt/gradle.**

```
export PATH=$PATH:/opt/gradle/gradle-7.6.2/bin
```

**Update PATH:**

```
source /home/<user>/.zshrc
```

**Clone Ghidra:**

```
git clone https://github.com/NationalSecurityAgency/ghidra.git
cd ghidra
```

**Build dependencies:**

```
gradle -I gradle/support/fetchDependencies.gradle init 
```

**Build Ghidra:**

```
gradle buildGhidra
cd build/dist
unzip ghidra_11.1_DEV_20240420_linux_arm_64.zip
cd ghidra_11.1_DEV
```

**Run Ghidra:**

```
./ghidraRun
```

Add `ghidraRun` as an alias, update source and BOOM, done.
