---
description: 02/17/2026
---

# "dailycracking"

{% embed url="https://crackmes.one/crackme/5ab77f5333c5d40ad448c0e0" %}

## How it Works (TLDR)

Simple and easy challenge that uses a string that is stored in a buffer and uses a custom implementation to grab metadata from a System Information File (`.nfo`) to identify system date/time (specifically the day of the month).&#x20;

**There is a string within the `.rdata` section of the PE at offset `0x00403004`:**

```
;-- str.Crackit:
0x00403004          .string "Crackit" ; len=8
```

### Main()

```c
// WARNING: Variable defined which should be unmapped: var_20h

undefined4 main(void)
{
    int32_t iVar1;
    int32_t var_28h;
    char *s;
    int32_t var_20h;
    int32_t var_1ch;
    FILE *stream;
    FILE **var_14h;
    int32_t var_10h;
    int32_t *piStack_c;
    int32_t var_8h;
    
    var_10h = 0x10;
    fcn.00401970();
    ___main();
    var_10h = 0x20;
    var_8h = (int32_t)&var_20h;
    fcn.00401970();
    piStack_c = &var_10h;
    var_1ch = (int32_t)__iob + 0x20;
    var_20h = (int32_t)str.pass:;
    _fputs();
    stream = __iob;
    var_1ch = 8;
    var_20h = (int32_t)piStack_c;
    _fgets();
    var_20h = (int32_t)piStack_c;
    iVar1 = _isOk();
    if (iVar1 == 0) {
        var_20h = (int32_t)piStack_c;
        _secretB((char *)piStack_c);
        var_1ch = (int32_t)__iob + 0x20;
        var_20h = (int32_t)piStack_c;
        _fputs();
    } else {
        var_20h = (int32_t)piStack_c;
        _secretA((char *)piStack_c);
        var_1ch = (int32_t)__iob + 0x20;
        var_20h = (int32_t)piStack_c;
        _fputs();
    }
    return 0;
}
```

### Secret functions A & B

`_secretB` writes the string "`wrong!"` in a buffer and `_secretA` writes the string "`right!`" in a buffer and exits the program via the WinAPI call, `ExitProcess()`.&#x20;

### \_isOk function

```c
// WARNING: Variable defined which should be unmapped: var_2ch

undefined4 _isOk(int32_t param_1)
{
    int32_t iVar1;
    int32_t var_3ch;
    int32_t var_38h;
    int32_t var_34h;
    int32_t var_30h;
    int32_t var_2ch;
    char *src;
    char *s1;
    char **dest;
    char acStack_1b [7];
    undefined4 uStack_14;
    int32_t var_10h;
    undefined4 uStack_c;
    
    var_10h = (int32_t)&var_2ch;
    uStack_14 = 0x20;
    fcn.00401970();
    uStack_c = 0;
    s1 = (char *)0x8;
    src = *(char **)0x402000;
    var_2ch = (int32_t)&dest;
    _strncpy();
    var_2ch = (int32_t)acStack_1b;
    _getDay((char *)var_2ch);
    var_2ch = param_1;
    src = (char *)&dest;
    iVar1 = _strcmp();
    if (iVar1 == 0) {
        uStack_c = 1;
    }
    return uStack_c;
}
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The string aforementioned, "`Crackit`" is stored at `0x402000`. It is copied into a stack buffer.

`getDay()` calls `strftime(s, 3, "%d", ptr)` which writes the current day as a 2-character string (e.g. `"17"`) into `acStack_1b`.

`acStack_1b` overlaps at offset 5, so the "`it\0`" part of the string gets truncated and overwritten with the day of the month, producing "`Crack`+\<day-of-the-month>" as the password, generating a new password with each day.

* For example, today was February, 17th, so the password was `Crack17`.&#x20;

### Troubleshooting Weird printf/STDOUT issues

For some reason, I wasn't able to see the string "`right!`" or "`wrong!`" because of some type of issue, but I fixed that with the following command which could then be used to confirm the password was indeed correct:

```
printf "Crack17" | ./dailycracking.exe
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
