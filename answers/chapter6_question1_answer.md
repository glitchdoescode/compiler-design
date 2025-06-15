# Basics of Code Optimization

This document discusses the principal sources of optimization, the three main areas of code optimization, and differentiates between local and global transformations.

---

## What is Code Optimization?

Code optimization is the process of transforming a piece of code to make it more efficient (either in terms of execution time or memory usage) without changing its output or behavior. It is a crucial phase in a compiler, aiming to produce code that runs faster, uses less memory, or consumes less power.

---

## Principal Sources of Optimization

Optimizations can be achieved by identifying and eliminating redundancies, reorganizing computations, and utilizing hardware features more effectively. The principal sources of optimization opportunities arise from:

1.  **Redundant Computations:**
    *   **Common Subexpressions:** Expressions that are computed multiple times with the same operands.
        *   Example: `x = (a + b) * c; y = (a + b) / d;` Here, `(a + b)` is a common subexpression.
    *   **Loop Invariant Computations:** Calculations within a loop that produce the same result in every iteration.
        *   Example: `for(i=0; i<n; ++i) { x = y * z + i; }` If `y` and `z` don't change in the loop, `y * z` is loop invariant.

2.  **Inefficient Code Structures:**
    *   **Unreachable Code:** Code segments that can never be executed.
        *   Example: `if (DEBUG_MODE == false) { /* unreachable if DEBUG_MODE is always false */ }`
    *   **Dead Code:** Code whose results are never used.
        *   Example: `x = y + 1; x = z * 2;` The first assignment to `x` is dead code if `x` is not used before the second assignment.
    *   **Strength Reduction:** Replacing expensive operations with cheaper ones.
        *   Example: Replacing `x * 2` with `x + x` or `x << 1`.

3.  **Memory Access Patterns:**
    *   **Inefficient Array Access:** Accessing array elements in a non-sequential manner that leads to cache misses.
    *   **Redundant Loads and Stores:** Repeatedly loading a value from memory when it's already in a register, or storing a value that is immediately overwritten.

4.  **Control Flow Inefficiencies:**
    *   **Unnecessary Jumps:** Jumps to jumps, or jumps that can be eliminated by reordering code.
    *   **Complex Conditional Structures:** Boolean expressions that can be simplified.

5.  **Instruction-Level Parallelism:**
    *   Identifying independent instructions that can be executed in parallel by modern processors.

6.  **Register Utilization:**
    *   Keeping frequently used variables in registers to avoid memory access.
    *   Efficiently allocating registers to minimize spills (storing register contents to memory).

7.  **Procedure Calls:**
    *   **Inline Expansion:** Replacing a procedure call with the body of the procedure to reduce call overhead.
    *   **Tail Call Optimization:** Converting certain recursive calls into iterative loops.

---

## Three Main Areas of Code Optimization

Code optimization can be broadly categorized into three main areas based on the scope and type of transformations applied:

### 1. High-Level Optimizations (Source-Level or Intermediate Representation)

These optimizations are typically performed on a high-level intermediate representation (like Abstract Syntax Trees or Three-Address Code) before target code generation. They are often machine-independent.

*   **Focus:** Algorithmic improvements, structural changes, and high-level transformations.
*   **Examples:**
    *   **Loop Optimizations:**
        *   **Loop Unrolling:** Reducing loop overhead by duplicating the loop body.
        *   **Loop Fusion (Jamming):** Combining adjacent loops with the same iteration range.
        *   **Loop Fission (Distribution):** Splitting a single loop into multiple loops.
        *   **Loop Invariant Code Motion:** Moving computations that are constant within a loop outside the loop.
    *   **Procedure Optimizations:**
        *   **Function Inlining:** Replacing a function call with its body.
        *   **Tail Recursion Elimination:** Converting tail-recursive functions into iterative constructs.
    *   **Data Structure Optimizations:**
        *   **Array Restructuring:** Changing array layout for better cache performance (e.g., array of structs to struct of arrays).
    *   **Algorithmic Optimizations:** (Sometimes guided by pragmas or programmer input)
        *   Replacing an inefficient algorithm with a more efficient one (e.g., bubble sort with quicksort). This is often beyond automatic compiler capabilities but can be enabled via library calls or specific constructs.
    *   **Constant Folding:** Evaluating constant expressions at compile time.
        *   Example: `x = 2 + 3;` becomes `x = 5;`
    *   **Constant Propagation:** Replacing variables with their known constant values.
        *   Example: `PI = 3.14; area = PI * r * r;` becomes `area = 3.14 * r * r;`
    *   **Dead Code Elimination:** Removing code that does not affect the program's output.

### 2. Low-Level Optimizations (Target-Dependent)

These optimizations are performed on a lower-level intermediate representation or directly on the target machine code. They are often machine-dependent and exploit specific hardware features.

