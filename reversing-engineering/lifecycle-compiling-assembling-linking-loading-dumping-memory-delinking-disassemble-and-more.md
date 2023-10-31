---
description: 10/27/2023
cover: https://media1.giphy.com/media/zLi05iFElqOZimqAAX/giphy.gif
coverY: 22
---

# Lifecycle: Compiling, Assembling, Linking, Loading, Dumping Memory, Delinking, Disassemble, and more

## A Binary's Lifecycle, the Process of Forward & Reverse Engineering

### Forward Engineering at a High Level

1. Natural Language
2. Pseudocode
3. Source Code
4. Assembly
5. Object
6. Executable
7. Process

### Reverse Engineering at a High Level

1. Process
2. Executable
3. Object
4. Assembly
5. Source Code
6. Pseudocode
7. Natural Language

> _**"Always draft out pseudocode for forward and reverse engineering to better "paint the picture".**_

## <mark style="color:yellow;">Compiling</mark> C to ASM

Take C source code and convert to ASM code (source).

{% code overflow="wrap" %}
```
gcc -S -masm=intel -fno-stack-protector -fno-pie -fcf-protections=none -fno-asynchronous-unwind-tables <binary.c> -o <assembly-code.s>
```
{% endcode %}

Upon running file on the `.s` file, we will see that the file signature is of "assembler source".

## <mark style="color:yellow;">Assembly</mark> to Object

Convert ASM source code to an object.

```
gcc -c <asm_code.s> -o <object_file.o>
```

We can run `hexdump` on this file w/ `-C` to obtain a hexdump of the object file in Little-Endian format.

We can also obtain the disassembly of the object file with `objdump -d <object_file.o>`.

(e.g.) `objdump -d main -M intel object_file.o`

This will allow us to view the disassembly of `main()` in the Intel-based format.

## <mark style="color:yellow;">Linking</mark> an Object to an Executable

Link an object file to an executable.

```
gcc <object_file.o> -nostartfiles -o <executable.exe>
```

Running `file` will show our executable is compiled as an ELF binary.

View disassembly:

```
objdump -d main -M intel <executable.exe>
```

We can see `.plt` section in the disassembly since the `libc` library has now been linked to the executable!

## <mark style="color:yellow;">Loading</mark> an Executable to Process

We can automate behavior in `gdb` with a `.txt` file and create a process memory dump via `.core` files.

`file` will indicate the file signature as a `corefile`.

## <mark style="color:yellow;">Dumping</mark> Memory & <mark style="color:yellow;">Process</mark>2Executable

We can utilize the following binary, `core2ELF` and take a process, dump it, and create a "rebuilt" binary.

Yes, we can actually utilize it and run it normally.

{% embed url="https://github.com/enbarberis/core2ELF64" %}

## <mark style="color:yellow;">Delinking</mark> an Executable to an Object

For the delinking process we will be utilizing the Witchcraft Compiler Collection (WCC).

{% embed url="https://github.com/endrazine/wcc" %}

```
wcc -c <executable.exe> -o <unlinked.o>
```

## <mark style="color:yellow;">Disassembling</mark> an Object to ASM

Here, we can utilize IDA, Ghidra, Objdump, radare2, gdb, and objconv.

(e.g.) `objdump -d -M intel <unlinked.o> > <disassembly.s>`

`cat <disassembly.s>`

## <mark style="color:yellow;">Decompiling</mark> ASM to C

Converting machine code to decompiled C-like pseudo code.

Ghidra Headless comes in handy here -- analyzeHeadless.

Essentially, this performs: **`binary -> decomp.c`**

#### Note:

At this point, it would be a fantastic idea to put the decompiled function in ChatGPT and ask what it is doing:

> _**"What does the following C program do?"**_
>
> `void foo()`
>
> `{`
>
> `bar;`
>
> `}`

