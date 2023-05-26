# Arenas

## Introduction

In order to handle multi-threaded applications, <mark style="color:yellow;">glibc allows for more than one region of memory to be active at a time</mark>.&#x20;

These memory regions are called "<mark style="color:yellow;">arenas</mark>".

There is one arena, called the main arena. The main arena of the program contains the heap that single threaded applications will use and is found after the binary has been loaded into memory.
