## 1. Write runtime polymorphism without using `virtual`

```cpp
class Base {
public:
    void (*func_ptr)(); // Function pointer

    void setFunc(void (*f)()) {
        func_ptr = f;
    }

    void callFunc() {
        func_ptr();
    }
};

void sayHello() {
    std::cout << "Hello from derived!" << std::endl;
}
```

### ✅ Core Concept  
Manual runtime polymorphism using function pointers instead of virtual functions.

### 🔍 Why it's used  
Useful in systems programming or embedded environments where you want low-level control or cannot afford virtual tables.

### ⚠️ Caveats  
- No automatic type safety
- No inheritance hierarchy enforcement
- Error-prone for large systems

### 🧠 Analogy  
Think of a function pointer as a personal hotline instead of calling through a receptionist (virtual function table).

### ❓ Follow-up Questions and Answers

**Q: Can this be used with member functions?**  
A: Yes, but with added complexity. You’d use pointers to member functions and bind them with object instances manually.

**Q: What’s the performance difference with `vtable`?**  
A: Function pointers can be marginally faster in some cases as there's no indirection through the vtable, but lose flexibility.

**Q: When is this approach preferred?**  
A: In constrained environments (firmware, drivers) or when dynamic polymorphism is needed without full class hierarchies.

---

## 2. What is `vtable` and `vptr`?

### ✅ Core Concept  
- `vtable`: Compiler-generated table storing function pointers to virtual methods.
- `vptr`: Hidden pointer inside each object pointing to its class’s `vtable`.

### 🔍 Why it's used  
To support runtime polymorphism (late binding).

### ⚠️ Pitfalls  
- Missing virtual destructor causes only base class destructor to be called.
- Increased object size due to hidden `vptr`.

### 🧠 Analogy  
`vptr` is like your food order pointing to a menu (`vtable`). Depending on what you are (derived type), your menu is different.

### ❓ Follow-up Questions and Answers

**Q: Where is `vptr` stored?**  
A: Inside each object, typically as a hidden first member by the compiler.

**Q: Does `vtable` increase object size?**  
A: No, but `vptr` does. The table itself is shared among class instances.

**Q: Can `vtable` be shared?**  
A: Yes, all instances of a class share the same `vtable`. Only `vptr` is per-object.

---

## 3. Can a destructor be virtual?

### ✅ Core Concept  
Yes. Needed to ensure correct cleanup of derived objects via base pointers.

### 🔍 Why it's used  
To allow proper destruction of derived objects through base class pointers.

### ⚠️ Pitfalls  
Omitting `virtual` leads to undefined behavior and resource leaks.

### 🧠 Real-World Reasoning  
Without virtual destructors, destructors don’t behave polymorphically—major resource management issue in OOP.

### ❓ Follow-up Questions and Answers

**Q: Can a destructor be pure virtual?**  
A: Yes, and it must still have a definition even if declared `= 0`.

**Q: Will compiler still generate one if not defined?**  
A: Yes, unless explicitly deleted or overridden, compiler synthesizes default destructor.

---

## 4. Can a constructor be virtual?

### ❌ No, it cannot.

### 🔍 Why  
Because constructors build the vtable; they can’t rely on it for dispatch.

### ⚠️ Common Misunderstanding  
People confuse factory pattern usage with constructor polymorphism.

### 🧠 Analogy  
You can't call the help desk before the phone line is connected.

### ❓ Follow-up Questions and Answers

**Q: How to achieve polymorphic object creation?**  
A: Use factory or abstract factory pattern to construct correct derived types at runtime.

---



---

## 5. What is Singleton Pattern? Write its implementation.

### ✅ Core Concept  
Singleton ensures that only one instance of a class exists throughout the lifecycle of the program.

### ✅ Implementation

```cpp
class Singleton {
private:
    static Singleton* instance;
    Singleton() {}
public:
    static Singleton* getInstance() {
        if (!instance)
            instance = new Singleton();
        return instance;
    }
};
Singleton* Singleton::instance = nullptr;
```

### 🔍 Why it's used  
Used for global shared resources like configuration manager, logging service, etc.

### ⚠️ Pitfalls  
- Not thread-safe in this naive version.
- Can lead to hidden dependencies and tight coupling.

### 🧠 Analogy  
Like a CEO: there should be only one at a time.

### ❓ Follow-up Questions and Answers

**Q: How to make it thread-safe?**  
A: Use `std::mutex` or C++11 static local initialization (`static Singleton& getInstance() { static Singleton s; return s; }`).

**Q: Can Singleton be subclassed?**  
A: Generally no — that violates the purpose. If subclassing is needed, it’s a code smell.

---

## 6. Solve the Rain Water Trapping problem.

### ✅ Approach  
Use two-pointer technique to track left max and right max at each step.

