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

* <mark style="color:yellow;">Well, if you try to write or access outside of an array, you will get a segfault</mark>

<mark style="color:yellow;">All of our global memory is mixed up and we will need to translate addresses. We will need to also know what parts of our array belong with our Word and Excel process. Otherwise, we will get the wrong data from these processes</mark>.

<mark style="color:red;">**Therefore, what happens if we access outside of that memory?**</mark>

<mark style="color:red;">**SEGFAULT!**</mark>

<mark style="color:yellow;">These errors can occur when a program attempts to access memory outside of the pre-allocated space/memory for the program. This will always result in a segfault</mark>.

Also, if a program attempts to use a pointer that is pointing to `NULL`, it will segfault as well since 0 is an invalid region of memory and address 0 does not exist.&#x20;

## So how does it work? (Technical)

When an application is in use, data from that program is stored in a physical address using RAM. A <mark style="color:yellow;">memory management unit</mark> (<mark style="color:yellow;">MMU</mark>) will then map the address to RAM and automatically translate addresses. The MMU can for example, map a logical address space to a corresponding physical address.&#x20;

If/when the RAM space is needed for something more urgent, data can be swapped out of RAM and into virtual memory. The computer's memory manager is in charge of keeping track of the shifts between physical and virtual memory.&#x20;

If that data is ever needed again, the computer's MMU will use a <mark style="color:yellow;">context switch</mark> to resume execution. A context switch is the process of storing the state of the process or thread so that it can be restored and resumed execution at a later point once it is needed again.&#x20;

A context switch allows multiple processes to be able to share a single CPU.

While copying the virtual memory into physical memory, the OS divides memory with a fixed number of addresses into <mark style="color:yellow;">pagefiles</mark> or <mark style="color:yellow;">swap files</mark>. Each page is then stored on disk, and when the page is needed, <mark style="color:yellow;">the OS copies it from the disk to main memory and translates the virtual addresses to physical ones using the context switch as mentioned before</mark>!

However, this is a lengthy and computationally slow process to swapping from virutal to physical is inefficient. This means that computers with more RAM will always generally be faster.&#x20;

<mark style="color:yellow;">NOTE</mark>: <mark style="color:yellow;">Virtual memory is still volatile even if it is temporarily stored on disk during this process</mark>. So if you lost power or crashed, you would lose this data. However, it could be possible to recover from forensics unless it was overwritten.&#x20;

