---
description: 02/21/2026
---

# "safe"

{% embed url="https://crackmes.one/crackme/5ab77f6533c5d40ad448cb87" %}

## How it Works (TLDR)

Simple and easy reverse PE32 Assembly challenge that uses `MessageBoxA()` from the WinAPI to create a simple numerical combination to a safe.&#x20;

&#x20;&#x20;

<figure><img src="../../.gitbook/assets/image (367).png" alt=""><figcaption></figcaption></figure>

As we can see the possibilities are numbers 1-5.

If incorrect, `kernel32.dll!Beep()` will be used to create a noise and `ExitProcess()` will swiftly terminate the program.

Upon inspecting further, a call to `strlen` can be seen in conjunction with the string, `"1234567890"`. It is checking if it has a length of `6`, if it does not, it will return and call `ExitProcess()`. However, if it has a length of `6` but is not the correct combination, it will call `kernel32.dll!Beep()` and play a sound from the speaker, then call `ExitProcess()`.&#x20;

### Locating Input Buffer

We can see that our input buffer is `0x403000`, at the start of the `.data` section of the portable executable (PE).&#x20;

```c
void __cdecl fcn.004010b1(int32_t arg_4h)
{
    int32_t iVar1;
    
    iVar1 = sub.kernel32.dll_lstrlenA(0x403000);
    *(int32_t *)(iVar1 + 0x403000) = arg_4h;
    return;
}
```

There is the another function with `void` type that holds the combination and performs some comparison checks with our input buffer and checks buffer positions in a specific order.

```c
void fcn.004010d3(void)
{
    int32_t iVar1;
    
    iVar1 = sub.kernel32.dll_lstrlenA(0x403000);
    if (iVar1 != 6) {
        return;
    }
    if (*(char *)0x403000 != (char)(str.1234567890[4] + -1)) {
        fcn.00401185();
    }
    if (*(char *)0x403002 != str.1234567890[4]) {
        fcn.00401185();
    }
    if (*(char *)0x403001 != str.1234567890[2]) {
        fcn.00401185();
    }
    if (*(char *)0x403004 != '1') {
        fcn.00401185();
    }
    if (*(char *)0x403005 != '3') {
        fcn.00401185();
    }
    if (*(char *)0x403003 != '5') {
        fcn.00401185();
    }
    if (data.00403109 != (code)0x1) {
        sub.user32.dll_MessageBoxA(0, "Good Job.", data.00403214, 0);
        return;
    }
    sub.kernel32.dll_Beep(100, 1000);
    return;
}
```

By reversing this chronologically from top-to-bottom, we come up with `453135`. However, paying attention closer, we see that the comparison checks are being conducted in a non-chronological order:

```
  0x403000  → position 0
  0x403002  → position 2
  0x403001  → position 1
  0x403004  → position 4
  0x403005  → position 5
  0x403003  → position 3
```

This changes our combination from `453135` to `435513`.&#x20;

<figure><img src="../../.gitbook/assets/image (368).png" alt=""><figcaption></figcaption></figure>

This does not give us a `kernel32.dll!Beep()`, but calls `MessageBoxA()` once more, passing `"Good Job."` confirming our success!
