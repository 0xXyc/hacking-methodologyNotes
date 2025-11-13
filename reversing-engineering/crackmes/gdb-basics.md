---
description: 07/09/2023
---

# "GDB Basics"

**Platform Unix/linux etc.**

**Difficulty: 1.3**

**Quality: 4.9**

**Arch: x86**

**Description**

{% embed url="https://www.sourceware.org/gdb/documentation/" %}

The challenge can be found here:

{% embed url="https://crackmes.one/crackme/645d3d4e33c5d43938913079" %}

## Enumeration

Let's simply just run the program:

```
./a.out 
Enter a number : 1337
I cannot provide you a flag !
```

### File Info

{% code overflow="wrap" %}
```
file a.out 
a.out: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=07adab7654c353add9669995f1587e01439c24db, for GNU/Linux 3.2.0, not stripped
```
{% endcode %}

* non-stripped
* PIE executable

### Strings

```
strings a.out
/lib64/ld-linux-x86-64.so.2
__isoc99_scanf
puts
printf
__cxa_finalize
__libc_start_main
libc.so.6
GLIBC_2.7
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u/UH
[]A\A]A^A_
Enter a number : 
There is the flag : I_LOVE_YOU
I cannot provide you a flag !
;*3$"
GCC: (Debian 10.2.1-6) 10.2.1 20210110
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.0
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
main.cpp
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
puts@GLIBC_2.2.5
_edata
printf@GLIBC_2.2.5
__libc_start_main@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
__isoc99_scanf@GLIBC_2.7
__TMC_END__
_ITM_registerTMCloneTable
__cxa_finalize@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.got.plt
.data
.bss
.comment
```

I thought it was pretty funny that we were given the flag right away, however, what is the number that will initiate this? Let's find out.

## Reversing

### Ghidra

Let's check out our `main()` function in the disassembly:

<figure><img src="../../.gitbook/assets/image (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

We can see that there is a for loop that generates the accepted number to print the flag.&#x20;

It consists of a simple test using `printf()` and `scanf()`.

If the inputted number, which I am assuming is what Ghidra is interpretting as local\_14, then it will print the flag. However, if the number is incorrect, it will not provide us the flag. This means that local\_c is the number that we are inputting (taken by `scanf()`).

It is important to note that the flag is not important in this challenge, the number generated is.&#x20;

With that knowledge, I decided to make this process easier on me and change the variable names.

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

We need to calculate our input-number to get the correct number for the flag.

Let's step through this in `gdb`.

#### `gdb a.out`

<figure><img src="../../.gitbook/assets/image (5) (7).png" alt=""><figcaption></figcaption></figure>

With some research, we were able to find out the the for loop begins with the `jg` intstruction or `0x000000000000116f.`

Remember, this is important because it is where the program is computing the number we need for the flag.

However, we need to look shortly after it where we input our number using `scanf()`.&#x20;

Consider the logic of the program. It wants us to input the correct number and it will spit us out the flag upon doing so.

<figure><img src="../../.gitbook/assets/image (4) (8).png" alt=""><figcaption></figcaption></figure>

Let's place a breakpoint at the `cmp` instruction. Here is where the program is comparing our inputted number stored in the `eax` register from scanf() to the correct value stored within the `rbp` register.

Upon breaking at the compare, this allows us to pause execution, view registers, and the values contained within them.

Let's view the decimal value of the `rbp` register:

<figure><img src="../../.gitbook/assets/image (14) (6).png" alt=""><figcaption></figcaption></figure>

Congrats!
