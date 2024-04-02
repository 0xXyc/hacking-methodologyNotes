---
description: 04/02/2024
---

# User Input

## Introduction

We will now be introducing user input into our C++ program.&#x20;

Previously, we used `cout`, we will now be using `cin` to get user input.

### Example

**`user-input.cpp`:**

```cpp
#include <iostream>

int main() {
  int x;
  std::cout << "Type your number here: ";
  std::cin >> x;
  std::cout << "Your new number is: " << x << "\n";
  return 0;
}
```

### Addition Calculator

Let's create a simple calculator that obtains the sum of two numbers.

**`addition-calc.cpp`**:

```cpp
#include <iostream>

int main() {
    int x, y;
    int sum;

    std::cout << "Simple addition calculator!\n\n";
    std::cout << "Type a number: ";
    std::cin >> x;

    std::cout << "Type another number to obtain the sum of both numbers: ";
    std::cin >> y;

    sum = x + y;
    std::cout << "The sum of your two numbers is: " << sum << "\n";

    return 0;

}
```

**Result:**

<figure><img src="../../.gitbook/assets/image (192).png" alt=""><figcaption><p>Wooohoooo math!</p></figcaption></figure>
