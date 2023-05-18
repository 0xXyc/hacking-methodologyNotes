# Primitives // Data Types

## Data Types

Fundamental to any program. <mark style="color:yellow;">Types tell us what our data means and what kind of operations we can perform on them</mark>.&#x20;

### Primitives

This is classified as the <mark style="color:yellow;">smallest possible data type</mark>.&#x20;

This is not by size, but by the fact that they cannot be broken down into pieces.&#x20;

* A better word would be indivisible

### Complex - struct

Using a <mark style="color:yellow;">primitive data type to make something more complex</mark>.

Note: it is possible to combine multiple structs to make even bigger structures

The reason you would do this is because of very complex data structures.

#### Example 1

**Explanation:** The variable `today` is of data type `Date`.

```c
Date today;
```

#### Example 2

**Explanation:** The variable `lol` is of data type `coordinates`.

```c
coordinates lol;
```

### Linked List

<mark style="color:yellow;">A linked list is a chain of things</mark>. This can be whatever you want to be.&#x20;

For example, you could make a linked list of `coordinates`.

You have to build this data type yourself.

It can also be explained as a linear data structure that includes a series of connected nodes. Each node will store the data and the address of the next node.

<figure><img src="https://cdn.programiz.com/sites/tutorial2program/files/linked-list-concept.png" alt=""><figcaption></figcaption></figure>

#### Example

```
struct node
{
  int data;
  struct node *next;
};
```

### Arrays

<mark style="color:yellow;">An array allows you to store one or more things inside of a group</mark>.&#x20;

<mark style="color:yellow;">You can name it whatever you want</mark>.

It is stored in <mark style="color:yellow;">square brackets</mark> `[]`.

#### Example 1

```c
int grades[10];
```

|    |    |    |
| -- | -- | -- |
| 1  | 2  | 3  |
| 4  | 5  | 6  |
| 12 | 20 | 44 |
| 11 |    |    |

#### Example 2

```c
char balloons[3];
```

|     |     |     |
| --- | --- | --- |
| 'A' | 'B' | 'C' |

### String

<mark style="color:yellow;">Simply put, a string is a double quotes sequence of characters</mark>.

Each entry is a character, including spaces.

Note: DO NOT GET THIS CONFUSED WITH `char`.

Char is always single quoted, not double quoted.

#### Example

```
"I like programming in C"
```

However, syntactially, you must terminate the string at the end and expect to add another character for null/termination. At the very end of the string, you must put a null character.

The null character is written as `\0` in C.

#### Example

```
"I like programming in C"\0
```

### Constant

<mark style="color:yellow;">This is a variable that cannot change</mark>.&#x20;

A value that does not change. It is constant.

> "Something you can write down."

#### Example 1

```c
int ghosts = 98;
```

This can also be used with floats.

#### Example 2

```c
float ghosts = 98.5
```

## Numeric Data Types

We will be discussing <mark style="color:yellow;">Int</mark>, <mark style="color:yellow;">Float</mark>, and <mark style="color:yellow;">Double</mark> data types here.

### Int

If you wanted a data type that will be used for the number of dogs that you own, you would want to use int because I highly doubt you will have a fraction of a dog... lol.

#### Example

```c
int dogs = 6;
```

### Float

<mark style="color:yellow;">Precision</mark> -- <mark style="color:yellow;">how much data a variable can hold</mark>.

Floats do not take up much space.

A very precise number, not a whole number (1,2,3...).

```
4.4444242344242
```

#### Example

```
printf("%
```

### Double

When you want to store lots and lots of data.

Doubles are much more standardized.&#x20;

However, doubles take up double the space that a float would, just keep this in mind.

If you are on a small device with not much memory, you might want to consider using floats.

#### Example 1

We want to print some values:

This will get turned into a double automatically by C.

```c
printf("%i %f", 241242214214, 2424424242, 2424242424242422);
```

#### Example 2

```c
printf("%i %f %f\n", 1, 1.111, 1.1111F);
```
