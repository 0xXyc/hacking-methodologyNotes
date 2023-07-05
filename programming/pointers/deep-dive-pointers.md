---
description: 07-05-2023
---

# Deep Dive: Pointers

Basically, I am trying to FULLY wrap my head around pointers in C and what makes C such a superior language when it comes to vulnerability research and making your program do EXACTLY what you want.

{% embed url="https://www.freecodecamp.org/news/pointers-in-c-are-not-as-difficult-as-you-think/#1-what-exactly-are-pointers" %}

## Part 1: What are Pointers?

Let's use this snippet of code for our understanding:

```c
#include <stdio.h>

int main(void)
{

int digit = 42;
printf("The address of digit is: %d\n", &digit);

}
```

#### What is going on above?

A <mark style="color:yellow;">block of memory</mark> is <mark style="color:green;">reserved</mark> by the compiler to hold an `int` value.

The name of this block is `digit` and the value stored is `42`.

We do not care about the value of the address as this is randomized each time. However, we can access this value using the <mark style="color:yellow;">address-of operator</mark> (ampersand) to obtain the value of the address.

#### Execution

```
./pointers

The address of digit is: 1960835900
```

However, <mark style="color:yellow;">what if we wanted to get the value of</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`digit`</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">rather than the address of</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">`digit`</mark><mark style="color:yellow;">?</mark>

We would use the <mark style="color:yellow;">value-at or the dereference operator which is simply an asterisk (\*)</mark>.

The syntax would be:

```c
printf("The value of digit is: %d\n", *(&digit));
```

#### Execution

```
./pointers

The value of digit is: 42
```

## Part 2: Definition and Notation

The address of a variable can be stored in another variable known as a pointer variable.

The <mark style="color:yellow;">syntax for storing a variable's address to a pointer is</mark>:

```c
dataType *pointerVariableName = &variableName;
```

In our case:

```c
#include <stdio.h>

int main (void)
{

    int digit = 42;
    int *addressOfDigit = &digit;
    addressOfDigit = &digit;

    printf("Hello, we will be learning about pointers here!\n\n");
    printf("The value of our int data type, digit is: %d\n", *addressOfDigit);
    printf("The address of the digit variable is: \nDecimal: %d Hex: %p\n", addressOfDigit);


}
```

#### Key Takeaways:

* A pointer (\*) variable is a special variable in the sense that it is used to store an address of another variable.

## Part 3: Practical Usage

```c
#include <stdio.h>

struct Person
{
    char name[64];                              // roughly a 68-byte structure -- 0x44
    int age;
};

int main(int argc, char *argv[])
{
    struct Person people[100];                  // create an array of structures named people

    struct Person * p_Person = &people[0];         // points to the first person in the array

    int i = 0;
    for (i = 0; i < 100; i++)
    {
        p_Person->age = 0;

        p_Person += 1;

        p_Person->name[0] = 0;
    }

    return 0;
}
```
