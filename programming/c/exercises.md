# Exercises

{% embed url="https://www.w3resource.com/c-programming-exercises/basic-declarations-and-expressions/index.php" %}

## Exercise 1

person-data.c:

```
#include <stdio.h>

int main()
{
    printf("Name: Alexandra Abramov\n");
    printf("DOB: July 14, 1975\n");
    printf("Mobile: 99-9999999999\n");    
    return 0;
}
```

### Output:

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Output</p></figcaption></figure>

## Exercise 2

version.c:

```
#include <stdio.h> 

int main(int argc, char** argv) 
{
#if __STDC_VERSION__ >=  201710L
  printf("We are using C18!\n");
#elif __STDC_VERSION__ >= 201112L
  printf("We are using C11!\n");
#elif __STDC_VERSION__ >= 199901L
  printf("We are using C99!\n");
#else
  printf("We are using C89/C90!\n");
#endif
  return 0;
}
```

### Output:

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption><p>Output</p></figcaption></figure>

## Exercise 3

block-letters.c:

