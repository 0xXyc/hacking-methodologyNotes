---
description: 04/24/2024
---

# while loop

## What is a `while` loop?

This is a type of loop that is used to repeat a specific block of code an unknown number of times until a specific condition is met. In other words, it will continue to loop through the section of code as long as the predetermined condition remains true.

A _counter_ is not required in a `while` loop.

Below is a `while` loop in c. This will be specifying an incrementing value.\
`while.c`:

```c
#include <stdlib.h>
#include <stdio.h>

int main() {

    int i = 0;
    while (i < 10){
        printf("Hi\n");
        i++;
    }

    return 0;

}
```

Next
