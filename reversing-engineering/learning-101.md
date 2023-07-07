---
description: 07/06/2023
---

# Learning 101

Let's take a simple program in C and take it apart a few different ways!

## Static Code Analysis

pointers2.c:

```c
#include <stdio.h>

int main (void)
{

    int digit = 42;
    int *addressOfDigit = &digit;

    printf("Hello, we will be learning about pointers here!\n\n");
    printf("The value of our int data type, digit is: %d\n", *addressOfDigit);
    printf("The address of the digit variable is: \nDecimal: %d Hex: %p\n", addressOfDigit);

}
```

**Line 6**: it starts off with declaring a variable of `digit`, expecting an `int` data type of `42`.

**Line 7**: Next, we are declaring `*addressOfDigit` as a pointer to the address-of our previously declared variable, `&digit`.

**Line 10-12**: A series of prints for variable digit, our pointer, and address-of variable.

## Ghidra

Since I compiled this binary with the debug arg (`-g`), we have all of the debugging symbols and cheatcodes that a reverse engineer could ever ask for. This is not common to find in the wild, so just keep that in mind.&#x20;

