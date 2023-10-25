---
description: 10/25/2023
---

# Debugging

## Introduction

We can utilize a `.txt` file for automating the sending of commands to gdb for us.

(e.g.) **We can dump the process memory to a specified file:**

`gdb-commands.txt:`

```
# load process
start

# set bitmask for which memory segments to dump
!sudo echo 0x7 > /proc/$(pgrep aslr1)/coredump_filter

# dump process memory to file process.core
gcore process.core

# no confirm, so we can quit without interaction
set confirm off
quit
```

**NOTE:** <mark style="color:yellow;">Be sure to change the following pgrep binary with your specified binary you are targeting</mark>.

* In this case, our target binary is `aslr1`

**Run with gdb:**

```
gdb -q --command=gdb-commands.txt ./aslr1
```

We then have a memory dump of that process under the file name, `process.core` in which we can use <mark style="color:yellow;">to further analyze our binary in memory</mark>.
