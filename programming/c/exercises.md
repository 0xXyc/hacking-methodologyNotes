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

```
#include <stdio.h>

int main()
{
    printf("######\n#\n#\n#####\n#\n#\n#\n#\n");

    printf("   #######\n ##      ##\n#\n#\n#\n#\n#\n ##      ##\n  #######");
    
    return 0;
}
```

### Output:

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>Ouput</p></figcaption></figure>

## Exercise 4

reverse.c:

```
#include <stdio.h>
#include <string.h>

//Test Characters: 'X', 'M', 'L'
//We want to reverse XML to LMX

int main ()
{
    char char1 = 'X';
    char char2 = 'M';
    char char3 = 'L';

    printf("Hello, the reverse of %c%c%c is %c%c%c\n",
        char1, char2, char3,
        char3, char2, char1);

    return 0;
}
```

### Output:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Output</p></figcaption></figure>

## Exercise 5

perimeter-area.c:

```
#include <stdio.h>


//Compute the perimeter and area of a rectangle with a length of 7 inches and a width of 5 inches
//Perimeter of a rectangle P = 2(l+w)

int main()
{
    int length, width, perimeter;
    printf("Please enter the length of the rectangle (in inches): \n");
    scanf("%i", &length);

    printf("Please enter the width of the rectangle (in inches): \n");
    scanf("%i", &width);

    perimeter = 2*(length+width);
    printf("The perimeter of a rectangle with a length of %i inches and a width of %i is: %i \n", length, width, perimeter);

    return 0;
}
```

### Output:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>Output</p></figcaption></figure>
