---
description: 06/22/2024
---

# Practical Fuzzing

## Structure Aware Fuzzing

### Libprotobuf Fuzzing

This fuzzing tool/technique will allow us to be able to fuzz complex data structures such as classes, structs, etc.&#x20;

Let's step through this awesome blog post together that Mobile Hacking Lab has put together through us.&#x20;

By the end, we should be able to "wrap" our heads around structure aware fuzzing.

### References

{% embed url="https://www.mobilehackinglab.com/blog/libprotobuf-fuzzing" %}
Definitely check this out
{% endembed %}

### When to use Structure Aware Fuzzing

We can choose to use this technique when we run into a series of target functions that we want to fuzz and are bound to predefined structs.

The tool of choice here would undoubtedly be [Protobuf](https://github.com/google/libprotobuf-mutator)!&#x20;

{% embed url="https://github.com/google/libprotobuf-mutator" %}
Repo for Protobuf
{% endembed %}

## Analyzing Vulnerable Code

**`vuln.c`:**&#x20;

```c
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person {
    unsigned long id;
    char name[64];
};

extern void add_person(struct person *new_person) {
    char buf[64];
    memset(buf, 0, sizeof(buf));
    unsigned int copied_size = 0;
    char *store_person_name = NULL;

    if (new_person->id > 100000 && new_person->id < 200000) {
        for (size_t i = 0; i < sizeof(buf) - 1; i++) {
            if (new_person->name[i] == '\x00') {
                if (i > 32){
                
                    copied_size = i;
                store_person_name = (char *)malloc(copied_size);
                break;
            }
            else{
                return;
            }
            buf[i] = new_person->name[i];
        }

        if (store_person_name) {
            memcpy(store_person_name, buf, copied_size);
        }
        
    }
}
```

Now that we have analyzed the code above, we now need to build a fuzzing harness for libprotobuf in order to perform structure-aware fuzzing on this particular function, `add_person`, and perform a heap buffer overflow crash.

Notice how we are creating a harness around the `person_data` struct which will act as a custom data type essentially. This will mutate all variables with their types. `Unsigned long id` (unsigned 64-bit integer) and `char name[64]` (byte value).

{% hint style="info" %}
During normal fuzzing, we'd try to pass a randomly mutated `char *`. However, in this case, we cannot due to the struct. That's why we are going this route.
{% endhint %}

### Pinpointing the Vulnerability

We can see that `new_person->id` is coming from input between 100,000-200,000 and continues to loop between and check every byte of `new_person->name`.

During the 2nd if statement (`if (new_person->name[i] == '\x00')`), if the byte is `\x00` (NULL byte), the loop is already at 32-bytes like above, each byte will then be allocated as heap memory hence `store_person_name` which then leads to a `malloc` call.

The programmer decided to allocate memory on the heap (using `malloc`) only if the length of the `new_person->name` is greater than `32`.&#x20;

Lastly, at the last line of code, 64-bytes of memory will be copied to the allocated heap memory and if the ACTUAL allocated memory is less than 64, new\_person->name will contain a NULL byte, causing a heap overflow.

#### In other words...

The loop that copies the `new_person->name` into `buf` should terminate before the `else` statement that returns the length of `new_person->name` into `buf` is less than or equal to `32`.

Now that we have the vulnerability identified, it's time to create a "Proto" file so we can identify this bug with our fuzzer.&#x20;

## Creating Proto File

In order to find this bug with fuzzing, we will need to rely on the **Libprotobuf structure-aware fuzzing** technique.

{% hint style="info" %}
<mark style="color:red;">**What makes this bug so interesting is that unlike stack overflows, heap overflows will not always cause an application to crash.**</mark>
{% endhint %}

With that said, we are going to need to rely on an <mark style="color:yellow;">Address Sanitizer</mark> (ASAN), which will <mark style="color:yellow;">detect any out of bound access attempts during runtime</mark>.&#x20;

In order to utilize ASAN during runtime and for our assessment, we need to compile our code with `clang` with the `-fsanitize=address` flag.&#x20;

In addition to our vulnerable source code, we will need to create a new file named `person_data.proto`. This will allow us to define our structure-aware fuzzing syntax.

`person_data.proto`:

```c
syntax = "proto2";

message person_data_fuzz {
	required uint64 fuzz_id = 1;
	required bytes fuzz_name = 2;
}
```

{% hint style="info" %}
Note how this looks almost identical to our structure that was contained within our source code.

```
struct person {
    unsigned long id;
    char name[64];
};
```
{% endhint %}

For our file, it needs to be in the same order, `1` and `2` that it comes declared within our `struct`.

This will allow us to properly mutate our inputted data.



