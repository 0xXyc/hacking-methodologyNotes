---
description: Having fun with dynamic instrumentation!
cover: https://redfoxsec.com/wp-content/uploads/2022/04/root-bypass-new-1.png
coverY: 72
---

# 🤖 Using Frida

## Introduction: Frida

### How Does Frida Work?

Frida utilizes the <mark style="color:yellow;">Java Virtual Machine</mark> (JVM) to write code directly into process memory.

Frida is also split into <mark style="color:yellow;">two main components</mark>, following the <mark style="color:yellow;">client/server</mark> model:

* Frida (on your attacking device)
* `frida-server` (installed on the target device)

When attaching Frida to an already running process, `ptrace` is used in the background to hijack the thread (A.K.A. _<mark style="color:yellow;">**ptrace-thread process injection**</mark>_).

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

### Troubleshooting Frida

During your testing, you may run into some issues. Notoriously, and most commonly, `frida-server` will crash and not run itself again. You will need to kill the process manually, and re-run the above command.

Obtain PID for `frida-server`:

```
adb shell ps | grep frida

root          6592   265   63024  34896 poll_schedule_timeout efec3bb9 S frida-server
```

With this information, we area able to obtain the PID for the `frida-server` process, which is the first number.

**We can then use this PID to kill the process by passing the corresponding identifier:**

```
adb shell kill -9 <PID_HERE>
```

**Lastly, restart `frida-server`:**

```
adb shell /data/local/tmp/frida-server &
```

### Using Frida

We can utilize `frida-ps` to list all running applications on the device:

```
frida-ps -Uai

 PID  Name                  Identifier                      
----  --------------------  --------------------------------
4499  Calendar              com.android.calendar            
4519  Clock                 com.android.deskclock           
1733  Email                 com.android.email               
2368  FridaLab              uk.rossmarks.fridalab   # <--- OUR TARGET        
2331  Gallery               com.android.gallery3d           
1226  Phone                 com.android.dialer              
   -  Amaze                 com.amaze.filemanager           
   -  Calculator            com.android.calculator2         
   -  Camera                com.android.camera2             
   -  Contacts              com.android.contacts            
   -  Custom Locale         com.android.customlocale2       
   -  Dev Tools             com.android.development         
   -  Development Settings  com.android.development_settings
   -  Files                 com.android.documentsui         
   -  Messaging             com.android.messaging           
   -  Music                 com.android.music               
   -  Search                com.android.quicksearchbox      
   -  Settings              com.android.settings            
   -  Superuser             com.genymotion.superuser        
   -  WebView Shell         org.chromium.webview_shell
```

We have now obtained our application identifier for the FridaLab app, `uk.rossmarks.fridalab`.

We <mark style="color:yellow;">need this information in order to spawn or attach Frida</mark> to our target application.

Spawning application (FridaLab) using Frida (without a Frida script):

This allows us to natively utilize Frida without using a Frida script at the application's start up.

```
frida -U -f uk.rossmarks.fridalab
```

<figure><img src="../../.gitbook/assets/image (188).png" alt=""><figcaption><p>Spawning target application with Frida instrumentation</p></figcaption></figure>

## Your First Frida Script

We will be using the Frida API when writing Frida scripts. This allows us to "tap into" the full power of Frida. As of the writing of this post, you can write Frida scripts in two different languages, JavaScript and Python; both leveraging the Frida API.&#x20;

The JavaScript approach simply just utilizes JavaScript and imports the Frida API via `Java.perform()`. This will attach the current thread to the JVM.

The Python approach is really interesting because it utilizes both Python and JavaScript syntax. However, for your JavaScript portion that you want to "embed" within your Python script, you simply create a wrapper around that code with (`"""<JS_CODE_HERE>"""`).

<mark style="color:yellow;">**NOTE**</mark>: Before we dive in, I was having some errors while running my finished script. This is due to all of the classes not being loaded prior to hooking. To fix this, simply execute `%reload` while in your Frida window and it will fix this issue. Also, for some changes to be reflected, the button must be pressed to invoke the correct methods and give you the successful output.

### Challenge #1: Change class challenge\_01's variable 'chall01' to: 1

