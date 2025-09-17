---
description: Diving into the code...
cover: https://sevaa.com/app/uploads/2018/09/ft-image-static-analysis.png
coverY: 117
---

# 🔍 Static Analysis

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

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Contents of <code>/FridaLab</code></p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Contents of the <code>AndroidManifest.xml</code> file for the FridaLab APK using Android Studio</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Using <code>batcat</code> to quickly analyze our manifest file</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="335"><figcaption><p>Analyzing the APK file structure</p></figcaption></figure>

You can now see that we have access to all of the files pertaining to the source code for the FridaLab APK!

There are not any library files (`.so`) files, so we do not have to worry about loading these into Ghidra to analyze these shared object binary blobs for C-based native Android implementations.&#x20;

Once you have reached this point, we can begin analyzing the files for anything that we may be able to leverage in an attempt to exploit this application.

## Analyzing Code w/ Android Studio

Inside of the Gradle project file, we can begin diving into the code base to further our understanding of the application in search of vulnerabilities that we can exploit to obtain our goal.&#x20;

We will be focusing on all application logic files, no external resources. So, anything in `/app/src/main/java/uk/rossmarks/fridalab`.&#x20;

Let's start at the `MainActivity.java` file. MainActivity is the first screen that appears when the user launches the app. Other activities can also exist that are used to perform different actions. Each activity can start other activities.

<figure><img src="../../.gitbook/assets/image (187).png" alt=""><figcaption><p><code>MainActivity.java</code></p></figcaption></figure>

{% embed url="https://developer.android.com/guide/components/activities/intro-activities#oncreate" %}
`MainActivity.onCreate()`
{% endembed %}

### `MainActivity.java`

```java
package uk.rossmarks.fridalab;

import android.os.Bundle;
import android.support.v4.internal.view.SupportMenu;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import java.util.Random;
import java.util.Timer;
import java.util.TimerTask;

/* loaded from: classes.dex */
public class MainActivity extends AppCompatActivity {
    public int[] completeArr = {0, 0, 0, 0, 0, 0, 0, 0};

    public boolean chall03() {
        return false;
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.AppCompatActivity, android.support.v4.app.FragmentActivity, android.support.v4.app.SupportActivity, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_main);
        ((Button) findViewById(R.id.check)).setOnClickListener(new View.OnClickListener() { // from class: uk.rossmarks.fridalab.MainActivity.1
            @Override // android.view.View.OnClickListener
            public void onClick(View view) {
                if (challenge_01.getChall01Int() == 1) {
                    MainActivity.this.completeArr[0] = 1;
                }
                if (MainActivity.this.chall03()) {
                    MainActivity.this.completeArr[2] = 1;
                }
                MainActivity.this.chall05("notfrida!");
                if (MainActivity.this.chall08()) {
                    MainActivity.this.completeArr[7] = 1;
                }
                MainActivity.this.changeColors();
            }
        });
        challenge_06.startTime();
        challenge_06.addChall06(new Random().nextInt(50) + 1);
        new Timer().scheduleAtFixedRate(new TimerTask() { // from class: uk.rossmarks.fridalab.MainActivity.2
            @Override // java.util.TimerTask, java.lang.Runnable
            public void run() {
                int nextInt = new Random().nextInt(50) + 1;
                challenge_06.addChall06(nextInt);
                Integer.toString(nextInt);
            }
        }, 0L, 1000L);
        challenge_07.setChall07();
    }

    /* JADX INFO: Access modifiers changed from: private */
    public void changeColors() {
        TextView textView = (TextView) findViewById(R.id.chall01txt);
        TextView textView2 = (TextView) findViewById(R.id.chall02txt);
        TextView textView3 = (TextView) findViewById(R.id.chall03txt);
        TextView textView4 = (TextView) findViewById(R.id.chall04txt);
        TextView textView5 = (TextView) findViewById(R.id.chall05txt);
        TextView textView6 = (TextView) findViewById(R.id.chall06txt);
        TextView textView7 = (TextView) findViewById(R.id.chall07txt);
        TextView textView8 = (TextView) findViewById(R.id.chall08txt);
        if (this.completeArr[0] == 1) {
            textView.setTextColor(-16711936);
        } else {
            textView.setTextColor(SupportMenu.CATEGORY_MASK);
        }
        if (this.completeArr[1] == 1) {
            textView2.setTextColor(-16711936);
        } else {
            textView2.setTextColor(SupportMenu.CATEGORY_MASK);
        }
        if (this.completeArr[2] == 1) {
            textView3.setTextColor(-16711936);
        } else {
            textView3.setTextColor(SupportMenu.CATEGORY_MASK);
        }
        if (this.completeArr[3] == 1) {
            textView4.setTextColor(-16711936);
        } else {
            textView4.setTextColor(SupportMenu.CATEGORY_MASK);
        }
        if (this.completeArr[4] == 1) {
            textView5.setTextColor(-16711936);
        } else {
            textView5.setTextColor(SupportMenu.CATEGORY_MASK);
        }
        if (this.completeArr[5] == 1) {
            textView6.setTextColor(-16711936);
        } else {
            textView6.setTextColor(SupportMenu.CATEGORY_MASK);
        }
        if (this.completeArr[6] == 1) {
            textView7.setTextColor(-16711936);
        } else {
            textView7.setTextColor(SupportMenu.CATEGORY_MASK);
        }
        if (this.completeArr[7] == 1) {
            textView8.setTextColor(-16711936);
        } else {
            textView8.setTextColor(SupportMenu.CATEGORY_MASK);
        }
    }

    private void chall02() {
        this.completeArr[1] = 1;
    }

    public void chall04(String str) {
        if (str.equals("frida")) {
            this.completeArr[3] = 1;
        }
    }

    public void chall05(String str) {
        if (str.equals("frida")) {
            this.completeArr[4] = 1;
        } else {
            this.completeArr[4] = 0;
        }
    }

    public void chall06(int i) {
        if (challenge_06.confirmChall06(i)) {
            this.completeArr[5] = 1;
        }
    }

    public void chall07(String str) {
        if (challenge_07.check07Pin(str)) {
            this.completeArr[6] = 1;
        } else {
            this.completeArr[6] = 0;
        }
    }

    public boolean chall08() {
        return ((String) ((Button) findViewById(R.id.check)).getText()).equals("Confirm");
    }
}

```

