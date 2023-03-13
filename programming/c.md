---
description: My favorite compiled language
---

# C

## Introduction

The C programming language is often called a middle-level cmoputer language as it combines the elements of high and low-level languages with the functionalism of assembly language. C programming allows the manipulation of bits, bytes, and addresses -- <mark style="color:yellow;">giving the programmer more control over exactly how the program will behave.</mark>

## Flexibility

C is not tied to any particular hardware or system. This makes it easier for a user to write programs that will run without many (or any) changes on practically all machines.

## OPSEC Considerations

* This will cover OPSEC considerations an operator/developer should follow when compiling C code

## GNU Project C and C++ Compiler (GCC)

### Arguments

* Stripping: -<mark style="color:yellow;">s</mark> -- Always enable
* \-<mark style="color:yellow;">fstack-protector</mark> -- Adds a canary value in the stack to catch buffer overflows (<mark style="color:red;">sometimes</mark>)&#x20;
* \-<mark style="color:yellow;">g</mark> -- <mark style="color:yellow;"></mark> <mark style="color:red;">Never enable when rolling out and sending around. This is the debugging arg.</mark>

## Cheat sheets

{% embed url="https://lzone.de/cheat-sheet/GCC" %}
Cheat sheet
{% endembed %}

{% embed url="https://man7.org/linux/man-pages/man1/gcc.1.html" %}
Official man page
{% endembed %}
