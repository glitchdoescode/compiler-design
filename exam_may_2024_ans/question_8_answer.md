# Answer to Question 8

---

### Write brief notes: (any three)

Here are brief notes on all four topics.

#### a) Peephole Optimization

Peephole optimization is a machine-dependent optimization technique performed late in the compilation process, either on the intermediate code or, more commonly, on the final target code. It works by examining a small, sliding window of adjacent instructions (the "peephole") and replacing them with a shorter or faster sequence that produces the same result.

*   **Goal:** To improve performance by eliminating redundant instructions and using more efficient machine-specific idioms.
*   **Key Techniques:**
    *   **Redundant Instruction Elimination:** Removing unnecessary loads and stores. For example, a `STORE R, x` followed by a `LOAD x, R` is redundant if `x` is not modified in between; the `LOAD` can be deleted.
    *   **Flow-of-Control Optimization:** Simplifying jumps. A jump to an unconditional jump can be replaced by a single jump to the final destination (e.g., `JMP L1; ... L1: JMP L2` becomes `JMP L2`). Unreachable code after an unconditional jump can also be removed.
    *   **Algebraic Simplification:** Applying algebraic identities at the machine code level. For example, `ADD R, #0` (adding zero) or `MUL R, #1` (multiplying by one) can be eliminated.
    *   **Strength Reduction:** Replacing expensive instructions with cheaper ones, like replacing `MUL R, #2` with a faster left-shift instruction `LSL R, #1`.
*   **Significance:** It's a simple yet powerful technique for cleaning up inefficiencies that may have been introduced during the code generation phase.

---

#### b) Global data flow equations

Global data-flow analysis is a requirement for performing global optimizations. It involves collecting information about how data is transmitted across the boundaries of basic blocks in a control-flow graph (CFG). This information is computed by setting up and solving systems of **data-flow equations**.

*   **Structure:** For each basic block `B`, we have equations that define `OUT[B]` in terms of `IN[B]` (or vice-versa), based on the analysis being performed.
*   `IN[B]` = Information flowing into the start of block `B`.
*   `OUT[B]` = Information flowing out of the end of block `B`.

The exact form of the equations depends on the specific data-flow problem:

**1. Reaching Definitions:** A definition `d` of a variable `v` *reaches* a point `p` if there is a path from `d` to `p` with no other definition of `v` along the path.
*   **Use:** Detecting possible uninitialized variables.
*   **Equations (Forward Analysis):**
    *   `OUT[B] = gen[B] ∪ (IN[B] - kill[B])`
    *   `IN[B] = ∪ (OUT[P])` for all predecessors `P` of `B`.
    *   `gen[B]` = definitions created within `B`.
    *   `kill[B]` = definitions from elsewhere that are overridden in `B`.

**2. Live-Variable Analysis:** A variable `v` is *live* at a point `p` if its value may be used along some path starting from `p`.
*   **Use:** Register allocation, dead code elimination.
*   **Equations (Backward Analysis):**
    *   `IN[B] = use[B] ∪ (OUT[B] - def[B])`
    *   `OUT[B] = ∪ (IN[S])` for all successors `S` of `B`.
    *   `use[B]` = variables used in `B` before being defined.
    *   `def[B]` = variables defined in `B` before being used.

These sets of equations are solved iteratively until the `IN` and `OUT` sets for all blocks converge.

---

#### c) Dynamic Storage Allocation

Dynamic storage allocation refers to the management of memory at runtime, where the size and number of data objects are not known at compile time. This is handled in a region of memory called the **heap**. This strategy provides the most flexibility but comes with performance overhead.

*   **Mechanism:**
    1.  **Allocation:** The program requests a block of memory of a specific size from the runtime system (e.g., via `malloc()` in C or `new` in C++). The memory manager finds a suitable free block in the heap and returns a pointer to it.
    2.  **Deallocation:** The memory remains allocated until it is explicitly released by the programmer (e.g., `free(ptr)`) or automatically reclaimed by a garbage collector.
*   **Key Issues:**
    *   **Overhead:** The process of searching for and managing free blocks is slower than the simple pointer arithmetic of stack allocation.
    *   **Fragmentation:** Over time, the heap can become fragmented into small, non-contiguous free blocks. **External fragmentation** occurs when there is enough total free memory but no single block is large enough to satisfy a request.
    *   **Memory Leaks:** In languages without garbage collection, if a programmer forgets to free memory that is no longer needed, the program's memory usage will continuously grow, which can lead to it crashing. This is a common and serious bug.
*   **Use Cases:** Essential for data structures that can change size dynamically, such as linked lists, trees, and hash tables, and for objects whose lifetime must extend beyond the function call that created them.

---

#### d) Error Detection and Recovery

A key role of a compiler is to detect errors in the source program and report them to the user in a clear and useful manner. The compiler should also attempt to **recover** from errors so it can continue processing the rest of the program to find more errors in a single run.

*   **Error Types:**
    *   **Lexical Errors:** Invalid characters or sequences (e.g., `_@#`). Handled by the scanner.
    *   **Syntax Errors:** The code violates the grammatical rules of the language (e.g., a missing semicolon or parenthesis, `while x > 5 do...`). Detected by the parser.
    *   **Semantic Errors:** The code is grammatically correct but logically nonsensical, violating the language's meaning rules (e.g., adding a string to an integer, calling a function with the wrong number of arguments). Detected by the semantic analyzer.
    *   **Logical Errors:** The program compiles and runs but produces incorrect output. The compiler cannot detect these.

*   **Recovery Strategies (primarily for syntax errors):**
    *   **Panic Mode:** This is the simplest method. When the parser encounters an error, it discards input tokens one at a time until it finds a "synchronizing" token (e.g., `;` or `}`). While easy to implement, it can skip large sections of the input.
    *   **Phrase-Level Recovery:** The parser performs a local correction on the remaining input, replacing a prefix of it with a string that allows the parse to continue. For example, it might insert a missing semicolon. This is more difficult to implement.
    *   **Error Productions:** The grammar is augmented with special productions that match common errors. For instance, `stmt -> if (expr) stmt; | error;`. When the `error` production is used, the parser can report the error but recover and continue.
    *   **Global Correction:** In theory, a compiler could find the minimal sequence of changes to make a program syntactically correct, but this is computationally expensive and not practical. 