---
description: >-
  Gotta start somewhere. Misewellbehere! Simple solutions that will ultimately
  push me into my ultimate goal of becoming 1337.
---

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

<figure><img src="../../.gitbook/assets/image (2) (7).png" alt=""><figcaption><p>Output</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (10) (4).png" alt=""><figcaption><p>Output</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (4) (3).png" alt=""><figcaption><p>Ouput</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption><p>Output</p></figcaption></figure>

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

 #include <stdio.h>

//Compute the perimeter and area of a rectangle with user-given dimensions

int main()
{

int length, width, perimeter, area;

    printf("The Hacker's Geometry Program!");

    printf("\nPlease enter the length of the rectangle (in inches): \n");
    scanf("%i", &length);

    printf("\nPlease enter the width of the rectangle (in inches): \n");
    scanf("%i", &width);

    perimeter = 2*(length+width);
    printf("\nThe perimeter of a rectangle with a length of %i inches and a width of %i is: %i inches \n", length, width, perimeter);

    area = length * width;
    printf("\nThe area of the rectangle is %d square inches \n", area);

    printf("\nKeep on coding!!!");

    return 0;
}
```

### Output:

<figure><img src="../../.gitbook/assets/image (4) (13).png" alt=""><figcaption><p>Output</p></figcaption></figure>

## Exercise 8

years.c:

```
#include <stdio.h>
#define daysinaweek 7

//Write a C program to convert specified days into years, weeks and days.
//Test Data : Number of days : 1329 Expected Output : Years: 3 Weeks: 33 Days: 3 

int main()
{
    int xdays, years, weeks, days;

    printf("Please enter a number of days:\n");
    scanf("%d", &xdays);
    years = xdays / 365;
    weeks = (xdays % 365) / daysinaweek;
    days = (xdays % 365) % daysinaweek;
    printf("\n%d days is equivalent to %d year(s), %d week(s), and %d day(s).\n",
        xdays, years, weeks, days);
    
    printf("\nThank you for using us! Take care!");

    return 0;
}
```

### Output:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (2).png" alt=""><figcaption><p>Output</p></figcaption></figure>

## Exercise 9

sum-integer.c:

```
#include <stdio.h>

// Write a program that accepts two integers from the user and calculate the sum of the two integers.

int main()
{

    int num1, num2, sum;

    printf("Hello, please add your first number!\n");
    scanf("%i", &num1);

    printf("Please enter your second number!\n");
    scanf("%i", &num2);

    sum = (num1 + num2);
    printf("\nThe sum of your two numbers is: %i", sum);
    printf("\nKeep on coding!!");

    return 0;

}
```

### Output:

<figure><img src="../../.gitbook/assets/image (1) (12).png" alt=""><figcaption><p>Output</p></figcaption></figure>

## Exercise 10

product.c:

```
#include <stdio.h>

// Write a program that accepts two integers from the user and calculates the product of them.

int num1, num2, product;

int main()
{

printf("Hello, please add your first number\n");
scanf("%i", &num1);

printf("Please add your second number!\n");
scanf("%i", &num2);

product = (num1*num2);
printf("\nThe product of your two numbers is %i", product);

return 0;

}
```

### Output:

<figure><img src="../../.gitbook/assets/image (1) (3) (2).png" alt=""><figcaption><p>Output</p></figcaption></figure>
