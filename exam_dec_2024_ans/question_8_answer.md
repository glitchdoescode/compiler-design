# Answer to Question 8

---

### (a) What is Local transformation and Global transformation?

Local and global transformations refer to the scope over which a code optimization technique is applied. The distinction is crucial because global optimizations are generally more powerful but also more complex to implement.

#### Local Transformation

A **local transformation** is an optimization that is applied to a small, localized region of the code, typically within a single **basic block**. A basic block is a sequence of straight-line code with no branches in or out, except at the beginning and end.

*   **Scope:** A single basic block.
*   **Analysis Required:** The compiler only needs to analyze the code within the basic block. This is relatively simple because the control flow is linear.
*   **Examples:**
    *   **Local Common Subexpression Elimination:** Detecting and eliminating redundant calculations within one block (e.g., using a DAG).
    *   **Peephole Optimization:** Examining a small, sliding "window" of instructions and replacing them with a shorter or faster sequence. For instance, eliminating redundant loads and stores like `STORE R, a; LOAD a, R`.
    *   **Constant Folding:** Evaluating expressions with constant operands at compile time (e.g., changing `x = 2 + 3;` to `x = 5;`).

#### Global Transformation

A **global transformation** is an optimization that is applied over a larger scope than a single basic block, such as an entire function, procedure, or loop. These transformations require a broader analysis of the program's behavior.

*   **Scope:** An entire function, procedure, or loop structure.
*   **Analysis Required:** The compiler must perform **data-flow analysis** on a **control-flow graph (CFG)** of the program. This analysis tracks how information (like variable definitions and uses) "flows" between basic blocks.
*   **Examples:**
    *   **Global Common Subexpression Elimination:** Identifying and eliminating redundant expressions across multiple basic blocks.
    *   **Loop-Invariant Code Motion:** Moving a calculation that is constant within a loop to outside the loop.
    *   **Copy Propagation:** Replacing a variable with its value across basic block boundaries.
    *   **Dead Code Elimination:** Removing code that is unreachable or whose result is never used.

In essence, "local" means within a basic block, while "global" means across basic blocks, requiring an understanding of the overall program flow.

---

### (b) What are the 3 areas of code optimization? Explain each one in detail.

Code optimization can be broadly categorized into three main areas based on the techniques applied and their impact on the code. These areas are **Peephole Optimization**, **Local Optimization**, and **Global (or Intraprocedural) Optimization**.

#### 1. Local Optimization

Local optimization focuses on transforming individual basic blocks in isolation. A basic block is a straight-line sequence of code without any jumps into or out of the middle. Because the control flow is simple, these optimizations are relatively easy and safe to apply.

*   **Goal:** To improve the efficiency of computations within a single block.
*   **Techniques:**
    *   **Common Subexpression Elimination (Local):** As described previously, this involves representing the basic block as a Directed Acyclic Graph (DAG) to find and eliminate redundant computations. For example, in `a=b+c; x=b+c;`, the second `b+c` is redundant.
    *   **Constant Folding and Propagation:** Evaluating constant expressions at compile time (`x=2*50` becomes `x=100`) and propagating these constant values (`y=x+1` becomes `y=101`).
    *   **Algebraic Simplification and Strength Reduction:** Replacing expensive operations with cheaper ones. For instance, replacing `x*2` with `x+x` or `x<<1` (strength reduction), or simplifying `x-0` to `x` (algebraic simplification).
    *   **Dead Code Elimination (Local):** Removing instructions within a block that compute a value that is never used.

#### 2. Global Optimization (Intraprocedural)

Global optimization operates on the scale of an entire function or procedure. It analyzes the program's **control-flow graph (CFG)**, which represents all possible paths of execution between basic blocks. This requires a more complex form of analysis called **data-flow analysis**.

*   **Goal:** To find optimization opportunities that span across multiple basic blocks.
*   **Techniques:**
    *   **Global Common Subexpression Elimination:** Finding and eliminating redundant computations across the entire function, not just within a single block.
    *   **Loop Optimizations:** This is a critical subset of global optimization as programs spend most of their time in loops.
        *   **Loop-Invariant Code Motion:** Moving calculations that don't change inside the loop to outside the loop.
        *   **Induction Variable Elimination:** Simplifying or eliminating variables that are updated in a simple, linear fashion on each loop iteration.
        *   **Loop Unrolling:** Replicating the loop body multiple times to reduce loop overhead and expose more instruction-level parallelism.
    *   **Global Copy Propagation:** Propagating copy statements (e.g., `x=y`) across basic block boundaries.
    *   **Global Dead Code Elimination:** Removing entire basic blocks that are unreachable or variables whose values are never used across the function.

#### 3. Peephole Optimization

Peephole optimization is a simple but effective technique that is typically performed late in the compilation process, often on the target machine code or a very low-level intermediate representation. It involves examining a small, sliding "window" of adjacent instructions (the "peephole") and replacing these instructions with a shorter or faster sequence.

*   **Goal:** To catch inefficiencies introduced during code generation and to exploit specific features of the target machine's instruction set.
*   **Techniques:**
    *   **Redundant Instruction Elimination:** Removing redundant loads and stores. For example, if a value is stored to memory and then immediately loaded back into the same register (`STORE R0, addr; LOAD addr, R0`), the load is unnecessary.
    *   **Flow-of-Control Optimization:** Simplifying jumps. A jump to an unconditional jump can be replaced by a single jump to the final destination (`JUMP L1; ... L1: JUMP L2` becomes `JUMP L2; ... L1: JUMP L2`).
    *   **Algebraic Simplification:** Similar to local optimization, but at the instruction level (e.g., `ADD R0, #0` can be removed).
    *   **Use of Machine Idioms:** Replacing a sequence of instructions with a single, more powerful machine instruction that has the same effect (e.g., using an auto-increment addressing mode instead of a separate `ADD` instruction). 