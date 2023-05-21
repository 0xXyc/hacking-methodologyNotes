# Linked Lists

Think of it as a complex, linear data structure.

## Creating the Node

```
struct node {
  int data;
  struct node *next;
};
```

Create a structure, consists of data (data),&#x20;

## Malloc

You must then call the `malloc()` function to allocate memory for this node. You should do this in the main function.

```
int main()
{
  struct node * head = malloc(sizeof(struct node));
  head->data = 45;
  head->link = NULL;
  return 0;
}
```
