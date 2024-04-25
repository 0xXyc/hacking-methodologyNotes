---
description: 04/24/2024
---

# while loop

## What is a `while` loop?

This is a type of loop that is used to repeat a specific block of code an unknown number of times until a specific condition is met. In other words, it will continue to loop through the section of code as long as the predetermined condition remains true.

A _counter_ is not required in a `while` loop. The number of iterations (counter) is not known and unspecified in a `while` loop.

Below is a `while` loop in c. This will be specifying an incrementing value.\
`while.c`:

```c
#include <stdlib.h>
#include <stdio.h>

int main() {

    int i = 0;
    while (i < 10){
        printf("Hi\n");
        i++;
    }

    return 0;

}
```

Next, we will be doing this in ASM:

{% hint style="info" %}
Note: you can still set this up to either increment or decrement, it's up to you. I'll challenge you to set up a variation that decrements :smirk:. `If you want to see the variation instead, view the last code snippet at the bottom of this blog.`&#x20;
{% endhint %}

### Incrementing

`while.s`:

```armasm
.global _start

.section .data
	msg: .ascii "While loop lol\n"
	len = . - msg
	end_msg: .ascii "You hit the end of the while loop!\n"
	len2 = . - end_msg
.section .text
_start:

mov x15, #0         // Initialize counter (i = 10)

while_loop:

    cmp x15, #10      // Compare counter with 0
    beq while_end  // If counter is less or equal to 0, end loop

    // "do something" goes here
    mov x8, #64
    mov x0, #1
    ldr x1, =msg
    mov x2, len
    svc 0

    add x15, x15, #1  // Increment counter (i--)
    b while_loop    // Jump back to start of loop

while_end:
    // Continue with rest of program...
    mov x8, #64
    mov x0, #1
    ldr x1, =end_msg
    mov x2, len2 
    svc 0

    // Exit program
    mov x8, #93
    mov x0, #0
    svc 0

```

**Assembling and Linking:**

```
aarch64-linux-gnu-as while.s -o while.o && aarch64-linux-gnu-ld while.o -o while 
```

### Decrementing (Spoiler)

`while.s`:

```armasm
.global _start

.section .data
	msg: .ascii "While loop lol\n"
	len = . - msg
	end_msg: .ascii "You hit the end of the while loop!\n"
	len2 = . - end_msg
.section .text
_start:

mov x15, #10         // Initialize counter (i = 10)

while_loop:

    cmp x15, #0      // Compare counter with 0
    b.le while_end  // If counter is less or equal to 0, end loop

    // "do something" goes here
    mov x8, #64
    mov x0, #1
    ldr x1, =msg
    mov x2, len
    svc 0

    sub x15, x15, #1  // Increment counter (i--)
    b while_loop    // Jump back to start of loop

while_end:
    // Continue with rest of program...
    mov x8, #64
    mov x0, #1
    ldr x1, =end_msg
    mov x2, len2 
    svc 0

    // Exit program
    mov x8, #93
    mov x0, #0
    svc 0

```

**Assembling and Linking:**

```
aarch64-linux-gnu-as while.s -o while.o && aarch64-linux-gnu-ld while.o -o while 
```
