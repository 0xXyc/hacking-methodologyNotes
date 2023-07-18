---
description: 07/06/2023
---

# Reversing Your First Program

Let's write a simple program in C, take it apart a few different ways, and ultimately reverse it!

## Static Code Analysis

hello.c:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int cars = 5;
    int *carsAddress = &cars;

    printf("Hello world! This is a simple program to help me learn about pointers in c.\n");
    printf("We currently have %d cars\n", cars);
    printf("Our cars can be found at the following address in memory: %p\n", (void*) &cars);
}
```

**Line 6**: it starts off with declaring a variable of `cars`, expecting an `int` data type of 5.

**Line 7**: Next, we are declaring `*carsAddress`as a pointer to the address-of our previously declared variable, `&cars`.

**Line 10-12**: A series of prints for variable digit, our pointer, and address-of variable.

#### Time to compile

```
gcc -g hello.c -o hello
```

## Ghidra

Since I compiled this binary with the debug arg (`-g`), we have all of the debugging symbols and cheatcodes that a reverse engineer could ever ask for. This is not common to find in the wild, so just keep that in mind.&#x20;

We are able to identify the <mark style="color:yellow;">entry point</mark> of our program:&#x20;

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

What is the entry point you might be asking?

The <mark style="color:yellow;">entry point</mark> performs any pre-compilation tasks before calling `main()`

Let's dig deeper:

This can be found at memory address: `0x00103da8`

<figure><img src="../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption><p>Disassembly of main()</p></figcaption></figure>

Above, we can see the `main()` function.

This was rather easy to find. I was able to quickly locate it on the left side within my Symbol Tree.

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

From here, we can see that the decompilation proves to be rather fruitful.

This looks rather similar to our source code, doesn't it?

Although it is a tad different, we are able to understand what is going on here.&#x20;

#### Wonder why it looks different?

Keep in mind that it will look different since `gcc` does weird things at time in regard to mitigations, protections, and optimization.

#### Using our Intuition

We see `return 0` which means we are likely using the `int` data type for `main()`.

This means that we can change the undefined data type by right-clicking it and selecting "Edit Function Signature".

* Change it to `int main(void)`

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

Let's dig into our functions within the Symbol Tree:

<figure><img src="../.gitbook/assets/image (2) (2).png" alt=""><figcaption></figcaption></figure>

### Time to Compile

Our decompilation is really looking good now, isn't it?

With that said, let's create a new file named `reversed-hello.c` and <mark style="color:yellow;">inject our header files at the top. Next, we will be ready for compilation!</mark>

<figure><img src="../.gitbook/assets/image (13) (7).png" alt=""><figcaption></figcaption></figure>

<mark style="color:yellow;">We can go ahead and ignore our errors</mark> since this was produced by the compiler anyways, it should be able to understand our code when re-interpreted at compile time.

Compile:

```
gcc -g reversed-hello.c -o reversed-hello
```

Execute:

```
./reversed-hello 
Hello world! This is a simple program to help me learn about pointers in c.
We currently have 5 cars
Our cars can be found at the following address in memory: 0x7ffe5c5cf56c
```

Congrats, you just reversed your first program!!!!
