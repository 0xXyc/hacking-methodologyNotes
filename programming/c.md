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
* \-<mark style="color:yellow;">g</mark> -- <mark style="color:yellow;"></mark> <mark style="color:red;">Never enable when rolling out and sending around. This is the debugging arg.</mark>

## Cheat sheets

{% embed url="https://lzone.de/cheat-sheet/GCC" %}
Cheat sheet
{% endembed %}

{% embed url="https://man7.org/linux/man-pages/man1/gcc.1.html" %}
Official man page
{% endembed %}