We can see that this `MainActivity` is the primary UI when the application is in use. Also judging from the code, it appears that the UI will change based on the completion of the challenges.

### `challenge_01.java`

```java
package uk.rossmarks.fridalab;

/* loaded from: classes.dex */


public class challenge_01 {
    static int chall01;

    public static int getChall01Int() {
        return chall01;
    }
}
```

This Java file defines a Java class named `challenge_01`.

* `chall01` is a integer variable
* There is a method used to return the value of the variable `chall01`, `getChall01Int()`

### `challenge_06.java`

```java
package uk.rossmarks.fridalab;

/* loaded from: classes.dex */

public class challenge_06 {
    static int chall06;
    static long timeStart;

    public static void startTime() {
        timeStart = System.currentTimeMillis();
    }

    public static boolean confirmChall06(int i) {
        return i == chall06 && System.currentTimeMillis() > timeStart + 10000;
    }

    public static void addChall06(int i) {
        chall06 += i;
        if (chall06 > 9000) {
            chall06 = i;
        }
    }
}

```

Another Java class file, named `challenge_06`.

* `startTime()` is a method used to set the `timeStart` variable to the current time in milliseconds using `System.currentTimeMillis()` -- assuming this starts the timer for the beginning of the challenge
* `confirmChall06(int i)` is a method that takes `i` as an input and check if it matches the value of the static variable, `chall06`. If the current time (milliseconds) is greater than the `timeStart` + 10,000 milliseconds. Ultimately, if both of these conditions are met, it will return true, allowing us to pass the challenge.
* `addChall06(int i)` is a method that checks if the value of the variable `chall06` exceeds 9000, it will reset `chall06` to the value of `i`

### `challenge_07.java`

```java
package uk.rossmarks.fridalab;

/* loaded from: classes.dex */

public class challenge_07 {
    static String chall07;

    public static void setChall07() {
        chall07 = BuildConfig.FLAVOR + (((int) (Math.random() * 9000.0d)) + 1000);
    }

    public static boolean check07Pin(String str) {
        return str.equals(chall07);
    }
}

```

Yet another class file, `challenge_07` is in charge of managing the 7th challenge in the CTF.

We will need to input a PIN that is generated randomly and if it matches the value, we will pass the challenge.

* `setChall07()` is a method that generates a random PIN by using `BuildConfig.FLAVOR`. The way this works is by concatenating the build flavor with a random integer between 1000-9999
* `check07Pin(String str)` is a method that takes a `string` (`str`) as input and compares it with the generated pin number stored in the `chall07` variable. If it returns 'true', the challenge will be completed successfully.

### Conclusions

We can now use this logic going forward and utilize Frida for dynamic instrumentation and manipulate the application's logic at runtime.

{% content-ref url="using-frida.md" %}
[using-frida.md](using-frida.md)
{% endcontent-ref %}