```cpp
int trap(vector<int>& height) {
    int left = 0, right = height.size() - 1;
    int leftMax = 0, rightMax = 0, water = 0;

    while (left < right) {
        if (height[left] < height[right]) {
            leftMax = max(leftMax, height[left]);
            water += leftMax - height[left++];
        } else {
            rightMax = max(rightMax, height[right]);
            water += rightMax - height[right--];
        }
    }
    return water;
}
```

### 🔍 Why it works  
Water at a position = min(leftMax, rightMax) - height[i].

### ⚠️ Pitfalls  
- Don’t forget to handle edge cases like empty array or size < 3.

### ❓ Follow-up Questions and Answers

**Q: What’s the time complexity?**  
A: O(n), single pass with two pointers.

**Q: Can you solve it with a stack?**  
A: Yes — use monotonic stack to hold indices of walls.

---

## 7. Schedule meetings based on in-time and out-time intervals.

### ✅ Approach  
Sort meetings by end time and use greedy strategy to select non-overlapping intervals.

```cpp
bool comp(pair<int,int> a, pair<int,int> b) {
    return a.second < b.second;
}

int maxMeetings(vector<pair<int,int>> meetings) {
    sort(meetings.begin(), meetings.end(), comp);
    int count = 1, end = meetings[0].second;
    for (int i = 1; i < meetings.size(); i++) {
        if (meetings[i].first > end) {
            count++;
            end = meetings[i].second;
        }
    }
    return count;
}
```

### 🔍 Why it works  
Choosing the earliest finishing meeting leaves room for more meetings.

### ❓ Follow-up Questions and Answers

**Q: Can you do this with dynamic programming?**  
A: Yes, but greedy is optimal here due to the structure of the problem.

**Q: What’s the time complexity?**  
A: O(n log n) due to sorting.

---

## 8. Implement an LRU Cache

### ✅ Core Concept  
LRU = Least Recently Used. We evict the least recently accessed item on cache full.

### ✅ Implementation

```cpp
class LRUCache {
    int capacity;
    list<int> keys;
    unordered_map<int, pair<int, list<int>::iterator>> cache;

public:
    LRUCache(int cap) : capacity(cap) {}

    int get(int key) {
        if (cache.find(key) == cache.end()) return -1;
        keys.erase(cache[key].second);
        keys.push_front(key);
        cache[key].second = keys.begin();
        return cache[key].first;
    }

    void put(int key, int value) {
        if (cache.find(key) != cache.end())
            keys.erase(cache[key].second);
        else if (keys.size() == capacity) {
            cache.erase(keys.back());
            keys.pop_back();
        }
        keys.push_front(key);
        cache[key] = {value, keys.begin()};
    }
};
```

### 🔍 Why it's used  
Efficiently manage cache using O(1) access and O(1) eviction via hash map and doubly linked list.

### ❓ Follow-up Questions and Answers

**Q: Can this be thread-safe?**  
A: Needs external locking using mutex or atomic primitives.

**Q: What happens if keys are large objects?**  
A: Store references or use custom hash to avoid memory issues.

---



---

## 9. Given an array of n+2 elements where n elements are distinct and 2 are repeated, find the two repeated numbers.

### ✅ Core Concept  
Use either frequency map, XOR trick, or mathematical sum formulas to find the two duplicates.

### ✅ Efficient Approach
Using XOR:
1. XOR all array elements and numbers from 1 to n. The result will be XOR of the two repeated numbers (say x and y).
2. Use a set bit in the XOR result to divide elements into two sets and repeat XOR.

### ❓ Follow-up Questions and Answers

**Q: Why not just use a set or map?**  
A: That works but uses O(n) extra space. XOR solution gives O(1) space.

**Q: Can this be solved in-place?**  
A: Yes, by modifying array elements as negative flags (mark visited).

---

## 10. Explain the concept of Concurrency.

### ✅ Core Concept  
Concurrency means multiple tasks make progress within overlapping time periods. It's about structure and interleaving execution.

### 🧠 Analogy  
Cooking multiple dishes on different burners—one simmers while you stir another.

### ❓ Follow-up Questions and Answers

**Q: How is concurrency different from parallelism?**  
A: Concurrency = dealing with many things at once; Parallelism = doing many things at once (physically).

**Q: What are common problems in concurrency?**  
A: Race conditions, deadlocks, starvation, livelocks.

---

## 11. What is an Atomic variable in programming?

### ✅ Core Concept  
An atomic variable ensures that operations like read/write happen as a single, indivisible unit — no thread can interrupt.

### 🔍 Why it's important  
Used to prevent race conditions without locks in multi-threaded environments.

### ❓ Follow-up Questions and Answers

**Q: Is atomic always lock-free?**  
A: Not necessarily. It's implementation-dependent.

**Q: How are atomic operations implemented in hardware?**  
A: Often via atomic CPU instructions like `LOCK CMPXCHG` on x86.

