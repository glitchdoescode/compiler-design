# Code Optimization Techniques Explained

This document provides explanations and examples for several principal code optimization techniques used by modern compilers.

---

### a) Common Sub-expression Elimination

**Concept:**
A **common sub-expression** is an expression that appears multiple times in the code and computes the same value each time. This optimization technique involves identifying these redundant computations, calculating the result once, storing it in a temporary variable, and then using that temporary variable for all subsequent occurrences.

**Example:**
Consider the following code:
```
// Before Optimization
c = a * b + 5;
d = a * b - e;
```

**After Optimization:**
The expression `a * b` is common. The optimizer transforms the code to:
```
t1 = a * b;
c = t1 + 5;
d = t1 - e;
```
This avoids a redundant multiplication, making the code faster. This can be done locally (within a basic block) using a DAG, or globally (across a whole function) using data-flow analysis.

---

### b) Copy Propagation

**Concept:**
**Copy propagation** is the process of replacing occurrences of a variable with the value it was assigned from another variable. After a simple copy statement like `x = y`, the compiler replaces later uses of `x` with `y`, as long as the values of `x` and `y` have not changed in between. This optimization does not make the code faster on its own, but it often opens up opportunities for other optimizations, particularly dead code elimination.

**Example:**
```
// Before Optimization
x = y;
z = 10 + x;
```

**After Copy Propagation:**
```
x = y;
z = 10 + y;
```
Now, if the variable `x` is not used anywhere else, the assignment `x = y` becomes **dead code** and can be completely removed by the dead code elimination pass.

---

### c) Loop Invariant Computations (Code Motion)

**Concept:**
A **loop-invariant computation** is an expression inside a loop whose value does not change from one iteration of the loop to the next. Such computations are redundant and can be safely moved outside the loop to be computed only once. This technique is also known as **code motion**.

**Example:**
```
// Before Optimization
while (i < limit - 2) {
    // ...
    i = i + 1;
}
```

**After Optimization:**
The expression `limit - 2` is loop-invariant.
```
t = limit - 2;
while (i < t) {
    // ...
    i = i + 1;
}
```
This prevents the `limit - 2` subtraction from being performed in every single iteration of the loop.

---

### d) Strength Reduction

**Concept:**
**Strength reduction** is a technique, most often used in loops, that replaces a computationally "strong" (i.e., expensive) operation with an equivalent but "weaker" (i.e., cheaper/faster) one. The classic example is replacing a multiplication with an addition.

**Example:**
Consider accessing elements of an array `a` inside a loop, assuming each element takes 4 bytes.
```
// Before Optimization
for (i = 0; i < 10; i++) {
    t1 = 4 * i;   // Expensive multiplication
    val = a[t1];
    // ...
}
```

**After Strength Reduction:**
The optimizer introduces a new variable that holds `4*i` and updates it with a cheap addition in each iteration.
```
s = 0;
for (i = 0; i < 10; i++) {
    t1 = s;
    val = a[t1];
    // ...
    s = s + 4; // Cheap addition
}
```
The expensive multiplication `4 * i` inside the loop has been replaced by a simple addition `s = s + 4`.

---

### e) Peephole Optimization

**Concept:**
**Peephole optimization** is a machine-dependent optimization performed on the final target code. The optimizer looks at a small, sliding window of instructions (the "peephole"), typically 2 to 4 instructions at a time, and replaces these short sequences with a shorter or faster equivalent sequence.

**Examples:**

1.  **Redundant Load/Store Elimination:**
    ```assembly
    // Before
    STORE R1, A  // Store register R1 into memory location A
    LOAD  A, R1  // Immediately load A back into the same register R1
    ```
    The `LOAD` instruction is redundant and can be deleted.

2.  **Using Machine Idioms:**
    ```assembly
    // Before (e.g., adding 1)
    ADD #1, R1
    ```
    If the target machine has a faster single instruction for this:
    ```assembly
    // After
    INC R1
    ```

3.  **Algebraic Simplification:**
    `X = X + 0` or `X = X * 1` can be eliminated.

---

### f) Loop Optimization

**Concept:**
This is a broad category of techniques focused on making loops run faster, as this is where programs typically spend the most time. Code motion and strength reduction are specific types of loop optimization. Other techniques include:

*   **Loop Unrolling:** Reduces the overhead of the loop test and branch by duplicating the loop body and reducing the number of iterations.
*   **Loop Fusion (Jamming):** Combines two adjacent loops that iterate over the same range into a single loop, reducing loop overhead.
*   **Loop Fission (Distribution):** Splits a single loop into multiple loops, each handling a part of the original loop's body. This can improve data locality and enable other optimizations.

**Example of Loop Unrolling:**
```
// Before
for (i = 0; i < 100; i++) {
    a[i] = b[i] + c[i];
}
```

**After Unrolling (by a factor of 4):**
```
for (i = 0; i < 100; i += 4) {
    a[i]   = b[i]   + c[i];
    a[i+1] = b[i+1] + c[i+1];
    a[i+2] = b[i+2] + c[i+2];
    a[i+3] = b[i+3] + c[i+3];
}
```
This code has only 25 iterations instead of 100, meaning the loop condition `i < 100` and the increment `i += 4` are executed only 25 times, reducing the total loop control overhead.

---

### g) Register Allocation

**Concept:**
**Register allocation** is the process of assigning program variables to the limited number of fast CPU registers available on the target machine. The goal is to maximize the use of registers and minimize the number of slow memory accesses (loads and stores). Well-allocated registers can have a massive impact on program performance.

**Process:**
1.  **Allocation:** The compiler decides which variables are the best candidates to reside in registers. Variables used frequently, especially inside loops, are high-priority candidates.
2.  **Assignment:** The compiler assigns a specific physical register to each chosen variable.
3.  **Spilling:** If there are more variables that would benefit from being in a register than there are available registers, some variables must be "spilled" to memory. The compiler must generate `STORE` instructions to save the variable to the stack and `LOAD` instructions to bring it back when needed.

This is a complex problem, often modeled as a graph-coloring problem, and is one of the most important functions of an optimizing compiler. 