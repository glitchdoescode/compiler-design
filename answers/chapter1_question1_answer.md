# Compiler Structure and Phases

A compiler is a crucial software tool that translates source code written in a high-level programming language into a low-level language (like machine code or assembly language) that a computer's processor can execute. This translation process is complex and is systematically broken down into a series of steps known as phases.

## Structure of a Compiler: Analysis and Synthesis Model

The compilation process is broadly divided into two major parts:

1.  **Analysis Phase (Front-End):** This part of the compiler breaks down the source program into constituent pieces and imposes a grammatical structure on them. It then uses this structure to create an intermediate representation of the source program. The analysis phase also collects information about the source program and stores it in a data structure called a **symbol table**. The front-end is largely independent of the target machine.
    The primary phases involved in the front-end are:
    *   Lexical Analysis
    *   Syntax Analysis
    *   Semantic Analysis
    *   Intermediate Code Generation (often considered a bridge)

2.  **Synthesis Phase (Back-End):** This part constructs the desired target program from the intermediate representation and the information in the symbol table. The back-end is often dependent on the target machine architecture.
    The primary phases involved in the back-end are:
    *   Code Optimization (can also be machine-independent in parts)
    *   Target Code Generation

The **intermediate code** acts as an interface between the analysis and synthesis phases, allowing for a modular design where the front-end can be combined with different back-ends for various target machines.

---

## Phases of a Compiler

The compilation process typically involves the following six phases:

### 1. Lexical Analysis (Scanning)

*   **Function:** The lexical analyzer, or scanner, reads the stream of characters making up the source program and groups the characters into meaningful sequences called **lexemes**. For each lexeme, the lexical analyzer produces a **token** of the form `<token-name, attribute-value>`. The token-name is an abstract symbol that is used during syntax analysis, and the attribute-value points to an entry in the symbol table for this token.
    It also strips out comments and whitespace.

*   **Output for `a = (b + c) * (b + c) * 2`:**
    ```
    <id, pointer to 'a' in symbol table>
    <assign_op>
    <left_paren>
    <id, pointer to 'b' in symbol table>
    <add_op>
    <id, pointer to 'c' in symbol table>
    <right_paren>
    <mult_op>
    <left_paren>
    <id, pointer to 'b' in symbol table>
    <add_op>
    <id, pointer to 'c' in symbol table>
    <right_paren>
    <mult_op>
    <number, 2>
    ```
    *(Assuming `a, b, c` are identifiers, `=` is assignment, `+`, `*` are operators, `(`, `)` are parentheses, and `2` is a number. The attribute value would typically be a pointer to the symbol table entry for identifiers or the actual value for numbers.)*

*   **Output for `id1 = id2 + id3 * 50`:**
    ```
    <id, pointer to 'id1' in symbol table>
    <assign_op>
    <id, pointer to 'id2' in symbol table>
    <add_op>
    <id, pointer to 'id3' in symbol table>
    <mult_op>
    <number, 50>
    ```

### 2. Syntax Analysis (Parsing)

*   **Function:** The syntax analyzer, or parser, uses the tokens produced by the lexical analyzer to create a tree-like intermediate representation that depicts the grammatical structure of the token stream. A typical representation is an **Abstract Syntax Tree (AST)**, where each internal node represents an operation and the children of the node represent the arguments of the operation. This phase checks if the arrangement of tokens is grammatically correct according to the rules of the source language.

*   **Output for `a = (b + c) * (b + c) * 2` (AST):**
    ```
          assign (=)
          /    \
        id(a)   mult (*)
               /      \
            mult(*)  number(2)
           /     \
        add(+)   add(+)
       /  \     /  \
    id(b) id(c) id(b) id(c)
    ```

*   **Output for `id1 = id2 + id3 * 50` (AST):**
    ```
          assign (=)
          /    \
        id(id1) add(+)
               /    \
            id(id2) mult(*)
                   /     \
                id(id3) number(50)
    ```

### 3. Semantic Analysis

*   **Function:** The semantic analyzer uses the syntax tree and the information in the symbol table to check the source program for semantic consistency with the language definition. It gathers type information and saves it in either the syntax tree or the symbol table, for subsequent use during intermediate-code generation. An important part of semantic analysis is **type checking**, where the compiler checks that each operator has matching operands. For example, many programming language definitions require an array index to be an integer; the compiler must report an error if a floating-point number is used to index an array. It also handles type coercions, where necessary and allowed (e.g., converting an integer to a float for an operation).

*   **Output for `a = (b + c) * (b + c) * 2`:**
    The AST is annotated with type information. Assuming `a, b, c` are declared as, say, `int` or `float`.
    *   Types of `b` and `c` are checked.
    *   The result type of `(b+c)` is determined.
    *   The types for the multiplications are checked.
    *   The type of the final expression `(b+c)*(b+c)*2` is checked for compatibility with the type of `a`.
    *   If all types are compatible (possibly after coercions, e.g., `2` is `int` but might be promoted if `b` or `c` are `float`), the annotated AST is passed on. Otherwise, semantic errors are reported.

