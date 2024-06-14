---
description: 06/11/2024 -- Time to wreck some havoc
cover: https://i.imgur.com/hxRHztm.gif
coverY: 44.74285714285714
---

# 🔎 Pinpointing Low-Level Bugs in Android

## Introduction

By leveraging Android's Native Development Kit (NDK), it will directly grant us access to the Java Native Implementation (JNI). Essentially, this allows us to be able to utilize native, C code within Java at the same time. JNI essentially is a mechanism that makes this possible. The NDK is built on top of the JNI.

{% hint style="info" %}
**Reminder: JNI is usually implemented when file I/O, sound, graphical rendering, encryption, etc. is taking place. This is because C/C++ simply does it better and quicker.**
{% endhint %}

## Oracle JNI Docs

{% embed url="https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/design.html" %}
Oracle JNI Docs
{% endembed %}

<figure><img src="../../.gitbook/assets/image (203).png" alt=""><figcaption><p>JNI Illustration</p></figcaption></figure>

As you can see, the JNI is simply just an array of pointers (pointer-to-pointer array). Each JNI function will include a `Java_OnLoad` or begin with `Java_com`. There are two ways of calling JNI functions, static and dynamic.&#x20;

### Static Calls

Uses the `RegisterNatives` API call.

**Java side:**

The method within the Java class is going to be identified as follows during declaration:

```java
public class MyClass {
    public static void myStaticMethod() {
        // Implementation here
    }
}
```

**C side:**&#x20;

```cpp
JNIEnv *env
jclass cls = (*env)->FindClass(env, "MyClass");
jmethodID mid = (*env)->GetStaticMethodIDenv, cls, "myStaticMethod", "()V");
(*env)->CallStaticVoidMethod(env, cls, mid);
```

### Dynamic Calls

Using JNI native method name resololving.

Search for "`Java_`" within Ghidra's symbol tree.

## Leveraging Memory Leak Bugs

While performing static analysis on the target, it is always important to recognize and document any data ingestion points. This will allow us to know what to look for when debugging various functions.&#x20;

### What Kind of Data can be Leaked?

* Data
* Addresses
* PII (sensitive data)
* Passwords
* Arming ourselves with data to bypass ASLR

### Bugs That Exist

* [Format String Bugs](../../binary-exploitation/format-string-vulnerabilities.md) will be the most common
* [Use-After-Free](../../binary-exploitation/heap-exploitation/use-after-free.md)'s
* Heap Overflows
* Type Confusion

## Leaking Data in Memory via Memory Leak

The main one will be format string bugs.&#x20;

This is simply when an attacker possesses the ability to control the format-string specifier in format-string functions. Ultimately, this leads to the ability to leak and write data, which leads to code execution. These vulnerabilities will allow us to not only leak data, but write data in memory.

### What to look for

**Vulnerable:**

```c
void vulnerable_function(char *user_input) {
    printf(user_input);  // Vulnerable to format string attack
}
```

**Not Vulnerable:**

```c
void secure_function(char *user_input) {
    printf("%s", user_input);
}
```

When data is requested, we the attackers, can pass our own format string specifiers. For example, `%lx`, `%s`, `%n` (write data).

{% hint style="info" %}
**Note: %x will just be 32-bit, whereas %lx will be a 64-bit value!**&#x20;
{% endhint %}

#### Don't Forget About Positional Arguments When Leaking Data!

We can utilize positional arguments to leak specific data off of the stack, straight to our console!

```
%110$lx # -> Prints the 110th element from the stack
```

<mark style="color:yellow;">The ultimate goal is to find a location on the stack that is not changing, or reoccurring a few times after a couple executions. So, it's okay to run the program more than once to verify, take your time.</mark>

**For example:**

After leaking, we can use `$lx` to leak and leverage positional arguments if need be as well.&#x20;

Once leaking, we can obtain the stack base, since our leak will "start" from the stack base, or stack pointer.

<figure><img src="../../.gitbook/assets/image (204).png" alt=""><figcaption><p>Obtaining stack base (<code>sp</code>)</p></figcaption></figure>

We can then look for reoccurring addresses within the stack dump (`x/200gx $sp`) or we can `vmmap` certain addresses.

Below, you can see that we have three addresses obtained from our stack dump information.

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption><p>Utilizing vmmap to obtain memory mapping/segment information</p></figcaption></figure>

### Leaking an Address

The goal here is to find the offset to the image base (find an offset from the leaked address to the base address). After that, we will possess enough information to calculate the base address.

1. Leak an address
2. Find a leaked libc library address in memory; confirming with `vmmap`

### Using the Following Equation

<mark style="color:yellow;">**Offset = Leaked\_address (%lx -- libc addr, verified w/ vmmap) - stack\_base (x/200gx $sp)**</mark>

**Example:**

<mark style="color:green;">**Offset**</mark>** = 0x0000007b5ed2f2bc - 0x789db11670**

**Offset = **<mark style="color:green;">**2C121DC4C**</mark>

### How To: Leak

1. Through RE efforts, locate vulnerable bugs and format-string functions
2. While performing dynamic analysis and debugging, be sure to `disass <function_name>`, obtain the address of the format-string function and place a breakpoint on it
3. Take note of the stack address by examining it: `x/200gx $sp`
4. Take some time to analyze the stack dump and look for addresses that are occurring next to each other, contiguously in memory. This value will be known as our "constant" leak address. As it will remain unchanged and constant throughout numerous executions
5. `vmmap` leaked addresses in order to find out if they belong to our dynamic library, (e.g.) libc

### Leaking a LIBC Address!

{% hint style="info" %}
**Note: This is literally the same process as before, except with an additional step (step 5)!**
{% endhint %}

1. Through RE efforts, locate vulnerable bugs and format-string functions
2. While performing dynamic analysis and debugging, be sure to `disass <function_name>`, obtain the address of the format-string function and place a breakpoint on it
3. Take note of the stack address by examining it: `x/200gx $sp`
4. Take some time to analyze the stack dump and look for addresses that are occurring next to each other, contiguously in memory. This value will be known as our "constant" leak address. As it will remain unchanged and constant throughout numerous executions
5. `vmmap` leaked addresses in order to find out if they belong to our dynamic library, (e.g.) libc
