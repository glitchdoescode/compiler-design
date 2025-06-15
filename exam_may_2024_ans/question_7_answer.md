# Answer to Question 7

---

### a) Represent the following in flow graph: `i=1; sum=0; while(i<=10){sum=sum+i; i=i+1;}`.

A **control-flow graph (CFG)** is a directed graph used to represent the control flow of a program. Each node in the graph represents a **basic block**, which is a straight-line piece of code without any jumps or jump targets in the middle. The directed edges represent jumps in the control flow.

#### Step 1: Generate Three-Address Code (TAC)
First, we convert the given code into a sequence of three-address instructions.
```
1.  i = 1
2.  sum = 0
L1:
3.  if i > 10 goto L2
4.  sum = sum + i
5.  i = i + 1
6.  goto L1
L2:
// ... code after the loop
```

#### Step 2: Identify the Basic Blocks
Next, we partition the TAC into basic blocks. A new block starts with a labeled instruction (a jump target) or after a jump instruction. Let's define the blocks for a clear graph:

*   **Block B1 (Initialization):**
    ```
    i = 1
    sum = 0
    ```
    (This block unconditionally flows to B2).

*   **Block B2 (Loop Condition):**
    ```
    L1: if i > 10 goto L2  // In CFG terms, goto B4
    ```
    (This block has two possible exits).

*   **Block B3 (Loop Body):**
    ```
    sum = sum + i
    i = i + 1
    goto L1 // In CFG terms, goto B2
    ```
    (This block has one unconditional exit).

*   **Block B4 (After Loop):**
    This block is the target `L2` and contains any code that follows the loop. It is the exit block.
    ```
    L2: (exit)
    ```

#### Step 3: Draw the Flow Graph
We now draw the nodes for each basic block and connect them with directed edges representing the flow of control.

*   Execution starts at `B1`.
*   After `B1`, control unconditionally flows to `B2`.
*   `B2` is the loop's entry point and condition check.
    *   If the condition `i <= 10` is true (the `if i > 10` test is false), control flows to `B3`.
    *   If the condition is false (`i > 10`), control flows to the exit block `B4`.
*   `B3` contains the loop's body. After executing, it unconditionally jumps back to `B2` to re-evaluate the condition.

**The resulting flow graph is:**

```
      +-----------+
      |    B1     |
      |   i = 1   |
      |  sum = 0  |
      +-----------+
            |
            v
      +-----------+
      |    B2     |
      | if i > 10 |
      | goto B4   |
      +-----------+
(false)|       |(true)
        |       |
        v       v
      +-----------+   +-----------+
      |    B3     |   |    B4     |
      | sum=sum+i |   |  (Exit)   |
      |  i = i+1  |   +-----------+
      | goto B2   |
      +-----------+
          ^
          |
          '----------
```

---

### b) What are the different sources of optimization of basic blocks?

Optimizing a basic block is also known as **local optimization**. The goal is to transform the code within a single basic block to make it faster or smaller without changing its semantics. The primary sources or techniques for this optimization are derived from analyzing the structure and data flow within the block, often by using a **Directed Acyclic Graph (DAG)**.

The main sources of optimization are:

#### 1. Common Subexpression Elimination
This is one of the most significant sources of optimization. A common subexpression is an expression that was previously computed and whose operands have not changed since the last computation.
*   **How it works:** By building a DAG for the basic block, identical subtrees represent common subexpressions. The compiler can compute the expression once, save the result in a temporary variable (or register), and reuse that result for all subsequent occurrences within the block.
*   **Example:**
    ```
    x = a + b;
    ...
    y = a + b;
    ```
    This can be optimized to:
    ```
    t1 = a + b;
    x = t1;
    ...
    y = t1;
    ```

#### 2. Dead Code Elimination
Dead code is code that computes a value that is never used anywhere in the program.
*   **How it works:** In the context of a basic block, if a variable is assigned a value but is never read again before being reassigned (or before the block ends and the variable is not live), the assignment instruction is "dead" and can be removed. The DAG helps identify nodes whose computed values are not used by any subsequent instruction or are not associated with a live-out variable.
*   **Example:** If `x` is not used later, `x = y + 1` is dead code.

#### 3. Algebraic Identities and Simplification
The compiler can use algebraic laws to simplify expressions.
*   **How it works:** The compiler replaces complex expressions with simpler, equivalent ones.
*   **Examples:**
    *   `x = y + 0` can be replaced with `x = y`.
    *   `x = y * 1` can be replaced with `x = y`.
    *   `x = y - y` can be replaced with `x = 0`.

#### 4. Strength Reduction
This technique involves replacing an expensive (high-strength) operation with a cheaper (low-strength) one.
*   **How it works:** Certain operations are computationally more expensive than others. For example, multiplication is often slower than addition, and exponentiation is slower than multiplication.
*   **Examples:**
    *   `x = y * 2` can be replaced with `x = y + y` or `x = y << 1` (a bit shift).
    *   `x = y ^ 2` (y squared) can be replaced with `x = y * y`.

#### 5. Constant Folding and Propagation
*   **Constant Folding:** If all operands in an expression are constants, the expression can be evaluated at compile time.
    *   **Example:** `x = 2 * 5 * 10;` becomes `x = 100;`.
*   **Constant Propagation:** If a variable is assigned a constant value, the compiler can replace subsequent uses of that variable with the constant itself. This can, in turn, create new opportunities for constant folding.
    *   **Example:**
        ```
        x = 5;
        y = x + 10;
        ```
        becomes `y = 5 + 10;`, which is then folded into `y = 15;`.

</rewritten_file>