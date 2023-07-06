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

## Experimental -- needs fact checked (chatGPT Response)

{% code overflow="wrap" %}
```
Certainly! I apologize for the confusion in my previous response. ARM BTI (Branch Target Identification) is a feature introduced in ARMv8.5-A architecture to mitigate certain types of attacks that exploit the branch prediction mechanism in modern processors. It helps prevent speculative execution from speculatively accessing unintended or unauthorized branch targets.

ARM BTI works by introducing a new instruction called "BTI" (Branch Target Identification) and extending the existing branch instructions with BTI annotations. Here's how it works:

1. BTI Annotations: Software developers add BTI annotations to indirect branches in the code. An indirect branch is a branch instruction that does not specify a fixed target address but instead references a memory location where the target address is stored. BTI annotations provide information about the intended target of the branch to the processor.

2. Tagged Pointers: The branch targets are represented as tagged pointers, where the tag represents the trustworthiness of the target. A tag is a metadata associated with the target address that indicates its level of trust. It is derived from the execution context and is added to the pointer.

3. Branch Execution: When an indirect branch instruction with BTI annotation is encountered during program execution, the processor examines the tag associated with the target address. The tag determines the branch behavior.

   a. Trusted Branch: If the tag indicates that the target address is trusted, the processor speculatively executes the branch and continues with the speculative execution.

   b. Untrusted Branch: If the tag indicates that the target address is untrusted, the processor suppresses speculative execution and waits for the correct target address to be resolved before proceeding. This prevents the processor from speculatively accessing unintended or unauthorized branch targets.

4. BTI Instruction: The BTI instruction is used to set the policy for the branch target tags. It allows software to specify whether a particular branch target is trusted or untrusted.

   a. Setting Branch Target as Trusted: The BTI instruction can be used to set a branch target as trusted, allowing speculative execution to proceed. This is appropriate when the target address is known to be safe.

   b. Setting Branch Target as Untrusted: The BTI instruction can be used to set a branch target as untrusted, preventing speculative execution until the correct target address is resolved. This is useful when the target address is derived from user input or other potentially untrusted sources.

By incorporating BTI annotations and tagged pointers, ARM BTI provides a mechanism to enforce branch target identification and control speculative execution. It helps mitigate the risk of certain attacks, such as Spectre variants, that exploit speculative execution and branch prediction to leak sensitive information.
```
{% endcode %}
