---
description: 04/14/2024
cover: https://i.pinimg.com/736x/df/47/ee/df47ee3e5df1f4abcc0766585b493ba3.jpg
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# The Beauty of Programming Languages (A low-to-high abstraction)

## Introduction -- Why am I writing about this?

Okay all jokes aside, programming is still a skill that I am working on as a self-taught coder. I by no means am an expert, yet, but I will always strive to make others lives easier in any way I can throughout their security journey. Although it may appear daunting to beginners at first when they arrive to the infamous decision of "which language do I pick first to learn", but I'm here to hopefully put your mind at ease.&#x20;

> <mark style="color:yellow;">**Note**</mark>: Due to the inner complexities of this post, I will attempt to explain everything in a high-level manner as possible to avoid unnecessary rambling to get my desired point across.

### Why different languages exist and why it's okay to feel overwhelmed at first

Although there are many different programming languages out there, they can be split up into two main categories, high or low-level programming languages. From there, they can be divided even further; into subsections, that will possess different capabilities for a multitude of different use cases. This is a bit out of scope for this blog post, so I won't go much into detail about that here because I want to highlight that you can essentially learn any programming language you want and it really won't matter going forward as LONG as you are ensuring that you are learning the necessary programming jargon, fundamentals, and concepts.&#x20;

This includes loops, logic, conditionals, variables, data types, objects, object-oriented vs procedural, and so much more. Point being, master these concepts and other programming languages that come your way into the future will make so much more sense.

This is why anything you learn AFTER your first language is so much easier to understand because you already have these underlying concepts but they may be featured in a syntactically different manner in one language to another, but the fundamentals still remain the same.&#x20;

<figure><img src="../.gitbook/assets/image (193).png" alt=""><figcaption><p>Visualizing languages</p></figcaption></figure>

## Low-level vs High Level Languages

Now this may get really controversial really quick, but I think this is by far the easiest way to digest this information and we will in fact be saving this argument for another day, but in this blog post, C will indeed be a low-level programming language. This is solely due to the fact that we get "low-level" access to the computer and memory access. Also, it is&#x20;

Essentially look at it this way, master one programming language and use it as a "stepping stone" to learn the in's and out's of programming basics and use that knowledge baseline going forward.&#x20;

Which way you should start (choosing to start with a low or high-level language) should align strictly with your curiosities as well as what you want to get into. Some professionals will argue that you should start high, others will say low. Personally, I recommend starting low and building up essentially. This is where I started and other security researchers and reverse engineers have told me the same thing. Although I've also heard the opposite as well... it really depends on you.&#x20;

### Low-level

Examples of these programming languages are C and Assembly. As I said earlier, these are among some of the lowest level of programming languages due to their somewhat archaic and "unusual" syntax that these languages follow. However, mastering these languages grants you access to hardware resources through the kernel, and allows you to get closer to memory and the Central Processing Unit (CPU). Programming in these languages will fundamentally make you a better programmer. In these languages, there is no safety net, there is no garbage collection, you will be required to abide by best practices and secure your code or it will be riddled with bugs if you can even get it to compile.

### High-level

High-level programming languages or intermediary-level languages would be like Python, Java, etc. This is because they are more abstracted and take away less stress from the programmer and will require less attention to how you are handling memory accesses (I/O operations, stack/heap accesses, etc.). This is because these languages will incorporate their own garbage collection measures and you can focus solely on the mission. However, this comes at a cost, because now you no longer have access to this type of control of memory due to the inherent nature of these languages.

## Abstractions

Let's take three different programming languages for this example to lay these concepts out better for you and allow you to visualize this.&#x20;

You can see directly that the higher we go in language levels, there are less lines of code to enable us to accomplish our goals, but like I said earlier, we lose a degree of power due to abstraction.

**Let's start low and go high and see how we can leverage different languages to accomplish a similar goal and the differences between!**

### Assembly (Low-Level)

Remember, Assembly (ASM) is the lowest of languages that is still human-readable and allows us to gain the most control of EXACTLY what our program is doing on our computer. Assembly DIRECTLY allows us to manipulate memory by means of various CPU registers and memory moving data in various registers via moving, "pushing", or "popping" data, and various other low-level calculations in order to implement your program's desired behavior.&#x20;

Now keep in mind, Assembly comes in different flavors because it is platform/architecture specific and you need to abide by the Instruction Set Architecture ISA for your CPU's architecture. For example, there is x86\_64 Assembly, ARM AARCH64 Assembly etc. One way you can get around this is by leveraging emulation software such as QEMU to emulate these different instruction sets on different processors, but that is out of scope for this blog post in particular.&#x20;

{% embed url="https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md#x86-32_bit" %}
Documentation for Linux syscalls
{% endembed %}

In Assembly, you will learn about assemblers, linkers, and compilers.

Below is our source code for our Assembly program.

`hello.s` **(Intel x86) utilizing the NASM Assembler:**

{% code lineNumbers="true" %}
```nasm
section        .text         
global         _start          
_start:
    mov edx, len 
    mov ecx, msg 
    mov ebx, 1
    mov eax, 4
    int 0x80
    mov eax, 1
    int 0x80section        .data             
    msg        db "Hello world!", 0xa
    len        equ $ -msg
```
{% endcode %}

**Create binary object or "blob" with the NASM Assembler and utilize the GNU Linker (LD) to link the blob to a binary:**

```
nasm hello.s -f elf32 && ld hello.o -o hello
```

We now have a binary that we can execute on our systems called, `hello`.

<figure><img src="../.gitbook/assets/image (194).png" alt=""><figcaption><p>Hello world!</p></figcaption></figure>

This was accomplished in <mark style="color:green;">12 lines of code</mark>.

Now let's do this in C, still a lower-level language, but much easier to work with than Assembly.

### C (Low-Intermediary-Level)&#x20;

In C, we will still have to worry about memory management and secure coding practices. However, the syntax is much easier to understand than Assembly's and we can accomplish our "hello world" program in <mark style="color:green;">half of the lines of code that we did in our Assembly program</mark>! Also, we will not need to worry about abiding to ISA's with C. But, we will need to worry about compiling for various architectures say if your friend's computer has an Intel-based CPU and you have an ARM CPU.

`hello.c`:

{% code lineNumbers="true" %}
```c
#include <stdio.h>

int main() {
    puts("Hello world!");

    return 1;
}
```
{% endcode %}

**Compile:**

```
gcc hello.c -o hello
```

This will give us a binary, named `hello`.

<figure><img src="../.gitbook/assets/image (196).png" alt=""><figcaption><p>Hello world!</p></figcaption></figure>

### Python (High-Level)

Now, let's utilize a high-level programming language!! What's amazing about Python is we can essentially <mark style="color:green;">accomplish the same mission in EXACTLY one line</mark>. However, we will lose significant low-level control of our program but we are able to accomplish much more quickly.

Python is a scripting language. This means that it utilizes an interpreter and we will not need to worry about compiling our code before we need to execute it. This is all done in the background meaning the programmer does not need to worry about anything else, besides writing code.

Directly using the Python3 interpreter, we can print out the string "Hello world!" that we've been doing above!

**Access the Python3 interpreter directly (or you can write a script and invoke it with `python3`):**

```
python3
```

<figure><img src="../.gitbook/assets/image (198).png" alt=""><figcaption><p>Hello world!</p></figcaption></figure>

`hello.py` (Script):

```python
print("Hello world!")
```

It's that simple!!

## Conclusion

I hope I could ease your mind in being able to understand programming. Especially when it comes to selecting your first language.
