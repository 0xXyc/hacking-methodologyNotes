---
description: 06/07/2024
cover: https://securitybeans.files.wordpress.com/2013/10/smart-vs-dumb.jpg
coverY: 0
---

# 🧩 Fuzzing

## Introduction

### References

{% embed url="https://bishopfox.com/blog/fuzzing-aka-fuzz-testing" %}
Awesome writeup
{% endembed %}

Fuzzing is the art of testing software in an automated fashion for a multitude of vulnerabilities. It allows you to identify and target an ingestion point of the application or target and allows you to "throw" strategic or random input data at it. The primary goal is to discover programming errors and other vulnerabilities that can be exploited by attackers.

It's a process of testing a variety of API's with generated data. Our targets like I said earlier are anything that ingests data.&#x20;

**This really helps with:**

* C/C++-based bugs within the low-level implementations of the target
* Performance-critical implementations such as networking or cryptography
* If implemented incorrectly, this can lead to UAF's, BOF's, memory leaks, etc.

However, for the above, keep in mind that this only applies to native libraries since Java's VM (JVM) can't include these kinds of bugs since there is no direct memory access.&#x20;

### Examples of Real Fuzzers

Both are smart fuzzers.

[LLVM LibFuzzer](https://www.llvm.org/docs/LibFuzzer.html): Open source and great when you have access to the source code.

[AFL++](https://aflplus.plus/): Use case is for strategic input, usually used when certain data is required under constraints, made mandatory, etc in order to be accepted.

## Fuzzing Techniques

There's <mark style="color:yellow;">two main types of fuzzing</mark> techniques; "<mark style="color:yellow;">dumb</mark>" and "<mark style="color:yellow;">smart</mark>" fuzzing.&#x20;

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt="" width="375"><figcaption><p>Lol</p></figcaption></figure>

### Code Coverage

This is an attempt on the fuzzer's behalf to track/increase how much of the target's codebase is actually reached by the fuzzer's input.

In other words, the more code that we are able to reach with inputs is directly indicative of an increase in attack surface. With this in mind, it will seem like code-coverage-guided (smart) fuzzers are superior, and honestly, you'd be right.

### Dumb Fuzzing

Dumb fuzzing will involve random mutations on inputs, will not give you any feedback. There is zero strategic input whatsoever. It simply randomly mutates and throws a series of random garbage data at the target's data ingester.&#x20;

It shines on blackbox testing or when no source or knowledge of the app is in effect. This should be early stages of your research, if any.

```
Fuzzer -> Application -> Debugger
        ^ data         ^ Crash occurs here
```

### Smart Fuzzing

Smart fuzzing is an excellent choice and is feedback driven, so whatever data you're getting back, it will be able to adapt to! It is driven in pursuit of code coverage, so getting deep within the code; increasing the attack surface.&#x20;

It also utilizes [_<mark style="color:yellow;">symbolic execution</mark>_](https://www.cs.cmu.edu/\~aldrich/courses/17-355-18sp/notes/notes14-symbolic-execution.pdf) -- this is still a blackbox to me and seems fictional, but it's actually real and still blows my mind to this day. It is a theoretical code path/execution without the need of actually executing code.

It uses in-memory fuzzing, snapshot fuzzing for paused execution points, breakpoints (on a specific instruction), address sensitization,  structure-aware fuzzing, and so much more.

> :bulb: For your awareness, sanitizers compile source code with sanitizers in order to trigger specific bugs.&#x20;

```
       -> data        -> Crash
Fuzzer -> Application -> Debugger
       <- Feedback
```

Based off of code coverage:

* Range of fuzzing
* Code blocks, functions, etc.

<mark style="color:yellow;">However, to utilize a Fuzzer, we need to build out a fuzzing harness</mark>!

## Building a Fuzzing Harness

We can utilize Ghidra to do this.

Find a function that you like.&#x20;

1. File
2. Export Program
3. C/C++
4. Rename \<file.c>
5. Now, open it up in a code editor

Remember, we want to focus on finding low-level bugs when using fuzzers, so likely we will have to spend some time reversing a shared object file (`.so`). From here, a quick and consistent location to look at is for Java Native Implementation (JNI) function calls. This is where Java calls upon C/C++ implementations from native code and vice-versa. Look for `COM`.&#x20;

#### When Looking for JNI Calls

1. Identify the overall number of JNI function calls and names -> Take note and document them
2. Utilize Ghidra's function call tree on each and note the incoming/outgoing function calls
3. Identify references to known-vulnerable functions (`gets`, `malloc`, `free`, `strcpy`, etc.)

**Check out this list of banned C functions:**

{% embed url="https://wiki.openstack.org/wiki/StarlingX/Security/Banned_C_Functions" %}

### Building Harness and Fuzzing

The harness is made up of a few different parts:

1. The <mark style="color:yellow;">target</mark> is a function that you chose above
2. The <mark style="color:yellow;">corpus</mark> is a set of test inputs
3. The <mark style="color:yellow;">seed corpus</mark> is comprised of a series of inputs that are prepared for a fuzz target in order to obtain an initial coverage for discovery/enumeration purposes
4. The <mark style="color:yellow;">fuzzing engine/fuzzer</mark> is a tool that is used to find interesting inputs for a fuzz target by executing and ingesting data against it repeatedly
5. The <mark style="color:yellow;">harness</mark> connects to a fuzzer and the fuzzer's target

#### Overview/Illustration of a fuzzer and it's fuzzing harness

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption><p>Fuzzing Illustration</p></figcaption></figure>

### Give me Some EXAMPLES

Okay, LibFuzzer has a pre-defined function, known as, `FuzzerTestDneInput()`, where you simply just supply the target function to.

(e.g.)

```
FuzzerTestOneInput() {
    TargetFunctionHere();    # <-- This is your target here
    return 0;
}
```

**CLI:**

```
./fuzzer -dict=xml.dict -max_len=10000 fuzzer_corpus/ seeds/
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p>Running LibFuzzer against</p></figcaption></figure>

### Next Steps: Reproducing the Crash off of Previous Crash Data

After awhile, this will give you crash data upon successful crashes.

These are in `.xml` format where we can then feed it back to the LibFuzzer and reproduce the crash.

#### Reproducing Crash:

```
./fuzzer crash-83390246f84d717083da3ec375a23448dc952b56
```

### How Does the JNI Work?

It is possible due to the Native Development Kit (NDK) inside of the Android ecosystem.&#x20;

The NDK allows for native C/C++ code to run and coexist with Java and Kotlin code.

### Remember: APK Structure

This is useful to remember if you're feeling lost or overwhelmed.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>APK Illustration</p></figcaption></figure>

Remember, APK's can exist as `.apk`, `.xapk`, `.apks`, or even `.apkm`.
