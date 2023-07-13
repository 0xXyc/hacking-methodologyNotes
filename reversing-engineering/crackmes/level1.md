---
description: 07/09/2023
---

# "level1"

**Platform:** Unix/linux etc.

**Difficulty:** 1.0

**Quality:** 4.4

**Arch:** x86-64

**Description:**

_Very easy and beginner friendly crackme. Reverse Engineer and get the password._

The challenge can be found here:

{% embed url="https://crackmes.one/crackme/646627a933c5d439389131d9" %}

## Enumeration

Let's see what the file does:

```
./level1 
Welcome to Easy Crack MeWhat is the Secret ?1
Better luck next time. :(
```

Checking File info:

{% code overflow="wrap" %}
```
file level1
/home/user/Downloads/level1: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=89f40555f67806b4082e5785d91836bb6baaaa9d, for GNU/Linux 3.2.0, not stripped
```
{% endcode %}

Let's throw this in Ghidra and run strings on the binary:

```
strings ~/Downloads/level1 
/lib64/ld-linux-x86-64.so.2
__libc_start_main
__cxa_finalize
printf
__isoc99_scanf
libc.so.6
GLIBC_2.7
GLIBC_2.2.5
GLIBC_2.34
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
PTE1
u+UH
<supH
<uuhH
<duYH
<ouJH
<0u;H
<xu,H
Welcome to Easy Crack Me
What is the Secret ?
%64s
You are correct :)
Better luck next time. :(
;*3$"
GCC: (Debian 12.2.0-14) 12.2.0
Scrt1.o
__abi_tag
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.0
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
ch_first.c
__FRAME_END__
_DYNAMIC
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_start_main@GLIBC_2.34
_ITM_deregisterTMCloneTable
_edata
_fini
printf@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
_end
checkPass
__bss_start
main
__isoc99_scanf@GLIBC_2.7
__TMC_END__
_ITM_registerTMCloneTable
__cxa_finalize@GLIBC_2.2.5
_init
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
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

Nothing interesting was found here.

### Elf Header

```
readelf -h level1 
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1060
  Start of program headers:          64 (bytes into file)
  Start of section headers:          14064 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

* Entry point: `0x1060`

### `strace`/`ltrace`

`strace`: trace system calls and signals // Intercepts and records syscalls which are called by a process and the signals received by the process.

{% embed url="https://man7.org/linux/man-pages/man1/strace.1.html" %}

Run program with `strace` until it exits:

```
strace ./level1 
execve("./level1", ["./level1"], 0x7ffe7ed140e0 /* 44 vars */) = 0
brk(NULL)                               = 0x556a50da0000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffe3884d600) = -1 EINVAL (Invalid argument)
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f81d99b8000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=58835, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 58835, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f81d99a9000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0 \0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0"..., 48, 848) = 48
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0i8\235HZ\227\223\333\350s\360\352,\223\340."..., 68, 896) = 68
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=2216304, ...}, AT_EMPTY_PATH) = 0
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 2260560, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f81d9600000
mmap(0x7f81d9628000, 1658880, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x28000) = 0x7f81d9628000
mmap(0x7f81d97bd000, 360448, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1bd000) = 0x7f81d97bd000
mmap(0x7f81d9815000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x214000) = 0x7f81d9815000
mmap(0x7f81d981b000, 52816, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f81d981b000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f81d99a6000
arch_prctl(ARCH_SET_FS, 0x7f81d99a6740) = 0
set_tid_address(0x7f81d99a6a10)         = 63810
set_robust_list(0x7f81d99a6a20, 24)     = 0
rseq(0x7f81d99a70e0, 0x20, 0, 0x53053053) = 0
mprotect(0x7f81d9815000, 16384, PROT_READ) = 0
mprotect(0x556a4fc3b000, 4096, PROT_READ) = 0
mprotect(0x7f81d99f2000, 8192, PROT_READ) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
munmap(0x7f81d99a9000, 58835)           = 0
newfstatat(1, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}, AT_EMPTY_PATH) = 0
getrandom("\x9a\x4a\x5f\x20\xca\x91\xde\x6e", 8, GRND_NONBLOCK) = 8
brk(NULL)                               = 0x556a50da0000
brk(0x556a50dc1000)                     = 0x556a50dc1000
newfstatat(0, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}, AT_EMPTY_PATH) = 0
write(1, "Welcome to Easy Crack MeWhat is "..., 44Welcome to Easy Crack MeWhat is the Secret ?) = 44
read(0, 
```

