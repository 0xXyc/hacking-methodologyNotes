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

### Let's use arrays

Note: They will all increment by one.&#x20;

<mark style="color:yellow;">Scenario</mark>:

Consider this: you, the user, opens Microsoft Word and it loads what it needs (<mark style="color:yellow;">10MB</mark>).

<mark style="color:yellow;">Result</mark>:

Array\[0]->Array\[10]

<mark style="color:yellow;">Scenario</mark>:

You then load Excel and it loads all the memory it needs (<mark style="color:yellow;">5MB</mark>).

<mark style="color:yellow;">Result</mark>:

Array\[11]->Array\[16]

<mark style="color:yellow;">Scenario</mark>:&#x20;

While you are working with Word, you decide to open up a <mark style="color:yellow;">100MB</mark> document.

<mark style="color:yellow;">Result</mark>:

Array\[17]->Array\[117]

How cool is that????

This is done so that way you do not ever get the wrong data in an application. Everything stays where it should be.&#x20;

## Segmentation Faults (Segfaults)

Why is this being mentioned here?&#x20;

* Well, if you try to write or access outside of an array, you will get a segfault

<mark style="color:yellow;">These errors can occur when a program attempts to access memory outside of the pre-allocated space/memory for the program. This will always result in a segfault</mark>.

Also, if a program attempts to use a pointer that is pointing to `NULL`, it will segfault as well since 0 is an invalid region of memory and address 0 does not exist.&#x20;
