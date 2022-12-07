---
description: My favorite compiled language
---

# C

## OPSEC Considerations

* This will cover OPSEC considerations an operator/developer should follow when compiling C code

## GNU Project C and C++ Compiler (GCC)

### Arguments

* Stripping: -<mark style="color:yellow;">s</mark> -- Always enable
* \-<mark style="color:yellow;">fstack-protector</mark> -- Adds a canary value in the stack to catch buffer overflows (<mark style="color:red;">sometimes</mark>)&#x20;
* \-<mark style="color:yellow;">g</mark>
