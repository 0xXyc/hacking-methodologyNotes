# AsyncIO & Asynchronous Programming

## Introduction

### Traditional, Synchronous Programming

Normally, functions will run synchronously. For example, say that you had three different functions within your code, traditionally, your functions will wait until a return true or termination and then execute the next function in the code.

```
func1()
func2()
func3()
```

### Asynchronous Programming

Please note, this is <mark style="color:yellow;">NOT to be confused with multithreading</mark>. This is completely different.

Hypothetically, we have three functions and we are waiting on `func1()` to finish, however, we do NOT want to waste CPU time, so we decide to execute `func2()`. We are NOT running func1() and `func2()` simultaneously, but rather putting `func1()` to sleep or in a waiting state and use our CPU time on `func2()` or `func3()` while we wait to be very efficient.&#x20;

In order to achieve this, we will need to import the `asyncio` library.

```
import asyncio
```

<mark style="color:yellow;">You want to define a function as asynchronous</mark>, <mark style="color:yellow;">rather than the whole program</mark>.

To achieve this, you will use the `async` keyword:

```
async def main():
```

The next step is to sleep this&#x20;
