---
description: 07/05/2023
---

# Shadow Stack (Intel & ARM)

## Introduction -- What is the shadow stack?

<mark style="color:yellow;">The shadow stack is simply an additional stack next to the call stack</mark>.

Its purpose is to <mark style="color:yellow;">track the return addresses of function calls</mark>.

This is so that it can monitor for control flow hijacking attacks.

Overall, this helps mitigate attacks such as <mark style="color:yellow;">Return-Oriented Programming (ROP)</mark>, <mark style="color:yellow;">Call-Oriented Programming (COP)</mark>, and <mark style="color:yellow;">Jump-Oriented Programming (JOP)</mark>.

### Intricacies

Rather than mirroring the entire call stack, <mark style="color:yellow;">the shadow stack simply stores a copy of the particular function's return address</mark>.

<mark style="color:yellow;">If both address records are different</mark>,  this indicates high likelihood of flow hijacking or some other attack.

#### What happens if a possible attack is discovered?

<mark style="color:red;">If this occurs a Control Flow (CP Exception) occurs, crashing the program, and throwing an exception</mark>.

The exception is as follows: `FAST_FAIL_CONTROL_INVALID_RETURN_ADDRESS`

## Architectures

<mark style="color:yellow;">Intel uses Control-Flow Enforcement Technology, A.K.A. CET</mark>.

<mark style="color:yellow;">ARM uses Branch Target Identification, A.K.A. BTI</mark>.

## Intel: Control Flow Enforcement Technology (CET)

### Shadow Stack

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

On the left, you can see the normal call stack.

On the right, you can see the shadow stack.&#x20;

Notice how the shadow stack only copies the return values on both stacks.

### CET High Level

1. Shadow Stack Pointer
2. Shadow Stack Operations
3. Return Value Comparison
4. Branch Tracking

### Multithreading??

<mark style="color:yellow;">If we recall, multithreading puts multiple threads within their own call stacks. How would the shadow stack keep track of a multithreaded program?</mark>

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The instructions `SAVEPREVSSP` and `RSTORSSP` are used to save and restore the shadow stack pointer for thread switching.

Therefore, it is saved in the thread upon exiting that thread, and when a new thread opens, the shadow stack pointer uses `RSTORSSP` to restore the shadow stack pointer.

### Endbranch (EB)

This is an additional feature found in Intel's CET.

The instruction in disassembly appears as: `endbr64`

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

In the first five instructions, we can see our shadow stack being established.

`endbr64` <mark style="color:yellow;">is used to mark valid jump target addresses of indirect calls and jumps into the program</mark>.

### Conclusion

<mark style="color:yellow;">CET offers hardware protection against ROP, JOP, and COP attacks which are used to manipulate control flow</mark>.

This works by reusing existing code in the program, libraries, etc. for malicious programs.

<mark style="color:yellow;">CET</mark> has <mark style="color:yellow;">2 major features</mark>:

1. <mark style="color:yellow;">Shadow stack</mark>: Used for tracking return addresses
2. <mark style="color:yellow;">Indirect branch tracking</mark> (`endbr64`)

### `gcc`: Compiling your Program with CET Protections

{% embed url="https://www.intel.com/content/www/us/en/docs/cpp-compiler/developer-guide-reference/2021-9/fcf-protection-qcf-protection.html" %}

Linux:&#x20;

```
gcc binary.c -o binary -fcf-protection=full
```

Windows:

```
gcc binary.c -o binary -Qcf-protection=full
```

<mark style="color:yellow;">The</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`full`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">option will enable both the shadow stack and end branch protections</mark>.

## ARM: Branch Target Identification (BTI)

{% embed url="https://developer.arm.com/documentation/ddi0596/2021-12/Base-Instructions/BTI--Branch-Target-Identification-" %}

{% embed url="https://www.phoronix.com/news/ARM64-Linux-5.8-BTI-SCS" %}
