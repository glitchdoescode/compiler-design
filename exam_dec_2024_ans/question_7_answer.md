# Answer to Question 7

---

### (a) Write the differences between Synthesized and Inherited attributes.

Synthesized and inherited attributes are fundamental concepts in Syntax-Directed Translation (SDT) used to associate semantic information with grammar symbols. They define how meaning (or values) flows through a parse tree. The primary differences lie in how their values are computed and the direction of that information flow.

| Feature               | Synthesized Attributes                                                                        | Inherited Attributes                                                                                                       |
|:----------------------|:----------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|
| **Direction of Flow** | Information flows **up** the parse tree, from children to the parent.                           | Information flows **down** or **sideways** across the parse tree, from parent to children, or from a left sibling to a right sibling. |
| **Computation**       | The value of a synthesized attribute at a node is computed from the attributes of its **children**. | The value of an inherited attribute at a node is computed from attributes of its **parent** and/or its **siblings**.        |
| **Association**       | Associated with the non-terminal on the **left-hand side (LHS)** of a production rule.          | Associated with symbols on the **right-hand side (RHS)** of a production rule.                                               |
| **Analogy**           | Can be thought of as the "return values" of the parsing functions for non-terminals.            | Can be thought of as the "parameters" passed to the parsing functions for non-terminals.                                     |
| **Evaluation Order**  | Can be evaluated by a **post-order traversal** of the parse tree (bottom-up).                   | Require a more complex evaluation order, typically a depth-first, left-to-right traversal.                                   |
| **Use Case Example**  | Calculating the value of an arithmetic expression, where the result is passed up the tree.        | Propagating the data type of a variable declaration down to all identifiers in a list (e.g., `int a, b, c;`).             |
| **Grammar Type**      | Used exclusively in **S-attributed grammars**.                                                  | Used in **L-attributed grammars** (which can also include synthesized attributes).                                             |
| **Parser Suitability**| Well-suited for evaluation during **bottom-up parsing** (e.g., LR parsers).                     | Well-suited for evaluation during **top-down parsing** (e.g., LL parsers).                                                     |

In summary, the key distinction is that synthesized attributes pass information up the tree, consolidating data towards the root, while inherited attributes pass information down and across the tree, distributing contextual information from the top or from the left.

---

### (b) Explain the common subexpression elimination, copy propagation, and transformation for moving loop invariant computations in detail.

These are three important machine-independent code optimization techniques performed by compilers on the intermediate code to improve the efficiency of the final program.

#### i) Common Subexpression Elimination (CSE)

A **common subexpression** is an expression that appears multiple times in the code and computes the same value each time. CSE aims to eliminate the redundant computations.

*   **How it works:** The compiler identifies identical expressions. It computes the result of the expression once, stores it in a temporary variable, and then replaces all subsequent occurrences of that expression with a reference to the temporary variable.
*   **Local vs. Global CSE:**
    *   **Local CSE** operates within a single basic block. The DAG construction algorithm is a classic way to detect and eliminate local common subexpressions.
    *   **Global CSE** operates across an entire procedure or function, requiring data-flow analysis to track expression availability across basic block boundaries.
*   **Example:**
    ```c
    // Before CSE
    c = a * b + 5;
    d = a * b - e;
    ```
    ```c
    // After CSE
    temp = a * b;
    c = temp + 5;
    d = temp - e;
    ```
    This avoids re-calculating `a * b`.

#### ii) Copy Propagation

**Copy propagation** is an optimization technique that replaces occurrences of a variable with the value it was assigned from another variable. It is often used after CSE or other optimizations introduce copy statements (e.g., `x = y`).

*   **How it works:** After a copy statement `x = y`, the compiler replaces later uses of `x` with `y`, as long as neither `x` nor `y` is reassigned between the copy statement and the use of `x`.
*   **Goal:** The primary goal is to eliminate the copy statement itself, which becomes "dead code" if `x` is no longer used. This reduces the number of temporary variables and can expose further optimization opportunities.
*   **Example:**
    ```c
    // Before Copy Propagation
    temp = a * b;
    c = temp + 5;
    d = temp - e;
    ```
    ```c
    // After Copy Propagation
    // (Assuming 'c' is not used later)
    c = (a * b) + 5; // This seems counter-intuitive, but let's take a better example
    ```
    Let's refine the example to better illustrate the point:
    ```c
    // Code with a copy statement
    x = y;
    z = 10 + x;
    ```
    ```c
    // After Copy Propagation
    x = y;
    z = 10 + y;
    ```
    Now, if `x` is not used anywhere else, the assignment `x = y` becomes dead code and can be removed by dead code elimination.

#### iii) Loop-Invariant Code Motion

A **loop-invariant computation** is an expression within a loop whose value does not change from one iteration of the loop to the next. Such computations can be moved outside the loop to be executed only once, saving significant time if the loop runs many times.

*   **How it works:**
    1.  **Detection:** The compiler uses data-flow analysis to identify expressions inside a loop whose operands are all defined outside the loop (or are constants).
    2.  **Code Motion:** The invariant computation is moved to the "preheader" of the loop—a block of code that is executed just before the loop begins. The result is stored in a new temporary variable.
    3.  **Replacement:** All occurrences of the invariant computation inside the loop are replaced with the temporary variable.
*   **Example:**
    ```c
    // Before Loop-Invariant Code Motion
    while (i < limit - 2) {
        a[i] = x * y; // x*y is invariant
        i = i + 1;
    }
    ```
    ```c
    // After Loop-Invariant Code Motion
    temp = x * y; // Invariant computation moved out
    while (i < limit - 2) {
        a[i] = temp;
        i = i + 1;
    }
    ```
    This avoids recomputing `x * y` in every single iteration of the while loop, leading to a significant performance improvement. 