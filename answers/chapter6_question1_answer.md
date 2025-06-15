# Basics of Code Optimization

This document discusses the fundamentals of code optimization, including the principal sources of optimization, the main areas or scopes of optimization, and the difference between local and global transformations.

---

## 1. Principal Sources of Optimization

Code optimization aims to improve a program's execution efficiency (e.g., speed, memory usage) without changing its output or behavior. The opportunities for such improvements, known as the **principal sources of optimization**, arise from identifying and transforming inefficient code constructs. These are general categories of improvements that optimizers look for.

1.  **Function-Preserving Transformations:** These are transformations that improve code by eliminating redundancy or simplifying computations within a basic block or procedure.
    *   **Common Sub-expression Elimination:** Reusing the result of an expression that has already been computed.
    *   **Copy Propagation:** Replacing a variable with its value after a copy statement, which can lead to dead code.
    *   **Dead Code Elimination:** Removing code that is unreachable or whose result is never used.
    *   **Constant Folding:** Evaluating constant expressions at compile time instead of at runtime.

2.  **Loop Optimizations:** Since programs spend most of their time in loops, optimizing them yields significant performance gains.
    *   **Code Motion:** Moving a loop-invariant computation (an expression that yields the same result in every iteration) outside the loop.
    *   **Strength Reduction:** Replacing an expensive operation (like multiplication) inside a loop with a cheaper one (like addition).
    *   **Induction Variable Elimination:** Identifying and eliminating redundant variables that are simple functions of the loop counter.
    *   **Loop Unrolling:** Reducing loop overhead by duplicating the loop body and decreasing the number of iterations.

3.  **Machine-Dependent Optimizations (Peephole Optimization):** These optimizations leverage specific features of the target machine architecture by examining a small "peephole" of instructions.
    *   **Efficient Instruction Usage:** Replacing a sequence of instructions with a single, more powerful instruction (e.g., using `INC` instead of `ADD 1`).
    *   **Redundant Load/Store Elimination:** Removing unnecessary instructions that load a value into a register when it's already there.

---

## 2. Main Areas of Code Optimization (by Scope)

Code optimization can be classified by the scope of the code that is being analyzed.

### a) Local Optimization
This is the simplest form of optimization, confined to a single **basic block** (a straight-line sequence of code with no branches in or out).
*   **Analysis:** The optimizer only needs to analyze the code within one block, so it doesn't need to track control flow between blocks.
*   **Techniques:** Local common subexpression elimination, constant folding within a block, and peephole optimization are common local techniques. A DAG is an excellent tool for performing local optimizations.
*   **Advantage:** Simple and fast to implement.

### b) Global Optimization
This type of optimization operates on an entire **procedure or function**. It requires analyzing the flow of control and data across all the basic blocks in a procedure.
*   **Analysis:** Requires **data-flow analysis** to gather information about how data is used across different paths in a Control Flow Graph (CFG). This is much more complex than local analysis.
*   **Techniques:** Global common subexpression elimination, global copy propagation, code motion out of loops, and most loop optimizations fall into this category.
*   **Advantage:** Much more powerful than local optimization, as it can find optimization opportunities that span multiple basic blocks.

### c) Inter-procedural Optimization
This is the most complex and largest-scoped form of optimization, analyzing the entire program across different functions and files.
*   **Analysis:** Requires understanding how functions call each other and how data is passed between them. It must consider issues like pointers and passing arguments by reference.
*   **Techniques:**
    *   **Inlining:** Replacing a function call with the body of the called function to eliminate call overhead.
    *   **Procedure Cloning:** Creating specialized versions of a function for different call sites.
    *   **Whole-program analysis** to eliminate dead code (unused functions) or propagate constants across function boundaries.
*   **Advantage:** Can find the most significant optimizations, but is very complex and time-consuming to perform.

---

## 3. Differentiating Local and Global Transformations

| Feature             | Local Transformation                                       | Global Transformation                                           |
|:--------------------|:-----------------------------------------------------------|:----------------------------------------------------------------|
| **Scope**           | A single basic block.                                      | An entire function or procedure.                                |
| **Information Needed** | Only the instructions within the current basic block.      | Information about the entire Control Flow Graph (CFG).          |
| **Analysis Required** | Simple analysis (e.g., building a DAG for the block).      | Complex **data-flow analysis** (e.g., reaching definitions, liveness analysis). |
| **Complexity**      | Relatively simple and fast.                                | Complex and computationally intensive.                          |
| **Example**         | Eliminating a common subexpression `a+b` that appears twice *within the same basic block*. | Moving a loop-invariant computation `x*y` from inside a loop to the block before the loop starts. |
| **Power**           | Less powerful, as it cannot see redundancies across branches. | More powerful, as it can optimize code across loops and conditional branches. | 