```javascript
Java.perform(() => {
    Java.perform(function(){
		var instance;
		Java.choose("uk.rossmarks.fridalab.MainActivity",{
			onMatch : function(instancee){
				instance = instancee;
			},
			onComplete : function(){}
		});
	
	// Challenge 1
	// Load the class
        console.log(`\n\n[+] Setting class for Frida to load...`);
        let chall01 = Java.use('uk.rossmarks.fridalab.challenge_01');
        
        // Modify variable
        console.log(`\n\nModifying variable, chall01...`);
        chall01.chall01.value = 1;
        console.log(`\n\nVariable: `, + chall01.chall01.value);
        console.log(`\n\n[+] Challenge 1 passed! chall01 variable is now '1'!`);    
});
```

We need the APK package name and append the class name that we want to hook/load.

* `uk.rossmarks.fridalab.MainActivity`

I was pretty familiar to the concept of `Java.use`, but I never used `Java.choose`, until now.

**Use cases:**

* `Java.use`: assign the class to a variable that can be called by methods
* `Java.choose`: Searches the Java memory (heap) for any loaded instances in the memory of the class and hook into them; if it does not find one, a new instance will be created.

Next, we can easily modify the variable on the fly by finding the variable name from the source code, initializing that object, give it the property once more (which is just the variable name again), and give it the property, `value`. `chall01.chall01.value`.



<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="420"><figcaption><p>Result</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Console output</p></figcaption></figure>

### Challenge #2: run chall02()

Since we went ahead and instantiated instance with `Java.choose()`, we can simply just call `instance.chall02()` in order to call `chall02()`. This works because we are hooking into `MainActivity` with instance at the start by searching the Java heap for any instances of that Java class within the target application.

`onMatch(instance)`: this function is called when an instance of the specified class (`MainActivity`) is found. It receives the instance via argument and assigns it to the `instance` variable.

```javascript
        // Challenge 2
        console.log(`\n\nCalling chall02...`);
        instance.chall02();
        console.log(`\n\n[+] Challenge 2 completed! chall02 called!`);
});
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="426"><figcaption><p>Result</p></figcaption></figure>

### Challenge #3: Make chall03() return true

Here, we will be introducing the `.implementation` to our script.

This essentially allows us to overwrite the `chall03()` method and allow us to be able to do whatever we want with it; introducing new logic. Ultimately, changing the app's logic.

```javascript
        // Challenge 3
        console.log(`\n\nStarting challenge 3!`);
        const challenge03 = Java.use("uk.rossmarks.fridalab.MainActivity");
        challenge03.chall03.implementation = function(){ //hook chall03 method
            console.log('[+] Challenge 3 complete! Making chall03() return true...');
            return true;
        }
```

We must first hook the `chall03()` method and then we can overwrite the current implementation with our new one. Establish a new implementation of that function with `.implementation`.&#x20;

Lastly, set the return value to be of type, Boolean, returning true.

### Challenge #4: Send "frida" to chall04()

This one is another easy one due to our creative instance method we used with `Java.choose()` at the beginning of our script. Since we already instantiated the instance, we can just call `instance.chall04()` and pass `"frida"` as a parameter.

```javascript
Java.perform(() => {
    Java.perform(function(){
		var instance;
		Java.choose("uk.rossmarks.fridalab.MainActivity",{
			onMatch : function(instancee){
				instance = instancee;
			},
			onComplete : function(){}
		});
        
        // Challenge 4
        instance.chall04("frida");
        console.log('[+] Challenge 4 complete! Sending parameter "frida" to chall04().');

	});
});
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="412"><figcaption><p>Result</p></figcaption></figure>

### Challenge #5: Always send "frida" to chall05()

Here, we will be introducing the `overload` function. This little trick allows us to be able to change the parameter of a method and force it to always use what is specified. This sounds perfect for our challenge.

First, you start off by hooking the class you want to focus on, then obtain the method from the source code (in this case, `chall05()`), and then we are ready to call `.overload`, followed by a new `.implementation`.

