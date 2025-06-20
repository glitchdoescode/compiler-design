# Answer to Question 1

---

### Discuss the analysis and synthesis model of compilation. Explain each phase by using the input: `a = (b + c) * (b + c) * 2`.

The compilation process is a sequence of phases that transform a high-level source program into a low-level target program (e.g., machine code). This process is broadly divided into two main parts: the **Analysis (Front-end)** and the **Synthesis (Back-end)**.

*   **Analysis (Front-end):** This part breaks down the source program into its constituent pieces and creates an intermediate representation (IR). It includes lexical, syntax, and semantic analysis, culminating in the generation of the IR. It also manages the symbol table. The front-end is largely independent of the target machine.
*   **Synthesis (Back-end):** This part takes the intermediate representation from the front-end and constructs the desired target program. It includes code optimization and code generation. The back-end is dependent on the target machine's architecture.

Let's trace the compilation of the input `a = (b + c) * (b + c) * 2` through each phase.

---

### Analysis Phases (Front-end)

#### 1. Lexical Analysis (Scanning)
The lexical analyzer reads the stream of characters in the source program and groups them into meaningful sequences called **lexemes**. For each lexeme, it produces a **token** of the form `<token-name, attribute-value>`.

**Input:** `a = (b + c) * (b + c) * 2`
**Output (Tokens):**
```
<id, 1> <assign_op, => <lparen, (> <id, 2> <add_op, +> <id, 3> <rparen, )> <mul_op, *> <lparen, (> <id, 2> <add_op, +> <id, 3> <rparen, )> <mul_op, *> <num, 2>
```
*   The symbol table is populated with entries for identifiers `a`, `b`, and `c`. The numbers 1, 2, 3 are pointers to the symbol table entries.

#### 2. Syntax Analysis (Parsing)
The parser takes the token stream from the lexical analyzer and verifies that it can be generated by the grammar for the source language. It constructs a tree-like intermediate representation, such as a **parse tree** or **syntax tree**, that depicts the grammatical structure of the token stream.

**Input:** Token stream from the previous phase.
**Output:** A parse tree. The tree would represent the structure `assign -> id = expr`, `expr -> expr * term`, etc. A more condensed Abstract Syntax Tree (AST) for the expression part would look like this:

```
        *
       / \
      *   2
     / \
    +   +
   / \ / \
  b  c b  c
```
*(The assignment to `a` would be the root of the full AST)*

#### 3. Semantic Analysis
The semantic analyzer uses the syntax tree and the symbol table to check the source program for semantic consistency with the language definition. It performs type checking and gathers type information for the subsequent code-generation phase.

**Input:** Parse Tree / AST.
**Output:** An annotated AST.
*   **Type Checking:** It checks if the operands of `+` and `*` are compatible (e.g., both numeric). Let's assume `b` and `c` are floats. The analyzer infers that the result of `b+c` is a float. The literal `2` is an integer, so the analyzer may insert a type conversion (coercion) to turn `2` into `2.0` (float) before the final multiplication.
*   **Symbol Table:** The types of all identifiers are checked and recorded.

#### 4. Intermediate Code Generation
After semantic analysis, the compiler generates an explicit low-level or machine-like intermediate representation. **Three-Address Code (TAC)** is a common form. This representation is easy to produce and easy to translate into the target program.

**Input:** Annotated AST.
**Output (Three-Address Code):**
```
t1 = b + c
t2 = b + c
t3 = t1 * t2
t4 = int_to_float(2)
t5 = t3 * t4
a = t5
```

---

### Synthesis Phases (Back-end)

#### 5. Code Optimization
This phase attempts to improve the intermediate code to result in a better target program—one that is faster, shorter, or uses less power.

**Input:** Intermediate Code.
**Output (Optimized Intermediate Code):**
*   **Common Subexpression Elimination:** The optimizer notices that `t1` and `t2` are the same (`b+c`). It eliminates the redundant computation.
```
t1 = b + c
// t2 = b + c is removed
t3 = t1 * t1 // Use t1 again
t4 = int_to_float(2)
t5 = t3 * t4
a = t5
```

#### 6. Code Generation
The code generator takes the optimized intermediate code as input and maps it to the target machine language. It assigns registers to hold variables and translates the intermediate instructions into a sequence of machine instructions.

**Input:** Optimized Intermediate Code.
**Output (Target Assembly Code - example):**
```assembly
; Assume b, c, and a are in memory
; R1, R2 are floating-point registers
LOAD_F R1, b
LOAD_F R2, c
ADD_F  R1, R1, R2   ; R1 = b + c (this is t1)
MUL_F  R1, R1, R1   ; R1 = t1 * t1 (this is t3)
LOAD_I R2, #2       ; Load immediate integer 2
I_TO_F R2, R2       ; Convert integer in R2 to float (t4)
MUL_F  R1, R1, R2   ; R1 = t3 * t4 (this is t5)
STORE_F a, R1       ; Store the final result in a
```
This final phase produces the executable code. 