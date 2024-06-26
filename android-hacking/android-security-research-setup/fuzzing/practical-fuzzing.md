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
                if (i > 32)
                    copied_size = i;
                store_person_name = (char *)malloc(copied_size);
                break;
            }
            buf[i] = new_person->name[i];
        }

        if (store_person_name) {
            memcpy(store_person_name, buf, copied_size);
        } else {
            return;
        }
    }
}
```

Now that we have analyzed the code above, we now need to build a fuzzing harness for libprotobuf in order to perform structure-aware fuzzing on this particular function, `add_person`, and perform a heap buffer overflow crash.

Notice how we are creating a harness around the `person_data` struct which will act as a custom data type essentially. This will mutate all variables with their types. `Unsigned long id` (unsigned 64-bit integer) and `char name[64]` (byte value).