---

## 12. What is a Critical Section?

### ✅ Core Concept  
A code segment that accesses shared resources and must not be concurrently executed by more than one thread.

### 🧠 Analogy  
Like using a single toilet stall — only one person can use it at a time.

### ❓ Follow-up Questions and Answers

**Q: How do you protect a critical section?**  
A: Using synchronization primitives like mutex, spinlock, or atomic variables.

**Q: Can critical sections be nested?**  
A: Yes, but must avoid deadlocks or use recursive mutexes.

---

## 13. Write a code to protect a critical section without using mutex/semaphore.

```cpp
std::atomic_flag lock = ATOMIC_FLAG_INIT;

void enterCritical() {
    while (lock.test_and_set(std::memory_order_acquire)); // spinlock
    // critical section
    lock.clear(std::memory_order_release);
}
```

### 🔍 Why this works  
`test_and_set` provides atomicity to implement a spinlock.

---

## 14. Define and implement your own custom Mutex.

```cpp
class CustomMutex {
    std::atomic_flag flag = ATOMIC_FLAG_INIT;

public:
    void lock() {
        while (flag.test_and_set(std::memory_order_acquire));
    }

    void unlock() {
        flag.clear(std::memory_order_release);
    }
};
```

### ❓ Follow-up Questions

**Q: Is this fair?**  
A: No. Starvation is possible. Fairness requires queueing or ticket lock.

---

## 15. Difference between Mutex and Semaphore.

| Feature       | Mutex                        | Semaphore                    |
|---------------|------------------------------|-------------------------------|
| Ownership     | Owned by thread              | No ownership                  |
| Use           | Mutual exclusion             | Signaling / resource count    |
| Value         | Binary (0/1)                 | Counting (>=0)                |
| Locking       | Lock/unlock                  | Wait/post                     |

---

## 16. What is Virtual Memory?

### ✅ Core Concept  
Virtual memory allows a process to think it has contiguous memory, while in reality the OS maps virtual addresses to physical memory + disk (paging).

### ❓ Follow-up Questions and Answers

**Q: What are page faults?**  
A: When a page is not in RAM and must be loaded from disk.

**Q: What is the page table?**  
A: A structure that maps virtual to physical addresses.

---

## 17. What is DMA (Direct Memory Access) and how is it implemented?

### ✅ Core Concept  
DMA allows devices to read/write memory without involving CPU, freeing it for other work.

### 🔍 How it works  
A DMA controller handles transfers between IO device and RAM. It triggers interrupts once done.

---

## 18. Difference between User Space and Kernel Space

| Property         | User Space                 | Kernel Space               |
|------------------|----------------------------|-----------------------------|
| Access Level     | Restricted                 | Full privileges             |
| Direct Hardware  | No                         | Yes                         |
| Crash Impact     | Limited to process         | System crash possible       |
| Examples         | User apps                  | Device drivers, OS internals|

---

## 19. What is Shared Memory and how is it implemented?

### ✅ Core Concept  
Shared memory allows multiple processes to access the same region of memory.

### 🔍 Implementation (Linux)  
- `shmget()` — allocate
- `shmat()` — attach
- `shmdt()` — detach
- `shmctl()` — control ops

---

## 20. How does the Kernel share commands with user applications?

### ✅ Answer  
Via System Calls. User-mode programs request services from the kernel through well-defined syscall interfaces.

---

## 21. Explain IPC (Inter-Process Communication)

### ✅ Core Mechanisms  
- Pipes
- Shared Memory
- Message Queues
- Sockets

Each has its pros and cons. Choice depends on whether processes are related and on speed/complexity trade-offs.

---

## 22. Design 3 APIs: create, use, destroy

```cpp
void* ptr = nullptr;

void create() {
    if (!ptr) ptr = malloc(1024);
}

void use() {
    if (ptr) memset(ptr, 1, 1024);
}

void destroy() {
    if (ptr) {
        free(ptr);
        ptr = nullptr;
    }
}
```

### ❗ Constraints  
- `create()` must allocate only once.
- `use()` operates only if allocated.
- `destroy()` frees memory safely.

---

## 23. Solve: XYZ + XYZ + XYZ = ZZZ

### ✅ Brute Force  
Try all combinations of single-digit X, Y, Z where XYZ * 3 = ZZZ.

### ✅ Valid Answer  
XYZ = 185 → 185 * 3 = 555 → Z = 5 ✔

---

## 24. Explain System Calls and how they work.

### ✅ Core Concept  
System calls are how user-mode applications request services from the kernel (e.g., file read, process create).

### 🔍 How it works  
1. User calls wrapper function (e.g., `read()`).
2. Trap into kernel mode using `int 0x80` or `syscall`.
3. Kernel performs action and returns to user.

---