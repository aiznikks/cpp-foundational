# Interview Preparation Notes

> Copy and paste this into your `.md` editor (e.g., VS Code) and fill in examples or additional comments as needed.

---

## Table of Contents

1. [C Language Fundamentals](#c-language-fundamentals)  
2. [Linked Lists](#linked-lists)  
3. [Operating Systems](#operating-systems)  
4. [Data Structures & Algorithms](#data-structures--algorithms)  
5. [I/O and C Gotchas](#io-and-c-gotchas)  
6. [Final Tips](#final-tips)

---

## C Language Fundamentals

### 1. Integer and Pointer Size on 32-bit vs 64-bit

- **Why it matters**: Portability and ABI compatibility.
- **Typical sizes**  
  | Type    | 32-bit (ILP32) | 64-bit (LP64 on Linux) |
  | ------- | -------------- | ---------------------- |
  | `int`   | 4 bytes        | 4 bytes                |
  | `long`  | 4 bytes        | 8 bytes                |
  | pointer | 4 bytes        | 8 bytes                |

- **Key point**: On Windows x64 (LLP64), `long` remains 4 bytes but pointers are 8 bytes.

---

### 2. Little Endian vs. Big Endian

- **Definitions**  
  - **Little endian**: least-significant byte at lowest address.  
  - **Big endian**: most-significant byte at lowest address.

- **Detection program**  
  ```c
  #include <stdio.h>

  int main(void) {
      unsigned int x = 0x12345678;
      unsigned char *p = (unsigned char*)&x;
      printf("%s endian\n",
             (p[0] == 0x78) ? "Little" : "Big");
      return 0;
  }
  ```

- **Use-case**: Converting network byte order (big endian) to host order.

---

### 3. C Memory Map Sections

1. **`.text`** – program code  
2. **`.data`** – initialized globals/statics  
3. **`.bss`** – uninitialized globals/statics (zeroed)  
4. **Heap** – dynamic (`malloc`, `free`)  
5. **Stack** – local variables, call frames

- **Variable placement**  
  - Local (non-static) → stack  
  - `static`/global with init → `.data`  
  - `static`/global without init → `.bss`  
  - `malloc` → heap

---

### 4. Bitwise Operator Macros

```c
// Check if bit i is set
#define IS_SET(v, i)   ( ((v) >> (i)) & 1 )

// Set bit i
#define SET_BIT(v, i)  ( (v) |=  (1U << (i)) )

// Clear bit i
#define CLR_BIT(v, i)  ( (v) &= ~(1U << (i)) )

// Toggle bit i
#define TOG_BIT(v, i)  ( (v) ^=  (1U << (i)) )

// Count bits set (GCC/Clang builtin)
static inline int count_bits(unsigned v) {
    return __builtin_popcount(v);
}
```

- **In-depth**: Undefined if `i ≥` width of `v`. Hardware popcount vs software.

---

### 5. Complex Pointer Declarations

1. **Pointer to function `(int, float) → int`**  
   ```c
   int (*pf)(int, float);
   ```
2. **Pointer to array[5] of `int`**  
   ```c
   int (*p_arr)[5];
   ```
3. **Array[5] of pointers to functions `(int,int) → int`**  
   ```c
   int (*arr[5])(int, int);
   ```
4. **Array of pointers to functions returning pointer to function returning `void`**  
   ```c
   void (*(*arr_fp[])())(void);
   ```
5. **Function returning pointer to function `(int,int) → int`**  
   ```c
   int (*f(void))(int, int);
   ```
6. **Pointer to array[5] of pointers to array[10] of pointers to `int`**  
   ```c
   int *(*(*p)[5])[10];
   ```
7. **Pointer to array[10] of pointers to array[5] of function pointers `(int,int)→int`**  
   ```c
   int (*(*(*p)[10])[5])(int, int);
   ```

- **Parsing strategy**: Start at `p`, handle `[]`, `()`, then `*`.

---

### 6. Best Type for Pointer Arithmetic

- **Use** `uintptr_t` / `intptr_t` (from `<stdint.h>`) for portable pointer math.

---

### 7. Generic Pointer Type

- **`void *`** can hold address of any data object (but must cast to dereference).

---

### 8. `malloc` vs. `calloc`

| Feature         | `malloc(n)`        | `calloc(count, size)`   |
| --------------- | ------------------ | ----------------------- |
| Bytes allocated | `n`                | `count * size`          |
| Initialization  | Uninitialized      | Zero-initialized        |
| Use-case        | Fast allocation    | Arrays, zeroed memory   |

- **Detail**: `calloc` may use demand-zero pages.

---

### 9. Aligned `malloc` (4096 bytes) + `free`

```c
void *aligned_malloc(size_t size) {
    void *ptr = NULL;
    if (posix_memalign(&ptr, 4096, size) != 0)
        return NULL;
    return ptr;
}

void aligned_free(void *p) {
    free(p);
}
```

---

### 10. Pool Allocator (1 MB)

- **Approach**:  
  1. Static `char pool[1<<20];`  
  2. Maintain free-list of blocks in pool  
  3. On `malloc`, split a free block; on `free`, coalesce

- **In-depth**: Thread-safety (mutex or lock-free).

---

### 11. `sbrk`-based Allocator

```c
#include <unistd.h>

void *my_sbrk_malloc(size_t size) {
    void *p = sbrk(size + sizeof(size_t));
    if (p == (void*)-1) return NULL;
    *((size_t*)p) = size;
    return (char*)p + sizeof(size_t);
}

void my_sbrk_free(void *ptr) {
    // simplistic: cannot reclaim—real impl needs free-list
}
```

- **Note**: Modern systems prefer `mmap`.

---

### 12. Storage Classes in C

- `auto` (default local)  
- `register` (hint)  
- `static` (persistent/local or global scope)  
- `extern` (external linkage)

---

### 13. *(Reserved)*

---

### 14. `auto` in C vs C++

- **C**: storage-class specifier (default for locals).  
- **C++**: type deduction (`auto x = expr;`).

---

### 15. `volatile` Keyword

- **Prevents** compiler optimizations on object accesses.  
- **Use-case**: memory-mapped I/O, signal handlers.

---

### 16. `const` Keyword

- **Read-only pointer**: `const int *p;`  
- **Constant pointer**: `int * const p;`  
- **Both**: `const int * const p;`

---

### 17. `volatile const`

- **Use-case**: hardware status register—read-only but may change external to CPU.

---

### 18. `typedef` for Function Pointer

```c
typedef int (*binary_op_t)(int, int);
```

---

### 19. Struct Packing & Alignment

- **GCC**:  
  ```c
  struct __attribute__((packed, aligned(1))) Packed { … };
  ```
- **MSVC**:  
  ```c
  #pragma pack(push,1)
  struct Packed { … };
  #pragma pack(pop)
  ```

---

### 20. `sizeof(struct)` vs. `sizeof(union)`

- **Struct**: sum of member sizes + padding for alignment.  
- **Union**: size = largest member size (then padded to align).

---

### 21. Unions for Memory Optimization

- **Use-case**: variant data (tagged unions).

---

### 22. Custom `memcpy` / `strcpy`

```c
void *my_memcpy(void *dst, const void *src, size_t n) {
    unsigned char *d = dst, *s = (unsigned char*)src;
    while (n--) *d++ = *s++;
    return dst;
}

char *my_strcpy(char *dst, const char *src) {
    char *d = dst;
    while ((*d++ = *src++));
    return dst;
}
```

---

### 23. String Literals in Memory Map

- Stored in the read-only data segment (often `.rodata`).

---

### 24. Swap Two Integers

1. **With temp** (safe)  
   ```c
   int tmp = a; a = b; b = tmp;
   ```
2. **Without temp (arithmetic)**  
   ```c
   a = a + b;
   b = a - b;
   a = a - b;
   ```
3. **Without temp (XOR)**  
   ```c
   a ^= b; b ^= a; a ^= b;
   ```
- **Best**: method 1 (no overflow/UB).

---

### 25. Division Without `/`

```c
unsigned divide(unsigned dividend, unsigned divisor) {
    unsigned quotient = 0, bit;
    for (bit = 1U<<31; bit && divisor <= dividend; bit >>= 1) {
        if (divisor * bit <= dividend) {
            dividend -= divisor * bit;
            quotient |= bit;
        }
    }
    return quotient;
}
```

---

### 26. *(Reserved)*

---

## Linked Lists

1. **Basic CRUD** (`insert`, `delete` at k-th, `printList`)  
2. **Insert/Delete at middle** in one pass (slow/fast pointers)  
3. **Insert/Delete k-th from end** in one pass (two-pointer)  
4. **Cycle detection & entry** (Floyd’s algorithm + reset)  
5. **Intersection point** (two-pointer switch-heads technique)  
6. **Reverse list** (iterative three-pointer or recursion)  
7. **Remove duplicates**  
   - Sorted: one-pass compare adjacent  
   - Unsorted: hash set of seen values

---

## Operating Systems

1. **Priority inheritance**: prevent inversion by boosting lock-holder’s priority  
2. **Mutex vs Semaphore vs Spinlock**  
   - Mutex: exclusive, may block  
   - Semaphore: counter-based, may wake multiple  
   - Spinlock: busy-wait, no blocking  
3. **Interrupt handler & latency**: code run on IRQ, latency = IRQ→handler start  
4. **Internal interrupt flow**: exception → mode switch → context save → vector lookup → handler → restore  
5. **Producer–Consumer**: bounded buffer with two semaphores + mutex  
6. **Reader–Writer**: multiple readers or one writer; variants for fairness  
7. **Same address in different processes?** Yes—virtual memory isolates them

---

## Data Structures & Algorithms

1. **Stack using queues**: either push O(n)/pop O(1) or vice versa  
2. **Rotate matrix anticlockwise**: transpose → reverse columns  
3. **Undo/Redo/write** (Google Docs style)  
   - Two stacks: `undo`, `redo`  
   - **write**: push action to `undo`, clear `redo`  
   - **undo**: pop `undo`, apply inverse, push to `redo`  
   - **redo**: pop `redo`, reapply, push to `undo`

---

## I/O and C Gotchas

1. **Modify string literal**  
   ```c
   char *g = "Hello";
   g[0] = 'h'; // UB! Use char g[] = "Hello";
   ```

2. **Infinite self-ref struct**  
   ```c
   struct Node { int a; struct Node node; }; // ILLEGAL
   ```

3. **Correct self-ref struct**  
   ```c
   struct Node { int a; struct Node *next; };
   printf("%lu
", sizeof(struct Node));
   ```

4. **Pointer then char**  
   ```c
   struct Node { int a; struct Node *next; char c; };
   printf("%lu
", sizeof(struct Node)); // shows padding after 'c'
   ```

5. **Char between int and pointer**  
   ```c
   struct Node { int a; char c; struct Node *next; };
   printf("%lu
", sizeof(struct Node)); // may pad after 'c'
   ```

---

## Final Tips

- **Contextualize** each answer with real-world examples (embedded, networking, OS).  
- **Highlight pitfalls** (overflow, UB, alignment).  
- **Mention tools** (`readelf`, `valgrind`, `perf`).  
- **Engage** by asking: “Optimize for speed, size, or clarity?”

Good luck!
