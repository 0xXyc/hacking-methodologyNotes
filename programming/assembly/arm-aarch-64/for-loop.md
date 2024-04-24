---
description: 04/24/2024
---

# for loop

## What is a for loop?

For loops are essentially a section of code that can be broken down in <mark style="color:yellow;">two parts</mark>, establish a <mark style="color:yellow;">counter</mark> to <mark style="color:yellow;">define the iteration</mark> and then <mark style="color:yellow;">define the body that will execute once per iteration</mark> of the defined counter.&#x20;

A friend and fellow security researcher recently told me that it is a great idea to "picture" C when writing ASM and "picture" ASM when writing C. This is a useful skill to have when dealing with ASM.

{% hint style="info" %}
Note: For some fun, I put a shell in here for you so you can play around with it since we're going to be getting into some shellcode coming up!! I also thought this would make it a bit more fun!!
{% endhint %}

Example in C -> `forloop.c`:

```c
for (int i = 0; i < 10; i++) {
    // do something
    printf("Here's your loop!\n");
}
```

### However, there's a few ways of doing this...

Instead of <mark style="color:yellow;">incrementing</mark> your counter, you can <mark style="color:yellow;">start at a higher counter</mark> and then <mark style="color:yellow;">decrement from it using</mark> `i--`.&#x20;

**See below:**

```c
for (int i = 10; i > 0; i--){
    // do something
    printf("Here's your for loop!\n");
}
```

### What would this look like in ASM?

For <mark style="color:green;">incrementing</mark> the counter, your ASM will look like this:

`forloop.s`:

```nasm
.global main

.global _start
.section .text

main:
    mov x15, #10    // Initialize counter (i = 10)

loop:
    cmp x15, #0    // Compare counter to 0
    beq exit    // Branch to exit program if == 0

    # Write syscall
    mov x8, #64    // Syscall ID for write
    mov x0, #1    // First argument
    ldr x1, =msg    // Load address into x1
    mov x2, len    // Size of string
    svc 0        // Perform syscall -- Supervisor call

    sub x15, x15, #1    // Decrement counter (i--)
    b loop            // Branch to loop

exit:

    // Welcome shell
    mov x8, #64
        mov x0, #1
        ldr x1, =welcome_shell
        mov x2, len2
        svc 0

    // Shell
    ldr x0, =shell        // Load contents of shell variable into x0 register
    bl system        // Branch to system and link return

    // Exit syscall
    mov x8, #93        // Syscall ID for exit
    mov x0, #0        // Mov 0 into x0 register
    svc 0            // Supervisor call --> perform syscall

.section .data
    msg: .ascii "Fuck your for loop!\n"
    len = . - msg
    welcome_shell: .ascii "Here's a shell xD\n"
    len2 = . - welcome_shell
    shell: .ascii "/bin/sh"

```

For <mark style="color:red;">decrementing</mark> the counter, your ASM will look like this:

`forloop.s`:

```nasm
.global main

.global _start
.section .text

main:
	mov x15, #10	// Initialize counter (i = 10)

loop:
	cmp x15, #0	// Compare counter to 10 (i < 10)
	beq exit	// Branch to exit program if (i == 0)

	# Write syscall
	mov x8, #64	// Syscall ID for write
	mov x0, #1	// First argument
	ldr x1, =msg	// Load address into x1
	mov x2, len	// Size of string
	svc 0		// Perform syscall -- Supervisor call

	sub x15, x15, #1	// Decrement counter (i--)
	b loop			// Branch to loop

exit:

	// Welcome shell
	mov x8, #64
        mov x0, #1
        ldr x1, =welcome_shell
        mov x2, len2
        svc 0

	// Shell
	ldr x0, =shell		// Load contents of shell variable into x0 register
	bl system		// Branch to system and link return

	// Exit syscall
	mov x8, #93		// Syscall ID for exit
	mov x0, #0		// Mov 0 into x0 register
	svc 0			// Supervisor call --> perform syscall

.section .data
	msg: .ascii "Here's your for loop!\n"
	len = . - msg
	welcome_shell: .ascii "Here's a shell xD\n"
	len2 = . - welcome_shell
	shell: .ascii "/bin/sh"
```
