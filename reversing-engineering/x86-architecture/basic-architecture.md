# Basic Architecture

A computer application is simply a table of machine instructions stored in memory to which the binary numbers which make up the program are unique only in the way the CPU deals with them.

The basic architecture is made up of a CPU, memory, and input/output (I/O) devices that are all connected by a system bus.

<figure><img src="../../.gitbook/assets/Screenshot 2023-02-27 at 12.31.06 PM.png" alt=""><figcaption></figcaption></figure>

Let's talk about the CPU and what it consists of:

1. <mark style="color:yellow;">Control Unit</mark>- Retrieves and decodes instructions from the CPU and then stores/receives them from memory
2. <mark style="color:yellow;">Execution Unit</mark>- Where the execution of fetching and receiving instructions occurs
3. <mark style="color:yellow;">Registers</mark>- Internal CPU memory locations used as a temporary data storage
4. <mark style="color:yellow;">Flags</mark>- Indicates events when execution occurs

<mark style="color:red;">Control Unit</mark> -> <mark style="color:green;">Execution Unit</mark> -> <mark style="color:blue;">Registers</mark> -> <mark style="color:blue;">Flags</mark>

## How do 32-bit CPU's Execute Code?

<mark style="color:yellow;">First, the 32-bit CPU will fetch a double word (4 bytes) from a specific address in memory where it is read and loaded into the CPU</mark>. At this point the CPU looks at the binary pattern of bits within the double word and begins executing the procedure that the fetched machine instruction directs it to do.

<mark style="color:yellow;">Upon completion of the instruction (execution), the CPU goes to memory and fetches the next machine instruction in sequence. The CPU has a register called EIP or Instruction Pointer that contains the address of the next instruction to be fetched from memory and then executed.</mark>&#x20;

* We can immediately see that if we controlled flow of EIP,&#x20;
