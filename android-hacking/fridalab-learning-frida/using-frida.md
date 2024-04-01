---
description: Let's change some logic :D
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



<figure><img src="../../.gitbook/assets/image.png" alt="" width="420"><figcaption><p>Result</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Console output</p></figcaption></figure>

### Challenge #2: run chall02()

Since we went ahead and instantiated instance with `Java.choose()`, we can simply just call `instance.chall02()` in order to call `chall02()`. This works because we are hooking into `MainActivity` with instance at the start by searching the Java heap for any instances of that Java class within the target application.

`onMatch(instance)`: this function is called when an instance of the specified class (`MainActivity`) is found. It receives the instance via argument and assigns it to the `instance` variable.

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
        console.log(`\n\n[+] Challenge 1 completed! chall01 variable is now '1'!`);    

        // Challenge 2
        console.log(`\n\nCalling chall02...`);
        instance.chall02();
        console.log(`\n\n[+] Challenge 2 completed! chall02 called!`);
});
```

<figure><img src="../../.gitbook/assets/image (2).png" alt="" width="426"><figcaption><p>Result</p></figcaption></figure>

### Challenge #3: Make chall03() return true

Here, we will be introducing the `.implementation` to our script.

This essentially allows us to overwrite the `chall03()` method and allow us to be able to do whatever we want with it; introducing new logic. Ultimately, changing the app's logic.

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
        console.log(`\n\nStarting challenge 1!`);
        console.log(`\n\n[+] Setting class for Frida to load...`);
        const chall01 = Java.use('uk.rossmarks.fridalab.challenge_01');
        
        console.log(`\n\nModifying variable, chall01...`);
        chall01.chall01.value = 1;
        console.log(`\n\nVariable: `, + chall01.chall01.value);
        console.log(`\n\n[+] Challenge 1 completed! chall01 variable is now '1'!`);    

        // Challenge 2
        console.log(`\n\nStarting challenge 2!`);
        console.log(`\n\nCalling chall02...`);
        instance.chall02();
        console.log(`\n\n[+] Challenge 2 completed! chall02 called!`);

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
        console.log(`\n\nCalling chall02...`);
        instance.chall02();
        console.log(`\n\n[+] Challenge 2 completed! chall02 called!`);

        // Challenge 3
        console.log(`\n\nStarting challenge 3!`);
        const challenge03 = Java.use("uk.rossmarks.fridalab.MainActivity");
        challenge03.chall03.implementation = function(){ //hook chall03 method
            console.log('[+] Challenge 3 complete! Making chall03() return true...');
            return true;
        }

        // Challenge 4
        instance.chall04("frida");
        console.log('[+] Challenge 4 complete! Sending parameter "frida" to chall04().');

	});
});
```

<figure><img src="../../.gitbook/assets/image (4).png" alt="" width="412"><figcaption><p>Result</p></figcaption></figure>

### Challenge #5: Always send "frida" to chall05()

Coming soon.
