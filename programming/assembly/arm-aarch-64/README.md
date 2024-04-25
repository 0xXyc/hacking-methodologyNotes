---
description: 04/20/2024
---

# ⚒️ ARM AARCH 64

## Introduction

This is a quick little introduction into ARM Assembly. Why are we getting into ARM you might ask?

Well, if you're looking at certain trends in my research, you will notice that there has been a significant pivot towards mobile, embedded, IoT, and other devices of that nature. Guess what the vast majority of those devices run on? You guess it, the ARM architecture.

#### Ultimate Motivations

We want to obtain a solid understanding of ARM Assembly so we can write shellcode in ARM and start doing evil things at the lowest human-readable level possible. :imp:

### What is ARM?

<mark style="color:yellow;">ARM, or Advanced RISC Machine</mark> is clearly based on its predecessor, the RISC architecture which stands for Reduced Instruction Set Computer. This architecture is absolutely going to blow up in the scene of new devices, we are already seeing Apple placing them within all of their new devices, its only a matter of time everyone else follows suit. It is favored due to its simplicity and performance.

### CISC?

CISC, or the Complex Instruction Set Computing serves a different purpose for a multitude of applications mainly found in laptops, desktops, and servers. It is preferred in these systems because they are more complex and are able to pack more of a powerful + dynamic instruction set simultaneously.&#x20;

## ARM Assembly

There are two main instruction set <mark style="color:red;">modes</mark> in ARM, <mark style="color:red;">`THUMB`</mark> and <mark style="color:red;">`ARM`</mark> mode.

### THUMB Mode

* 16-bit instructions
* Byte code (`80` `b5`) will be converted directly into binary (0's and 1's)
* Instructions are human-readable to allow introspection into what exactly the CPU is doing

Example instruction (Binary):&#x20;

```
10010001010010001
^               ^
0               16
```

See how it is only 16-bits long?

### ARM Mode

* 32-bit instructions
* Byte code (00 00 00 ef) -> binary

Example instruction (Binary):

```
100100010100100011001000101001000
^                               ^
0                               32
```

<figure><img src="../../../.gitbook/assets/image (199).png" alt=""><figcaption><p>Depiction of offsets from the binary base address, instructions, and operands in Ghidra</p></figcaption></figure>

### Writing your first "hello world" in ARM ASM

So, if you are new to Assembly, this may come off a little confusing, but that's okay, give it some time and it will stick.

In order to get your program to work, you must do a couple of things, I will try my best to simplify this process as much as possible.

1. Assembly code (human-readable form) must first be assembled using an assembler.
2. Your assembled code will then be converted into a binary blob or object file (`.o` file). This new file consists of raw machine code and cannot be read by humans.
3. Your object file must then be linked to your assembly code. Where it is then converted into an executable format that your CPU/OS can understand and execute.

**Create ASM (`.s`) file:**

`hello.s`:

```armasm
.global _start

_start: 
        // Syscall ID for write
        mov x8, #64     // Place syscall number for write (64) in x8 register 
        mov x0, #1      // Place constant value (1) into x0 register
        ldr x1, =hello  // Load the string stored in hello in the x1 register
        mov x2, len     // Place the length of the hello string in x2 
        svc 0           // Perform syscall with supervisor call (svc) instr.

        // Syscall ID for exit
        mov x8, #93     // Move syscall number 93 for exit in x8 register
        mov x0, #0      // Mov constant value (0) in x0 register
        svc 0           // Perform syscall in privileged mode

.section .data
        hello: .ascii "My first ARM64 ASM Program!!!\n"
        len = . - hello


```

**Assemble assembly source code file:**

```
aarch64-linux-gnu-as hello.s -o hello.o
```

**Link binary object with source assembly code and create a linked/executable binary:**&#x20;

```
aarch64-linux-gnu-ld hello.o -o hello
```

Execute your new binary:

```
./hello
```

<figure><img src="../../../.gitbook/assets/image (200).png" alt=""><figcaption><p>Hello World!</p></figcaption></figure>

**Now, when you get really comfortable, you can conjoin the assemble and linking process in a one-liner command:**

{% code overflow="wrap" %}
```
aarch64-linux-gnu-as -mcpu=cortex-a57 hello.s -o hello.o && aarch64-linux-gnu-ld -mcpu=cortex-a57 hello.o -o hello
```
{% endcode %}
