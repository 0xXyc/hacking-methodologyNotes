---
description: Registaaaas
---

# General-Purpose Registers

<mark style="color:yellow;">The General-Purpose Registers are used to temporarily store data as it is processed on the processor</mark>.

The registers have evolved dramatically over time and will continue to do so.

Each new version of general-purpose registers is created to be backward compatible with previous processors. This means that code utilizing 8-bit registers on the 8080 chips will still function on today's 64-bit chipset.

<mark style="color:yellow;">General-Purpose registers can be used to hold any type of data</mark> to which some have acquired specific use which are used in programs. <mark style="color:yellow;">Let's review the 8 General-Purpose registers in an IA-32 architecture</mark>.

EAX: The Main Register used in arithmetic calculations. A.K.A. an accumulator, because it holds results of arithmetic operations and function return values.

EBX: The Base Register. Pointer to data in the DS segment. Used to store the base address of the program.

ECX: The Counter Register is often used to hold a value representing the number of times a process is to be repeated. Used for loop and string operations!

EDX: A General Purpose register. Additionally used for I/O ops. In addition to extend to 64-bits.

ESI: Source Index Register. Pointer to data in the segment pointed to by the DS register. USed as an offset address in string and array operations. It holds the address from where to read data.

EDI: Destination Index Register. Pointer to data (or destination) in the segment pointed to by the ES register. Used as an offset address in string and array operations. It holds the implied write address of all string operations.

EBP: Base Pointer. Pointer to data on the stack (in the SS segment). It points to the bottom of the current stack frame. It is used to reference local variables.
