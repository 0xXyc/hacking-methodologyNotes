---
description: Diving into the code...
cover: https://sevaa.com/app/uploads/2018/09/ft-image-static-analysis.png
coverY: 117
---

# 😉 Static Analysis

## Introduction: Static Analysis

This is a method of examining source code without actually executing the program. The main objective is to gain a further understanding of what the application is doing, how it is doing those things, as well as a deep dive into the design/implementation. Ultimately, this is done to pinpoint vulnerabilities within the code in order to create an exploit to fulfil the attacker's objectives.

{% embed url="https://asrp.darkwolf.io/ASRP-Plays/static" %}

## Diving into it

### Understanding the APK's Design and Compiler w/ APKiD (ASRP Play 05)

[APKiD](https://github.com/rednaga/APKiD) will allow us to gain a quick look at the APK to discover what technologies were used for the compilation of the binary itself ranging from detecting compilers, packers, obfuscators, and security techniques.

```
apkid -r --scan-depth 4 FridaLab.apk
[+] APKiD 2.1.5 :: from RedNaga :: rednaga.io
[*] FridaLab.apk!classes.dex
 |-> compiler : r8 without marker (suspicious)
```

&#x20;We can see from this output that the default Android compiler, R8 was used to compile Java bytecode into Dalvik (dex) byte code for this APK. We can also see that there is a `classes.dex` file, which is written in Java and then compiled into a class file, in charge of storing all of the application's logic. All APK's will contain this file.

### Decompress the APK w/ APKTool (ASRP Play 06)

[APKTool](https://apktool.org/) will allow you to decompress the APK and be able to obtain the Java/Dex source code for the application.&#x20;

```
apktool d -s FridaLab.apk
```

This will create a directory named `FridaLab`, where we can quickly load into our Integrated Development Environment (IDE) of choice such as Android Studio or JADX-GUI. Alternatively, you can just utilize utilities such as `cat` or `batcat` to view the `AndroidManifest.xml` file.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Contents of <code>/FridaLab</code></p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Contents of the <code>AndroidManifest.xml</code> file for the FridaLab APK using Android Studio</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>Using <code>batcat</code> to quickly analyze our manifest file</p></figcaption></figure>

### `AndroidManifest.xml` File

This `AndroidManifest.xml` file <mark style="color:yellow;">should serve as the primary starting point</mark> for the static analysis portion of your research with each new target.&#x20;

This is because each and every single APK to ever exist requires this file and it is responsible for containing essential metadata about the application itself.&#x20;

This includes the package name (`uk.rossmarks.fridalab`), activity names, the main activity (entry point for the application), hardware requirements, and exported activities (activities can be launched by other application components -- if true or the activity can be only launched by components of the same application -- if false).

### Decompile the APK w/ `jadx-gui` (ASRP Play 07)

We will now be utilizing [JADX](https://github.com/skylot/jadx) to decompile our binary while taking advantage of its tools to de-obfuscate and decompile its classes. Then, we will save it as a Gradle project and load it into Android Studio for examining the APK's filae structure.&#x20;

Be sure to follow the steps embedded in the ASRP or from JADX's GitHub to install.&#x20;

Open`FridaLab.apk` in `jadx-gui`.

**Then, follow the next few steps:**

* Select the option to _Open up a File_
* Go to _File->Preferences_, and make sure under _Decompilation_ that _Code Comments Level_ is set to DEBUG
* Go to _Tools->Decompile All Classes_
* Once the APK is decompiled, save this project onto your system as a **Gradle project folder** by going to _File->Save As Gradle Project -> I often rename this file to be `<application_name>-gradle`._

### Examine the APK File Structure Using Android Studio (ASRP Play 08)

We can now open up this Gradle project in Android Studio.

Select "Open..." and select the `FridaLab-gradle` file to import it into Android Studio.

<figure><img src="../../.gitbook/assets/image (4).png" alt="" width="335"><figcaption><p>Analyzing the APK file structure</p></figcaption></figure>

You can now see that we have access to all of the files pertaining to the source code for the FridaLab APK!

There are not any library files (`.so`) files, so we do not have to worry about loading these into Ghidra to analyze these shared object binary blobs for C-based native Android implementations.&#x20;

Once you have reached this point, we can begin analyzing the files for anything that we may be able to leverage in an attempt to exploit this application.

## Analyzing Code w/ Android Studio

Inside of the Gradle project file,&#x20;
