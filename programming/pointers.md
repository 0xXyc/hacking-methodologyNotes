---
description: 5-14-2023
---

# Pointers

In order to understand pointers, you must first understand memory.&#x20;

<mark style="color:yellow;">Memory will just have an address and a value</mark>.&#x20;

The <mark style="color:yellow;">address</mark> is the <mark style="color:yellow;">LOCATION of the MEMORY -- where the memory lives</mark>.

The <mark style="color:yellow;">value</mark> of that <mark style="color:yellow;">MEMORY is the DATA stored at that location -- what memory lives here</mark>.

Imagine the value `0x4` being stored in address `0x1000`.

You can make this happen in C with the following syntax:

```
int x = 4;
```

The value, 4 gets allocated to the stack, and now it lives there.

Now what happens if you were to put the value `0x1000` at address `0x1004`?

This means that you just created a pointer.&#x20;

What does this look like in C code?

```
int x = 4;
```

#### Explanation

int:

(integer) type of variable which is 4 bytes wide.

x:

is the variable.

\=:

Verbalizes the variable is set to.

4:

Integer who's name is x, is set to the value 4.

## What is a pointer?

<mark style="color:yellow;">A pointer is simply just a value that happens to be an address</mark>.&#x20;

<mark style="color:yellow;">By setting the value of a variable = to the address of another address, that variable now POINTS to the other</mark>.
