---
description: 06-02-2023
---

# GCC Conditional

## Conditionals

A conditional is a directive that <mark style="color:yellow;">INSTRUCTS the preprocessor to select whether or not to include a chunk of code in the final token stream passed to the compiler</mark>.&#x20;

#### Use Cases

Three general reasons to use a conditional:

1. A program may need to use different code depending on the machine or OS it is run on. <mark style="color:yellow;">Sometimes, data types or constants may not exist on the other system</mark>. <mark style="color:yellow;">The compiler would simply reject the program</mark>. With a preprocessing conditional, the bad code can be removed from the program when it is not valid.
2. <mark style="color:yellow;">You may have a situation where you want to be able to compile the same source file into two different programs</mark>. One version might make frequent time-consuming consistency checks on its intermediate data, or print the values of those data for debugging and the other not.&#x20;
3. A <mark style="color:yellow;">conditional whose condition is always false is one way to exclude code from the program but keep it as a sort of comment for future reference</mark>.

## Conditional Syntax

<mark style="color:yellow;">So, what does a conditional look like in C</mark>?

`#if`,`#ifdef`, or `#ifndef`

### [https://gcc.gnu.org/onlinedocs/gcc-3.0.1/cpp\_4.html](https://gcc.gnu.org/onlinedocs/gcc-3.0.1/cpp\_4.html)
