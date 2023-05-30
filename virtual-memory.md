---
description: 05-30-2023
---

# Virtual Memory

## Introduction

This is a memory management technique that was especially popular when RAM was very expensive.&#x20;

<mark style="color:yellow;">Secondary memory</mark> can be used as if it were part of the main memory.

It <mark style="color:yellow;">utilizes both hardware and software to enable a computer to compensate</mark> for low physical memory.

It works by <mark style="color:yellow;">transferring data from RAM to disk storage</mark>.

<mark style="color:yellow;">Mapping chunks of memory to disk files enables a computer to treat secondary memory as if it were main memory</mark>.&#x20;

### Practical Example

Sometimes computers would only have 4-8 GB of RAM, and some would argue that is insignificant for a computer nowadays due to the size of programs. This is where virtual memory shines. If the computer was to not have enough RAM to handle the load of the programs, it would initiate and begin sending data to the disks to handle the operations.&#x20;

#### Let's use arrays



## Segmentation Faults (Segfaults)

Why is this being mentioned here?&#x20;

* Well, if you try to write or access outside of an array, you will get a segfault

<mark style="color:yellow;">These errors can occur when a program attempts to access memory outside of the pre-allocated space/memory for the program. This will always result in a segfault</mark>.

Also, if a program attempts to use a pointer that is pointing to `NULL`, it will segfault as well since 0 is an invalid region of memory and address 0 does not exist.&#x20;
