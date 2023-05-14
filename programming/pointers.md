---
description: 5-14-2023
---

# Pointers

## What is a pointer?

A pointer is just an address.

<mark style="color:yellow;">A pointer is simply just a value that happens to be an address</mark>.&#x20;

<mark style="color:yellow;">By setting the value of a variable = to the address of another address, that variable now POINTS to the other</mark>.

## Memory

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

### Well as we described earlier, we made `0x4` get stored in address `0x1000`. First, what does this look like in C?

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

### Now what does the pointer look like in C?

```
int * pX = &x;
```

int:

Data type int. Four bytes wide.

\*:

When an asterisk is placed next to a data type, it modifies the type and is converted into a pointer. Meaning our variable points to a 4 byte value.

pX:

Integer pointer named pX (named pX because it is a good habit to note that it is a pointer).

\=:

Is set to.

\&x;:

The ampersand (&) simply is the address of the variable x.

#### English Explanation:

> "Integer pointer named pX is set to the address of x".

## Great, but what does this to for us?

Now we can use the pointer to access x by reference instead of value.

So instead of using:

```
int * pX = &x;
```

We can use this:

```
int y = *px;
```

#### English Explanation:

> "Integer named y is set to the thing pointed to by pX".

This means that we can pass `x` around as reference instead of value.

## Why does this matter? Why does anyone use pointers?

The answer is because we have to.

We must be able to break down functions by the actions that they perform.

Using pointers allows us to keep our code clean, avoids copying (saves space), and maintains readability and understandability.

#### Another Reason: Static VS Dynamic Memory Allocation

Static allocation is typically a variable that is placed on the stack.&#x20;

Dynamic allocation that comes from the heap such as malloc or sbrk.&#x20;

The difference is that static allocations has a FIXED size at compile time. Dynamic allocation can be changed at runtime at an as needed basis.
