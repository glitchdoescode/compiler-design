# Answer to Question 5

---

### (a) Explain various intermediate code generation techniques in brief.

Intermediate Code (IC) is a representation of a source program that is machine-independent and is generated during the intermediate phases of a compiler. It acts as a bridge between the front-end (parsing, semantic analysis) and the back-end (optimization, code generation). Using an IC makes retargeting the compiler to a new machine easier.

Several forms of intermediate code are used, each with its own advantages:

#### 1. Abstract Syntax Tree (AST)
An AST is a condensed, tree-based representation of the source code's structure. It is "abstract" because it omits details from the parse tree that are not essential for semantic analysis and code generation, such as punctuation (parentheses, semicolons).
*   **Structure:** Operators become interior nodes, and their operands are their children.
*   **Usage:** It's a high-level IC, useful for syntax-directed translations and some high-level optimizations.

#### 2. Postfix Notation (Reverse Polish Notation)
In postfix notation, the operator comes after its operands. For example, the infix expression `a + b` becomes `a b +`.
*   **Structure:** A linear, one-dimensional representation.
*   **Usage:** It's useful for stack-based virtual machines. The evaluation is straightforward: scan the notation, push operands onto a stack, and when an operator is found, pop the required number of operands, perform the operation, and push the result back.
*   **Example:** `(a-b)*(c+d)` becomes `a b - c d + *`.

#### 3. Three-Address Code (TAC)
TAC is one of the most popular intermediate representations. Each instruction has at most three addresses (one for the result, two for the operands). An address can be a name, a constant, or a compiler-generated temporary.
*   **Structure:** A sequence of simple instructions, resembling an assembly language.
*   **General Form:** `result = operand1 op operand2`
*   **Types of TAC Instructions:**
    *   **Assignment:** `x = y op z` or `x = op y`
    *   **Copy:** `x = y`
    *   **Unconditional Jump:** `goto L`
    *   **Conditional Jump:** `if x relop y goto L`
    *   **Procedure Calls:** `param x`, `call p, n`, and `y = call p, n`
    *   **Indexed Assignment:** `x = y[i]`, `x[i] = y`
*   **Advantages:** It is close to machine instructions, easy to generate, and allows for straightforward application of many code optimization techniques.

#### 4. Quadruples
This is a record-based representation of three-address code. A quadruple is a structure with four fields: `op`, `arg1`, `arg2`, and `result`.
*   **Structure:** A table or array of records.
*   **Example:** `x = y + z` is represented as `(+, y, z, x)`.
*   **Advantages:** The instruction order can be easily rearranged for optimization because the result is named explicitly.

#### 5. Triples
This is another record-based representation of TAC, but it uses pointers to refer to other instructions instead of explicit temporary names. A triple has three fields: `op`, `arg1`, and `arg2`.
*   **Structure:** A table of records where `arg1` and `arg2` can be pointers to other entries in the table.
*   **Example:** For `a = b * c; x = a + d`, the triples would be:
    *   `(0): (*, b, c)`
    *   `(1): (+, (0), d)`
*   **Disadvantage:** Reordering instructions is difficult because it requires updating all pointers that refer to the moved instruction, making optimization more complex than with quadruples.

---

### (b) Compute E.value for the root of the parse tree for the expression `2 & 3 & 5 # 6 & 4`.

**Grammar and Rules:**
1.  `E -> E₁ # T      { E.value = E₁.value * T.value }`
2.  `T -> T₁ & F      { T.value = T₁.value + F.value }`
3.  `T -> F           { T.value = F.value }`
4.  `F -> num         { F.value = num.value }`

The operators `#` and `&` have different precedence and associativity. The grammar indicates that `#` is left-associative and has lower precedence than `&`. `&` is also left-associative.

Let's parse the expression `2 & 3 & 5 # 6 & 4` and evaluate the attributes using the semantic rules. The parsing will group the expression as `(2 & 3 & 5) # (6 & 4)`.

We will evaluate this by building and annotating the parse tree from the bottom up.

**Step 1: Evaluate `(2 & 3 & 5)`**
This corresponds to the `E₁` part of the top-level `E -> E₁ # T` production.
*   The sub-expression is parsed as `T -> T₁ & F`.
*   First, `2 & 3` is evaluated:
    *   `T₁` is `2` (derived from `T -> F -> num`). `T₁.value = 2`.
    *   `F` is `3` (derived from `F -> num`). `F.value = 3`.
    *   Using rule 2 (`T -> T₁ & F`), the result is `T.value = 2 + 3 = 5`.
*   Next, `(2 & 3) & 5` is evaluated:
    *   The `T` from the previous step becomes the new `T₁`. `T₁.value = 5`.
    *   `F` is `5` (derived from `F -> num`). `F.value = 5`.
    *   Using rule 2 again, the result is `T.value = 5 + 5 = 10`.
*   So, the value for the entire sub-expression `2 & 3 & 5` is **10**. This is `E₁.value`.

**Step 2: Evaluate `(6 & 4)`**
This corresponds to the `T` part of the top-level `E -> E₁ # T` production.
*   This is parsed using `T -> T₁ & F`.
*   `T₁` is `6` (derived from `T -> F -> num`). `T₁.value = 6`.
*   `F` is `4` (derived from `F -> num`). `F.value = 4`.
*   Using rule 2 (`T -> T₁ & F`), the result is `T.value = 6 + 4 = 10`.
*   The value for the sub-expression `6 & 4` is **10**. This is `T.value` for the top-level rule.

**Step 3: Evaluate the final expression `E₁ # T`**
*   We have `E₁.value = 10` (from Step 1).
*   We have `T.value = 10` (from Step 2).
*   Using rule 1 (`E -> E₁ # T`), we get `E.value = E₁.value * T.value`.
*   `E.value = 10 * 10 = 100`.

**Final Answer:**
The value of `E.value` for the root of the parse tree for the expression `2 & 3 & 5 # 6 & 4` is **100**. 