```javascript
// Challenge 5
        console.log(`\n\nStarting challenge 5!`);
        const chall05 = Java.use("uk.rossmarks.fridalab.MainActivity");
        chall05.chall05.overload('java.lang.String').implementation = function(s){
            this.chall05("frida");
            console.log(`\n\n[+] Challenge 5 complete! Called chall05() with 'frida' as parameters`);
        }
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="381"><figcaption><p>Result</p></figcaption></figure>

### Challenge #6: Run chall06() after 10 seconds with correct value

Let's pull up the source code for this one.

**`challenge_06.java`**:

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

First, in order to manipulate this class, we need to hook it with `java.use`.

We can then use `.value` like we have before and modify the variable for `chall06` to be `1337`.&#x20;

We can then bypass the 10 second value by mofifying the value while the application is running and subtracting it by 10000 (10 seconds in milliseconds).

Then, call our instance with `chall06(<modified_value_here)`.

```javascript
 Java.perform(() => {
    Java.perform(function(){
		var instance;
		Java.choose("uk.rossmarks.fridalab.MainActivity",{
			onMatch : function(instancee){
				instance = instancee;
			},
			onComplete : function(){}
		});
 
 // Challenge 6
        console.log(`\n\nStarting challenge 6!`);

        var challenge06 = Java.use('uk.rossmarks.fridalab.challenge_06');
        challenge06.chall06.value = 1337; // Modify value of chall06 variable
        challenge06.timeStart.value = challenge06.timeStart.value - 10000; // Bypass 10 second wait time in milliseconds
        console.log(`\n\n[+] Calling chall06() with correct value`);
        instance.chall06(1337); // Call chall06 with correct value
        console.log(`\n\n[+] Challenge 6 completed. Called chall06(1) with our controlled value.`);
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="367"><figcaption><p>Result</p></figcaption></figure>

### Challenge #7: Bruteforce check07Pin() then confirm with chall07()

**`challenge_07.java`**:

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

This one was a doozy for me, however, we knew the correct PIN could only be 10000 combinations. By iterating through a 10000 integer values instantiated with the variable `i`. We know when to stop iterating because we hit the matched value from the PIN generated from integer value `i` using `check07Pin()`.

```javascript
Java.perform(() => {
    Java.perform(function(){
		var instance;
		Java.choose('uk.rossmarks.fridalab.MainActivity',{
			onMatch : function(instancee){
				instance = instancee;
			},
			onComplete : function(){}
		});

// Challenge 7
        console.log(`\n\nStarting challenge 7!`);
        
        console.log(`\n\nHooking into challenge_07 class!`);
        const challenge07 = Java.use('uk.rossmarks.fridalab.challenge_07');
        console.log(`\n\nIterating through values 0-9999 and checking for valid PIN...`);
        for (let i =0 ; i < 9999 ; i++) {
        if(challenge07.check07Pin(i.toString())) {
            console.log(`[+] PIN Found!`);
            instance.chall07(i.toString());
            console.log(`[+] Challenge 7 complete! The correct PIN is: ` + i);
            }
        }
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="365"><figcaption><p>Result</p></figcaption></figure>

### Challenge #8: Change 'check' button's text value to 'Confirm'

This one seems like it's really going to be an interesting one. Here, we are tasked with modifying one of the graphical components of the application itself directly within the UI. We will be changing 'Check' with 'Confirm'.

Since this UI component is a button, we need to first find a reference to it within the source code.&#x20;

This took me a bit of time to find, but by tracing through the source code, I was able to find the correct corresponding value to the button "check", obtain the hexadecimal resource ID, pass it to `instance.findViewById`, modify "check" with a `.$new` value of `"confirm"`, and boom, I was able to change the text on the button!

#### Tracing Source Code to Obtain the Resource ID

Through analyzing the MainActivity.java file, I was able to see references to `chall08`. Upon further inspection, I was able to see reference to `findViewById()` and a check for the string `.equals("Confirm")`. Meaning that the `R` class, must contain the button object. CTRL + Click on R to trace.

**Snippet of `MainActivity.java`:**

```java
 (...)
 public boolean chall08() {
        return ((String) ((Button) findViewById(R.id.check)).getText()).equals("Confirm");
    }
}
```

**Inside of the `R` class, we can see `check`:**

<figure><img src="../../.gitbook/assets/image (190).png" alt=""><figcaption><p>Identifying the button resource ID</p></figcaption></figure>

<mark style="color:yellow;">`0x7f07002f`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">is the resource ID</mark> that we can use to modify the button's text.

First, <mark style="color:yellow;">call the method</mark>, <mark style="color:yellow;">then we have to cast the result</mark> using `Java.cast`.

Now, by passing <mark style="color:yellow;">`0x7f07002f`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">to</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`findViewById()`</mark>, <mark style="color:green;">we can modify the button</mark>!

**Challenge 8:**

```javascript
Java.perform(() => {
    Java.perform(function(){
		var instance;
		Java.choose('uk.rossmarks.fridalab.MainActivity',{
			onMatch : function(instancee){
				instance = instancee;
			},
			onComplete : function(){}
		})

       // Challenge 8
        console.log(`\n\nStarting challenge 8!`);
        
        const button = Java.cast(instance.findViewById(0x7f07002f), Java.use('android.support.v7.widget.AppCompatButton'));
    	const string = Java.use('java.lang.String');
    	button.setText(string.$new("Confirm"));     // setText() and modify button
    	console.log("[+] PWNED!!!! Challenge08 Complete! The check button is now a confirm button! [+]");
```

<figure><img src="../../.gitbook/assets/image (189).png" alt=""><figcaption><p>Result</p></figcaption></figure>

Putting it all together, your entire script should look something similar to this in order to complete all of the challenges with a single run of your Frida script:

`FridaLab-Complete.js`:

```javascript
Java.perform(() => {
    Java.perform(function(){
		var instance;
		Java.choose('uk.rossmarks.fridalab.MainActivity',{
			onMatch : function(instancee){
				instance = instancee;
			},
			onComplete : function(){}
		});

        // Challenge 1
        console.log(`\n\nStarting challenge 1!`);

        console.log(`\n\n[+] Setting class for Frida to load...`);
        const chall01 = Java.use('uk.rossmarks.fridalab.challenge_01');

        console.log(`\n\nModifying variable, chall01...`);
        chall01.chall01.value = 1;
        console.log(`\n\nVariable: `, + chall01.chall01.value);
        console.log(`\n\n[+] Challenge 1 completed! chall01 variable is now '1'!`);    

        // Challenge 2
        console.log(`\n\nStarting challenge 2!`);

        console.log(`\n\nCalling chall02() method...`);
        instance.chall02();
        console.log(`\n\n[+] Challenge 2 completed! chall02 called!`);

        // Challenge 3
        console.log(`\n\nStarting challenge 3!`);

        const challenge03 = Java.use('uk.rossmarks.fridalab.MainActivity');
        challenge03.chall03.implementation = function() { //hook chall03 method
            console.log('[+] Challenge 3 complete! Making chall03() return true...');
            return true;
        }

        // Challenge 4
        console.log(`\n\nStarting challenge 4!`);

        instance.chall04("frida");
        console.log('[+] Challenge 4 complete! Sending parameter "frida" to chall04().');

        // Challenge 5
        console.log(`\n\nStarting challenge 5!`);

        const chall05 = Java.use('uk.rossmarks.fridalab.MainActivity');
        chall05.chall05.overload('java.lang.String').implementation = function(s) {
            this.chall05("frida");
            console.log(`\n\n[+] Challenge 5 complete! Called chall05() with 'frida' as parameters`);
        }

        // Challenge 6
        console.log(`\n\nStarting challenge 6!`);

        const challenge06 = Java.use('uk.rossmarks.fridalab.challenge_06');
        challenge06.chall06.value = 1337; // Modify value of chall06 variable
        challenge06.timeStart.value = challenge06.timeStart.value - 10000; // Bypass 10 second wait time in milliseconds
        console.log(`\n\n[+] Calling chall06() with correct value`);
        instance.chall06(1337); // Call chall06 with correct value
        console.log(`\n\n[+] Challenge 6 completed. Called chall06(1) with our controlled value.`)

        // Challenge 7
        console.log(`\n\nStarting challenge 7!`);
        
        console.log(`\n\nHooking into challenge_07 class!`);
        const challenge07 = Java.use('uk.rossmarks.fridalab.challenge_07');
        console.log(`\n\nIterating through values 0-9999 and checking for valid PIN...`);
        for (let i =0 ; i < 9999 ; i++) {
        if(challenge07.check07Pin(i.toString())) {
            console.log(`[+] PIN Found!`);
            instance.chall07(i.toString());
            console.log(`[+] Challenge 7 complete! The correct PIN is: ` + i);
            }
        }

        // Challenge 8
        console.log(`\n\nStarting challenge 8!`);
        
        const button = Java.cast(instance.findViewById(0x7f07002f), Java.use('android.support.v7.widget.AppCompatButton'));
    	const string = Java.use('java.lang.String');
    	button.setText(string.$new("Confirm"));     // setText() and modify button
    	console.log("[+] PWNED!!!! Challenge08 Complete! The check button is now a confirm button! [+]");

	});
});
```

## Future Goals

I stumbled upon this set of Frida-based challenged during my research. This looks like a slightly more complicated approach to FridaLabs and definitely want to look into it in the future. Check it out below!

{% embed url="https://github.com/DERE-ad2001/Frida-Labs" %}
Frida Labs -- next Frida challenge?
{% endembed %}