*   **Focus:** Efficient use of machine resources (registers, memory, instruction set).
*   **Examples:**
    *   **Register Allocation:** Assigning variables to CPU registers to minimize memory access.
    *   **Instruction Scheduling:** Reordering instructions to improve pipeline utilization and hide latencies.
    *   **Peephole Optimization:** Examining a small "window" of instructions and replacing them with a shorter or faster sequence.
        *   Example: `STORE R1, A; LOAD R1, A;` can be eliminated if R1 is not modified.
    *   **Strength Reduction:** Replacing expensive operations with equivalent cheaper ones (e.g., multiplication by a power of 2 with a shift).
    *   **Machine Idiom Recognition:** Replacing sequences of operations with specialized machine instructions.
        *   Example: Using an `increment` instruction instead of `load, add 1, store`.
    *   **Code Hoisting/Sinking:** Moving instructions to positions where they are executed less frequently or more optimally.

### 3. Data Flow Analysis Based Optimizations

These optimizations rely on data flow analysis, which gathers information about how data values are computed and used throughout the program. This information enables more sophisticated transformations.

*   **Focus:** Understanding the flow of data to enable more global and precise optimizations.
*   **Examples:**
    *   **Common Subexpression Elimination (CSE):** Identifying and eliminating redundant computations of the same expression.
        *   Local CSE: Within a single basic block.
        *   Global CSE: Across basic block boundaries.
    *   **Copy Propagation:** Replacing occurrences of a variable with its source after a copy assignment (e.g., `y = x; ... use y ...` becomes `... use x ...`).
    *   **Live Variable Analysis and Dead Code Elimination:** Identifying variables whose values are no longer needed, allowing the removal of their computations if they have no other side effects.
    *   **Reaching Definitions:** Determining which assignments to a variable might reach a particular point in the code.
    *   **Available Expressions:** Identifying expressions whose values have already been computed and are available for reuse.

---

## Differentiating Between Local and Global Transformations

Optimizations are also classified based on the scope of the program segment they operate on:

### Local Transformations (Intra-Basic Block)

*   **Definition:** Local transformations are applied to a small, contiguous segment of code, typically a **basic block**. A basic block is a sequence of instructions with a single entry point and a single exit point, with no jumps into or out of the middle of the block.
*   **Scope:** Limited to a single basic block.
*   **Complexity:** Generally simpler and faster to implement because they don't require analysis of the entire program's control flow.
*   **Information Used:** Relies on information available only within the basic block.
*   **Examples:**
    *   **Peephole Optimization:** Optimizing a small window of instructions.
    *   **Local Common Subexpression Elimination:** Eliminating redundant computations within the same basic block.
        *   Example:
            ```
            a = b + c;
            ... (no changes to b or c)
            d = b + c; // b+c is locally common
            ```
    *   **Local Constant Folding/Propagation:** Evaluating constants or propagating them within a block.
    *   **Local Dead Code Elimination:** Removing unused assignments within a block.
    *   **Strength Reduction (local):** Replacing `x = y * 2` with `x = y + y` within a block.

### Global Transformations (Inter-Basic Block)

*   **Definition:** Global transformations are applied to a larger scope, such as an entire procedure (function), a loop, or even the entire program. They consider the flow of control and data across basic block boundaries.
*   **Scope:** Extends beyond single basic blocks, often covering entire functions or loops.
*   **Complexity:** More complex and computationally intensive because they require data flow analysis of larger program segments.
*   **Information Used:** Requires information about control flow (e.g., control flow graphs) and data flow (e.g., live variables, reaching definitions) across basic blocks.
*   **Examples:**
    *   **Global Common Subexpression Elimination:** Eliminating redundant computations across different basic blocks.
    *   **Loop Optimizations:**
        *   **Loop Invariant Code Motion:** Moving code out of loops.
        *   **Loop Unrolling, Fusion, Fission.**
    *   **Global Constant Propagation:** Propagating constants across block boundaries.
    *   **Global Dead Code Elimination:** Removing unused computations whose effects don't propagate.
    *   **Register Allocation (Global):** Allocating registers across an entire function.
    *   **Data Flow Analysis based optimizations:** Most techniques like live variable analysis, reaching definitions, available expressions are inherently global.
    *   **Interprocedural Optimization:** Optimizations performed across function call boundaries (e.g., inlining, interprocedural constant propagation).

### Comparison

| Feature             | Local Transformations                 | Global Transformations                      |
|---------------------|---------------------------------------|---------------------------------------------|
| **Scope**           | Single Basic Block                    | Multiple Basic Blocks (Procedures, Loops)   |
| **Complexity**      | Simpler, Faster                       | More Complex, Slower                        |
| **Analysis Needed** | Minimal, within-block                 | Data Flow Analysis, Control Flow Graphs     |
| **Effectiveness**   | Good for localized improvements       | Can yield more significant improvements     |
| **Implementation**  | Easier                                | Harder                                      |

---

In summary, code optimization is a multifaceted process involving various techniques applied at different levels and scopes. Understanding the principal sources of optimization opportunities and the distinction between local and global transformations is key to designing effective compilers that produce high-performance code. 