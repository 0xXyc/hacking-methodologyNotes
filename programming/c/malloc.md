---
description: 05-18-2023
---

# malloc

## Introduction

<mark style="color:yellow;">Memory allocation or malloc</mark>, is a built-in function in C.

This function is used to assign a specified amount of memory for an array to be created.&#x20;

* Remember, arrays are used to store multiple values in a single variable

It also returns a pointer to the space allocated in memory using this function.

## Why does malloc exist?

There are many times where <mark style="color:yellow;">we only want an array to have a specific amount of space at run time</mark>.&#x20;

To do this, <mark style="color:yellow;">we can create an array that occupies that amount of space dynamically, using malloc</mark>.&#x20;

## Syntax

Malloc returns and it requires an input.&#x20;

What does it look like?

```
(castingDataType*)malloc(size_t size);
```

* `castingDataType` <mark style="color:yellow;">is the data type of the pointer that is being created</mark>
* `size_t size` <mark style="color:yellow;">is the size in bytes that is being allocated</mark>

> <mark style="color:red;">What happens if it the memory specified is not available</mark>??
>
> Simply malloc will return null and allocation will fail.