I didn't see anything of use here, but it is still a good idea to get a lay of the land.&#x20;

syscalls:

1. [`execve()`](https://man7.org/linux/man-pages/man2/execve.2.html): Executes the program referred to by pathname. This causes the program that is currently being run by the calling process to be replaced with a new program. It will begin to initialize a new stack, heap, and data.
2. [`mmap()`](https://man7.org/linux/man-pages/man2/mmap.2.html): Found within the C library, it will map or unmap files/devices into memory.
3. There are more but I wasn't familiar with these.

`ltrace`: A library call tracer // Intercepts and records the dynamic lib calls which are called by the executed process.

{% embed url="https://man7.org/linux/man-pages/man1/ltrace.1.html" %}

Run program with `ltrace` until it exits:

```
ltrace ./level1
printf("Welcome to Easy Crack Me")                                                  = 24
printf("What is the Secret ?")                                                      = 20
__isoc99_scanf(0x562990188032, 0x7ffe5f9804e0, 0, 0x7efceb815a00Welcome to Easy Crack MeWhat is the Secret ?1337
)                   = 1
printf("Better luck next time. :(")                                                 = 25
Better luck next time. :(+++ exited (status 0) +++
```

Nothing to see here either. Just simply using `printf()` to read strings and `scanf()` to take our input.

## Debugging

`gdb`: `gdb -d 1 level1`

View disassembly of the `main()` function:

<figure><img src="../../.gitbook/assets/image (5) (13).png" alt=""><figcaption></figcaption></figure>

#### Looking for interesting functions:

<mark style="color:yellow;">`checkPass`</mark>

We have identified a potentially valuable function for us. Now, we must understand what it is doing.

We see that <mark style="color:yellow;">`__isoc99_scanf`</mark>  is above it. This means that `checkPass` is called immediately after we enter our password that the program is asking us for.

Time to dig deeper into `checkPass`: `disass checkPass`

<figure><img src="../../.gitbook/assets/image (4) (8).png" alt=""><figcaption></figcaption></figure>

The instruction <mark style="color:yellow;">**mov QWORD PTR \[rbp-0x8],rdi**</mark> will move the value in the RDI register to the base pointer (RBP).

<mark style="color:yellow;">**mov rax,QWORD PTR \[rbp-0x8]**</mark> is responsible for moving the content of RBP into RAX.

<mark style="color:yellow;">**movzx eax,BYTE PTR \[rax]**</mark> is responsible for moving the value from the register RAX 64 bits to EAX 32 bits -- This might sound weird, but it is compilation optimization.

The next couple instructions, you will see byte values being moves and compared over and over again. This is important because this is our password!

We can also see this in Ghidra:

<figure><img src="../../.gitbook/assets/image (3) (4).png" alt=""><figcaption></figcaption></figure>

Let's string the bytes together:&#x20;

1. cmp al,0x73
2. cmp al,0x75
3. cmp al,0x64
4. cmp al,0x6f
5. cmp al,0x30
6. cmp al,0x78
7. cmp al,0x31
8. cmp al,0x38

Let's put this into proper byte order and covert the hex to string using `xxd`:

```
echo "x73x75x64x6fx30x78x31x38" | xxd -r -p
sudo0x18
```

Is this our password?

```
 ./level1
Welcome to Easy Crack MeWhat is the Secret ?sudo0x18
You are correct :)
```

Congrats!
