---
description: 02/21/2026
---

# crackme1

## Reverse Challenge

This is a simple challenge rated with a 4.5 quality and 1.0 difficulty.&#x20;



{% columns %}
{% column width="50%" %}
<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
{% endcolumn %}

{% column width="50%" %}
#### "crackme1"

**Language:**\
**C/C++**

**Quality:**\
**4.5**

**Difficulty:**\
**1.0**

<a href="https://crackmes.one/crackme/5ab77f5333c5d40ad448c100" class="button primary" data-icon="link">Check it out here</a>
{% endcolumn %}
{% endcolumns %}

{% columns %}
{% column %}
#### TLDR;

blah
{% endcolumn %}

{% column %}
<figure><img src="https://gitbookio.github.io/space-quickstart-images/sync-repo.svg" alt=""><figcaption></figcaption></figure>
{% endcolumn %}
{% endcolumns %}

## ReadMe.txt

```
Hi, 

This is my first CM. So, be nice. Hehe ;)

Date 		: 	12/10/2007 [dd/mm/year]


To Do & Rules	:	Stage 1 - Find Pass - No Patching.

			Stage 2 - Keygen - No Patching.

			Stage 3 - Patch it.


Additional  	:	Report any bug[s] you may come across. You may find

			one or two ;)		


Compiler 	:	Dev-C++


Difficulty	:	Level 1
```

Okay, this tells me that we are going to need to do this challenge sequentially.

1. Find the password.
2. Perform keygen.
3. Patch it.

## Finding Password

Bingo, we find a hardcoded password (string) within the `.rdata` section.

```
QbTTx1sE

// Also
    eax = "QbTTx1sE";
    var_3ch = "QbTTx1sE";
    
    (...)
        _scanf (data.004031fb, eax);
    eax = &s1;
    eax = _strlen (eax);
    if (eax != 8) {
        goto label_0;
    }
```

Digging further, we can see that there is a call to `scanf()` that will take user-input and stores the result in `eax`. We then call `strlen()` on `eax`, which contains our user-input from `scanf()`.

If our user-input does not have a length of `8` characters, we `jmp` to `label_0`, which is likely a program termination jump:

```
label_0:
        _printf ("\nSomething went wrong...\n\nPress Any Key To Quit");
        _getch ();
```

{% hint style="info" %}
`getch()` simply just reads a character input from the keyboard.
{% endhint %}

### Stage 1: Reversing the Password

```c
    _printf ("\n\nSTAGE 1\n");
    _printf ("*******\n\n");
    _printf ("Enter Password To Continue : ");
    eax = &s1;
    _scanf (data.004031fb, eax);
    eax = &s1;
    eax = _strlen (eax);
    if (eax != 8) {
        goto label_0;
    }
    var_10h = 0;
    do {
        if (var_10h > 7) {
            goto label_1;
        }
        eax = &var_ch;
        eax += var_10h;
        edx = eax - 0x20;
        eax = &var_ch;
        eax += var_10h;
        eax -= 0x20;
        eax = *(eax);
        al++;
        *(edx) = al;
        eax = &var_10h;
        *(eax)++;
    } while (1);
```

The variable `var_3ch` is initialized to `QbTTx1sE`.

`al++` is incrementing each character by one, but here is the caveat... the `strcmp` is checking `eax`, which also holds our password string, meaning that is not our password, but rather our password incremented by `1`.&#x20;

{% hint style="info" %}
If that was confusing, apologies.

Our password is literally that string, but decremented by `1` via `al++`.

So the reverse would be:

```
al--;
*(edx) = al;
eax = &var_10h;
*(eax)++;

//This would give us PaSSw0rD
```
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1).png" alt="" width="437"><figcaption></figcaption></figure>

## Stage 2: Keygen

```c
    _printf ("\nStage 1 completed!");
    _printf ("\n\n\nSTAGE 2\n");
    _printf ("*******\n\n");
    _printf ("\nName [2<=chars<=10] : ");
    eax = &s;
    _scanf (data.004031fb, eax);
    _printf ("\nSerial : ");
    eax = &var_54h;
    _scanf (data.00403241, eax);
    do {
        eax = &s;
        eax = _strlen (eax, 0, 0);
        if (var_10h > eax) {
            goto label_2;
        }
        eax = &var_ch;
        eax += var_10h;
        eax -= 0x40;
        eax = *(eax);
        eax += var_50h;
        eax--;
        var_50h = eax;
        eax = &var_10h;
        *(eax)++;
    } while (1);
```