*   **Output for `id1 = id2 + id3 * 50`:**
    Similarly, the AST is type-checked.
    *   Assuming `id1, id2, id3` are numeric types and `50` is an integer.
    *   Type of `id3 * 50` is determined.
    *   Type of `id2 + (result of id3 * 50)` is determined.
    *   This final type is checked for compatibility with `id1`.
    *   The annotated AST is produced, or errors are reported. For example, if `id3` was a string, `id3 * 50` might be a type error or a string repetition depending on the language. If numeric, `int(50)` might be converted to `float` if `id3` is `float`.

### 4. Intermediate Code Generation

*   **Function:** After syntax and semantic analysis, some compilers generate an explicit low-level or machine-like intermediate representation, which we can think of as a program for an abstract machine. This intermediate representation should be easy to produce and easy to translate into the target machine's language. A common form is **Three-Address Code (TAC)**, which consists of a sequence of instructions, each of which has at most three operands.

*   **Output for `a = (b + c) * (b + c) * 2` (Three-Address Code):**
    ```
    t1 = b + c
    t2 = b + c  // Or, if simple common subexpression is done: t2 = t1
    t3 = t1 * t2
    t4 = t3 * 2
    a = t4
    ```

*   **Output for `id1 = id2 + id3 * 50` (Three-Address Code):**
    ```
    t1 = id3 * 50
    t2 = id2 + t1
    id1 = t2
    ```

### 5. Code Optimization

*   **Function:** This phase attempts to improve the intermediate code to result in better target code. "Better" usually means faster, shorter code, or code that consumes less power. Various techniques are applied, such as:
    *   **Common Subexpression Elimination:** Avoid recomputing expressions whose value is already available.
    *   **Constant Folding:** Evaluate constant expressions at compile time.
    *   **Dead Code Elimination:** Remove code that is unreachable or whose result is unused.
    *   **Strength Reduction:** Replace expensive operations with cheaper ones (e.g., multiplication by 2 with a left shift).
    *   **Loop Optimizations:** Such as code motion (moving loop-invariant computations out of loops).

*   **Output for `a = (b + c) * (b + c) * 2` (Optimized TAC):**
    Assuming `t1 = b + c` from previous TAC:
    ```
    t1 = b + c
    // t2 = b + c  -- This is a common subexpression, can reuse t1
    t3 = t1 * t1  // Replaced t2 with t1
    t5 = t3 * 2   // Or t5 = t3 << 1 (strength reduction if integer)
    a = t5
    ```

*   **Output for `id1 = id2 + id3 * 50` (Optimized TAC):**
    In this specific case, without more context (e.g., if `id3` or `id2` are constants, or if this is in a loop), major optimizations might not be apparent.
    ```
    t1 = id3 * 50
    t2 = id2 + t1
    id1 = t2
    ```
    (If `id3` was a known constant, say `id3 = 4`, then `t1 = 4 * 50` could be folded to `t1 = 200`.)

### 6. Target Code Generation

*   **Function:** This is the final phase of compilation. It takes the (optimized) intermediate representation as input and maps it to the target machine language. This involves:
    *   **Instruction Selection:** Choosing appropriate machine instructions for each IR operation.
    *   **Register Allocation:** Deciding which variables and temporary values will be stored in machine registers at each point in the program.
    *   **Instruction Ordering:** Arranging instructions in a way that optimizes performance.
    The output is typically machine code or assembly code.

*   **Output for `a = (b + c) * (b + c) * 2` (Hypothetical Assembly-like Code, based on optimized TAC: `t1=b+c; t3=t1*t1; t5=t3*2; a=t5`):**
    ```assembly
    LOAD R1, b     ; Load value of b into register R1
    LOAD R2, c     ; Load value of c into register R2
    ADD R1, R1, R2 ; R1 = b + c (this is t1)
    MUL R2, R1, R1 ; R2 = t1 * t1 (this is t3)
    LOAD R3, #2    ; Load constant 2 into R3
    MUL R2, R2, R3 ; R2 = t3 * 2 (this is t5)
    STORE a, R2    ; Store the result from R2 into memory location for a
    ```
    (Alternative for `MUL R2, R2, R3` if `t5 = t3 << 1` was chosen: `SHL R2, R2, #1 ; R2 = t3 << 1`)

*   **Output for `id1 = id2 + id3 * 50` (Hypothetical Assembly-like Code, based on TAC: `t1=id3*50; t2=id2+t1; id1=t2`):**
    ```assembly
    LOAD R1, id3   ; Load value of id3 into R1
    LOAD R2, #50   ; Load immediate value 50 into R2
    MUL R1, R1, R2 ; R1 = id3 * 50 (this is t1)
    LOAD R2, id2   ; Load value of id2 into R2
    ADD R2, R2, R1 ; R2 = id2 + t1 (this is t2)
    STORE id1, R2  ; Store the result from R2 into memory location for id1
    ```

---

**Symbol Table Management:**
Throughout these phases, the compiler maintains a **symbol table**, which is a data structure containing a record for each identifier, with fields for the attributes of the identifier. This information is entered by the lexical analyzer and used/updated by later phases. For example, the semantic analyzer would use it to check types and the code generator would use it to determine memory addresses for identifiers.

**Error Handling:**
Each phase can encounter errors. A good compiler will not only detect these errors but also try to recover from them to continue processing the rest of the program, reporting multiple errors in a single compilation. For example, lexical errors (invalid characters), syntax errors (e.g., missing semicolon), semantic errors (type mismatch), and errors during code generation (e.g., value too large for target machine). 