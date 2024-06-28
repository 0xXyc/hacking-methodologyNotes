---
description: 06/22/2024
---

# Practical Fuzzing

## Structure Aware Fuzzing

{% hint style="info" %}
:rotating\_light: Before we jump in, you need to be aware that there is no support for the Android NDK for Apple Silicon/AARCH64 distributions of Linux. On your Mac, you will need to natively install protobuf via `brew install protoc` and utilize `aarch64-linux-androidx-clang++` during compilation processes.

Hopefully this avoids some frustration!
{% endhint %}

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

**Create a directory called `fuzzer`**.

```
mkdir fuzzer
```

**Now, create our Proto file**:

```
protoc person_data.proto --cpp_out=fuzzer
```

**This will create two files for us**:

```
person_data.pb.cc
person_data.pb.h
```

We now have the files required that we will need to include within our fuzzing harness.

## Creating the Fuzzing Harness

{% hint style="info" %}
Perform the next steps within the `fuzzer` directory.

Also, I recommend when developing your harness that you set up an environment where you can visually see both the source code/target function as well as the `harness.cpp` file you are creating for a much more smooth experience.
{% endhint %}

#### My setup

<figure><img src="../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

**Create `harness.cpp` file**:

```
touch harness.cpp
```

**`harness.cpp`**:

```c
#include "/path_to/libprotobuf-mutator/src/libfuzzer/libfuzzer_macro.h"
#include "person_data.pb.h"

struct person_data {
    unsigned long id;
    char name[64];
};

void add_person(struct person *new_person);

DEFINE_PROTO_FUZZER(const person_data_fuzz &input){
    struct person_data real_input;
    memset(&real_input, 0, sizeof(struct person_data));

    real_input.id = input.fuzz_id();

    unsigned int size = input.fuzz_name().length();
    const char *data = input.fuzz_name().data();

    if (size > sizeof(real_input.name)){
        memcpy(real_input.name, data, sizeof(real_input.name));
    }
    else{
        memcpy(real_input.name, data, size);
    }

    add_person(&real_input);
}
```

We need to define `DEFINE_PROTO_FUZZER` and add the arguments abiding to the protobuf syntax that we defined earlier within the `person_data_fuzz`. All variables of this object will be mutated by libfuzzer.&#x20;

So what we are pretty much doing is simply creating a standard `struct`, `person_data` object and copying (`memcpy`) any mutated data to this object and passing it to the `add_person()` function like it normally would in the normal application.

As you can see, we are calling pre-defined variables as functions for them to be copied into `real_input` objects.

### Compile the Harness

```
aarch64-linux-android21-clang++ -w -c vuln.c -o vuln.o
```

Now, compile the harness, vulnerable code, and Libprotobuf files together which will result in our complete task.

```
aarch64-linux-android33-clang++ libprotobuf-mutator/src/libfuzzer/libfuzzer_mutator.cc libprotobuf-mutator/src/libfuzzer/libfuzzer_macro.cc fuzzer/person_data.pb.cc fuzzer/harness.cpp vuln.o -o protobuf_fuzz -I libprotobuf-mutator -lprotobuf -lprotobuf-mutator -fsanitize=address,fuzzer
```

This will produce a binary called `protobuf_fuzz` that will act as our fuzzer.

## Running the Fuzzer

We're now ready to run our fuzzer!

```
./protobuf_fuzz
```

{% hint style="info" %}
<mark style="color:yellow;">**If applicable**</mark>**: You may want to add flags to the protobuf binary such as: `-ignore_ooms=1 -ignore_crashes=1 -ignore_timesouts=1 -fork=1`**
{% endhint %}

We get a crash!

{% hint style="info" %}
**Keep in mind that there are numerous reasons as to why a program can crash and a fuzzer can also create numerous crashes. Most of which, are not important to us! With that said, be sure to review all and armed with that information, revisit the code base once more for another review. At that point, we can begin distinguishing meaningful crashes against false positives.**
{% endhint %}

### Crash Analysis

<figure><img src="../../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

We can now review this crash in order to develop an exploit to meet our desired cyber effect goals. As a result, this may come in the form of shellcode, ROP, or JOP in order to deliver our payload to the target successfully. It just depends on your environmental constraints and target.&#x20;
