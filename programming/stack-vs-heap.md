---
description: 05-13-2023
---

# Stack VS Heap

## The Stack

It can be best described as a stack of books. Nothing can be accessed or touched below the top of the stack. Only things can be taken off of the top or added to the top.&#x20;

### Call Stack

In applications we have the call stack and it has a couple of responsibilities.&#x20;

Keeping track of the method that controls should be returned to after the current method has finished executing.&#x20;

The second responsibility is keeping track of local variables in your method.

### Visual Depiction

<figure><img src="../.gitbook/assets/image (23).png" alt="" width="375"><figcaption></figcaption></figure>

#### Machine Code

This is where your application is converted into instructions that your computer can understand.

## The Heap

The heap is much different. Anything can be accessed or removed at any time. However, adding items to the heap has a much higher overhead than it does to adding it to the stack. You can store in any order and access from anywhere.&#x20;

<mark style="color:yellow;">Generally you have data on the heap when it is going to outlive data on the stack</mark>.

Values for reference types are always stored in the heap.&#x20;

Usually, your pointer will stay on the call stack and the value for your reference type is stored on the heap. But this is not always the case.&#x20;

If it is a GLOBAL variable, it will always be stored on the heap.

Reference types are always on the heap.&#x20;

Value types are on the stack or heap depending on how you declare them.

Static variables will always be stored on the heap because your program needs to access them.&#x20;

The second that these are not used anymore, garbage collection comes for them.&#x20;

## Garbage Collection

What happens to a blob of memory once that method is finished executing?

This is as simple as things being cleared up once they are not being used anymore within the heap.&#x20;

## Threading/Asynchronous Methods

When threading is occuring, each thread will be assigned its own individual call stack. This is so each thread can finish independently because it is up to them to complete their instructions themselves (independently). All of your results will be stored and retrieved from the heap from the thread's call stack.
