# Answer to Question 6

---

### a) Discuss the various storage allocation strategies in detail.

Storage allocation is the process of assigning memory space to the variables and data of a program. The runtime environment of a language manages this process. There are three primary storage allocation strategies, each suited for different kinds of data and program structures.

#### 1. Static Allocation

In static allocation, the memory for all data objects is allocated at a fixed location by the compiler **before the program begins execution**. This memory remains allocated for the entire duration of the program's execution.

*   **When it's used:**
    *   Global variables.
    *   Static local variables (e.g., `static int count` in C).
    *   Constants and literal pools.
    *   Program code itself.
*   **Implementation:** The compiler determines the size and address of each static data object. These addresses are fixed and can be placed directly into the generated code.
*   **Advantages:**
    *   **Efficiency:** Access is very fast as addresses are absolute and known at compile time. No runtime overhead for allocation or deallocation.
    *   **Simplicity:** Easy for the compiler to manage.
*   **Disadvantages:**
    *   **Inflexible:** The size of the data must be known at compile time. This prevents the use of dynamic data structures (like variable-sized arrays) and recursion.
    *   **Wasteful:** Memory is allocated for the entire program run, even if the data is only needed for a small part of it.

#### 2. Stack Allocation

Stack allocation is a runtime strategy where memory is managed using a **stack** data structure. Memory is allocated and deallocated in a last-in, first-out (LIFO) manner.

*   **When it's used:**
    *   **Function/Procedure Activations:** Each time a function is called, a new **activation record** (or stack frame) is pushed onto the stack.
    *   **Local Variables:** Non-static local variables of a function are stored in its activation record.
    *   **Function Parameters:** Arguments passed to a function.
    *   **Control Information:** Return addresses, saved machine state (registers).
*   **Implementation:** A stack pointer (SP) register points to the top of the stack. When a function is called, the SP is decremented (on stacks that grow down) to allocate space for the new activation record. When the function returns, the SP is restored to its previous value, effectively "popping" the frame and deallocating the memory.
*   **Advantages:**
    *   **Efficient:** Allocation and deallocation are very fast, involving just pointer arithmetic.
    *   **Supports Recursion:** Each recursive call gets its own stack frame, allowing multiple instances of local variables to exist simultaneously.
    *   **Memory Reuse:** Memory space is automatically reclaimed and reused as functions return.
*   **Disadvantages:**
    *   **Fixed Size:** The size of local variables must be known at compile time to be placed in the stack frame. (Some languages allow limited dynamic stack allocation).
    *   **Limited Lifetime:** Data cannot be kept alive beyond the lifetime of the function that created it. A function cannot return a pointer to its local variable.

#### 3. Heap (Dynamic) Allocation

Heap allocation is the most flexible strategy. Memory is allocated and deallocated at **any time** during program execution, under the direct control of the programmer. The heap is a region of memory managed separately from the stack.

*   **When it's used:**
    *   For data objects whose size is unknown at compile time.
    *   For data objects that must persist longer than the function that created them.
    *   Complex data structures like linked lists, trees, and graphs.
*   **Implementation:** Programmers explicitly request memory using functions like `malloc()` in C or the `new` operator in C++/Java. This allocates a block of memory from the heap and returns a pointer to it. The memory remains allocated until it is explicitly freed (e.g., using `free()` in C) or automatically reclaimed by a **garbage collector**.
*   **Advantages:**
    *   **Maximum Flexibility:** Allows for fully dynamic data structures that can grow and shrink at runtime.
    *   **Global Lifetime:** Data exists until it is explicitly or implicitly deallocated, regardless of function calls.
*   **Disadvantages:**
    *   **Higher Overhead:** Allocation and deallocation are more complex and slower than stack allocation, as the memory manager must find a suitable block of free memory.
    *   **Memory Leaks:** In languages without automatic garbage collection, failing to free allocated memory leads to memory leaks, where the program's memory usage grows over time.
    *   **Fragmentation:** Over time, the heap can become fragmented into many small, non-contiguous free blocks, which may make it impossible to allocate a large contiguous block even if the total free memory is sufficient.

---

### b) Write the three address code sequence for the assignment statement. `P := (X + Y) * (X - C)`

Three-Address Code (TAC) is an intermediate representation where each instruction has at most one operator on the right-hand side. We use temporary variables (e.g., `t1`, `t2`) to store intermediate results.

**Statement:** `P := (X + Y) * (X - C)`

**Three-Address Code Sequence:**
```
t1 = X + Y
t2 = X - C
t3 = t1 * t2
P = t3
```

**Explanation:**

1.  `t1 = X + Y`
    *   The first sub-expression `(X + Y)` is evaluated.
    *   The result is stored in a temporary variable `t1`.

2.  `t2 = X - C`
    *   The second sub-expression `(X - C)` is evaluated.
    *   The result is stored in a new temporary variable `t2`.

3.  `t3 = t1 * t2`
    *   The results of the two sub-expressions (now in `t1` and `t2`) are multiplied.
    *   The final result of the right-hand side is stored in `t3`.

4.  `P = t3`
    *   The final result is assigned to the variable `P`. This is a copy instruction. In some optimized versions, this step might be combined with the previous one if the target architecture supports it (`P = t1 * t2`). 