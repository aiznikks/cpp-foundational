# Interview Preparation Questions

---

## C Questions

1. Integer and integer pointer size in 32-bit and 64-bit machines?
2. What is little endian, big endian?  
   Write a program to find if a machine is using little endian or big endian.
3. What sections are available in the C memory map?  
   Which variable is stored in which section?
4. Write macros using bitwise operators:
   - Check if bit is set or not
   - Set ith bit
   - Unset ith bit
   - Toggle ith bit
   - Count number of bits set
5. Define below pointers:
   1. Pointer to a function with argument `int`, `float` returning `int`.
   2. Pointer to an array with size 5.
   3. Array of 5 pointers to functions with arguments `int, int` returning `int`.
   4. Array of pointers to functions returning pointer to function returning `void`.
   5. Declare a function returning pointer to function with arguments `int, int` returning `int`.
   6. Pointer to an array of 5 pointers to arrays of 10 pointers to integers.
   7. Pointer to an array of 10 pointers to arrays of 5 pointers to functions that take two integers and return an integer.
6. Which type is best for pointer operations like `+`, `-`, `*`, `/` using address?
7. Which type of pointer is used to point to any kind of data?
8. Difference between `malloc` and `calloc`.
9. Write a custom `malloc` which returns an address aligned with 4096 and a `free` function.
10. Write a custom `malloc` and `free` using a memory pool of 1MB.
11. Write custom `malloc` and `free` using `sbrk` system call.
12. Storage classes in C.
13. *(Intentionally left blank or missing)*
14. Difference between `auto` keyword in C and C++.
15. What is the `volatile` keyword in C?
16. What is the `const` keyword? Define:
   - Read-only pointer
   - Constant pointer with writable value
   - Constant pointer with read-only value
17. Use case of `volatile` and `const` keyword together.
18. Define a function pointer type with `typedef`.
19. How to declare a structure without padding or with custom alignment?
20. How to calculate size of union and size of structure.
21. How can union be used to optimize memory?
22. Define custom `memcpy` and `strcpy` functions.
23. Where are string literals stored in the C memory map?
24. Swap two integers in the following 3 ways. Which one is best?
   - With temp
   - Without temp
   - Without temp using XOR
25. Divide operation without using `/` operator.
26. *(Intentionally left blank or missing)*

---

## Linked List

1. Write a linked list with `insert`, `delete` at kth position, and `printList` functions.
2. Write function to insert/delete node at middle in linked list (in one traversal).
3. Write function to insert/delete node at kth position from end (in one traversal).
4. Detect a cycle in linked list and find the starting point of the cycle.
5. Find the intersection point of two linked lists.
6. Reverse a linked list.
7. Remove duplicates from linked list.

---

## OS Questions

- [Priority Inheritance Explained – YouTube](https://www.youtube.com/watch?v=xw_OuOhjauw)
- [Qualcomm Interview Qs – GitHub](https://github.com/realabbas/big-companies-interview-questions/blob/master/companies/qualcomm/qualcomm.md)
- [Linux Interrupts in Kernel – Embetronicx](https://embetronicx.com/tutorials/linux/device-drivers/interrupts-in-linux-kernel/)

1. Priority inheritance and its solution.
2. Mutex, Semaphore, Spinlock – when to use which?
3. What is an interrupt handler? What is interrupt latency?
4. How is interrupt handled by the OS internally?
5. Producer-Consumer problem.
6. Reader-Writer problem.
7. Can two different processes have variables with the same address?

---

## DSA Questions

1. Implement stack using queue.
2. Rotate matrix anticlockwise.
3. Write `undo`, `redo`, and `write` functions for Google Docs:
   - Writing word by word
   - Can write word anywhere in document
   - Clear all redo after writing a new word

---



## I/O Questions

---

### 1. Modify String Literal

```c
#include <stdio.h>
int main() {
    char* greet = "Hello";
    greet[0] = 'h';
    printf("%s", greet);
}
```

---

### 2. Self-referential Structure (Incorrect – leads to infinite size)

```c
#include <stdio.h>
struct Node {
    int a;
    struct Node node;
};
int main() {
    return 0;
}
```

---

### 3. Self-referential Structure (Correct – using pointer)

```c
#include <stdio.h>
struct Node {
    int a;
    struct Node* node;
};
int main() {
    printf("%lu", sizeof(struct Node));
}
```

---

### 4. Structure: Pointer followed by Char

```c
#include <stdio.h>
struct Node {
    int a;
    struct Node* node;
    char c;
};
int main() {
    printf("%lu", sizeof(struct Node));
}
```

---

### 5. Structure: Char in between Int and Pointer

```c
#include <stdio.h>
struct Node {
    int a;
    char c;
    struct Node* node;
};
int main() {
    printf("%lu", sizeof(struct Node));
}
```
"""
