---
description: 06/22/2023 (abandoned) but now brought back to life in 04/02/2024!
cover: https://lordicon.com/icons/wired/flat/1324-c-code-language.gif
coverY: 0
---

# C++

## Introduction

C++ is a <mark style="color:yellow;">cross-platform language</mark> that can be used to create <mark style="color:green;">high-performance</mark> applications.

It also gives programmers the power to be able to have a "high level" control over system resources and memory.

Known as an _extension_ to the C programming language.

Although the syntax is somewhat similar, the difference between C and C++ is that C++ supports classes and objects while C does not.&#x20;

### Reasons to use C++

* Very popular
* Found in OS's, GUI's, and embedded systems
* <mark style="color:yellow;">Object Oriented Programming (OOP) language</mark> that grants the developer access to structures and classes at low performance costs
* Can be used to develop applications for a multitude of systems
* Possesses similarities to C and Java

### Motivations

I want to obtain a better understanding of object oriented programming!

{% embed url="https://dev.to/gervg/how-to-think-in-object-oriented-way-29a2" %}
Check this out!
{% endembed %}

{% embed url="https://www.w3schools.com/cpp/cpp_intro.asp" %}
Initial Resource for C++ Introduction
{% endembed %}

## Oh no, another "hello world"...

Don't worry, we'll be getting into some pretty complicated stuff here shortly! :smirk:

**`hello-world.cpp`:**

<pre class="language-cpp"><code class="lang-cpp">#include &#x3C;iostream>

<strong>using namespace std;
</strong>
int main() {
    cout &#x3C;&#x3C; "This is my first C++ program, I won't stop until I get comfortable with OOP or C++!!\nHello World!!\n";
    return 0;
}
</code></pre>

#### Compile

```
g++ hello-world.cpp -o hello-world
```

#### Output

<figure><img src="../.gitbook/assets/image (191).png" alt=""><figcaption><p>Output</p></figcaption></figure>

### Explanation

```
#include <iostream>
```

A header file for an external library; it allows us to work with input and output objects.

<pre><code><strong>using namespace std;
</strong></code></pre>

This is considered a directive that tells the compiler to consider everything within the `std` namespace as if it were within the global namespace context.

By using this, we do not need to explicitly prefix each standard library element with `std::`; allowing for more concise code and aiding in readability -- exactly what we need right now for learning this.

However, if you wish to run without the standard namespace library, we can replace it with the `std` keyword along with the `::` operator for objects.

**(e.g.)**

**`hello-world.cpp` (omitting the standard namespace library):**

```cpp
#include <iostream>

int main() {
  std::cout << "Hello World!";
  return 0;
} 
```

**Note:** that `cout` will not append a <mark style="color:yellow;">new line</mark> at the end of each output (<mark style="color:yellow;">`\n`</mark>), this will need to be manually implemented. This is known as the <mark style="color:yellow;">"newline" character,</mark> also referred to as an <mark style="color:yellow;">"escape sequence"</mark>.

### What if you wanted to print out a variable?

**`variable.cpp`:**

```cpp
#include <iostream>

int main() {
  int myNum = 50;
  std::cout << "Your number is: " << myNum << "\n";
  return 0;
}
```
