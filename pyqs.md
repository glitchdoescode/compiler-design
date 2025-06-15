### **Compiler Design PYQs: A Structured Study Guide**

This document organizes questions from the provided papers (Dec 2024, May 2024, May 2023, May 2022, Dec 2020) into a logical, chapter-by-chapter format.

---

### **Chapter 1: Introduction to Compilation**

This chapter covers the fundamental concepts of compilers and language translation.

**1. Structure and Phases of a Compiler (Most Frequently Asked)**
*   **Question:** Describe the structure of a compiler. Explain in detail the various phases of a compiler (the analysis and synthesis model). For each phase, explain its function and show the output for the following two input statements:
    *   `a = (b + c) * (b + c) * 2`
    *   `id1 = id2 + id3 * 50`
*(This single question combines similar requests from May 2023, May 2024, May 2022, and Dec 2020 papers.)*

**2. Compiler vs. Interpreter**
*   **Question:** What is a translator? Compare and contrast a compiler and an interpreter based on their translation method, memory requirements, and speed.
*(From Dec 2024 paper.)*

**3. Compiler Construction Tools**
*   **Question:** What are compiler construction tools? Explain the LEX tool in detail. Design a LEX program that recognizes the tokens of the C language and returns the token found.
*(This combines general questions on tools with specific questions about LEX from Dec 2024, May 2023, and Dec 2020.)*

**4. Data Structures in a Compiler**
*   **Question:** Describe the various data structures used in the implementation of a compiler.
*(From Dec 2020 paper.)*

**5. Cross Compiler & Optimizing Compilers**
*   **Question:**
    *   a) What is a cross compiler? Give an example.
    *   b) What are the properties of optimizing compilers?
*(From May 2022 paper.)*

---

### **Chapter 2: Lexical Analysis**

This chapter focuses on the first phase of the compiler, which scans the source code.

**1. Key Concepts in Lexical Analysis**
*   **Question:** Explain the following terms with examples:
    *   a) Lexical Analysis
    *   b) Tokens
    *   c) Input Buffering
*(From May 2022 and Dec 2020 papers.)*

**2. Finite State Machines (FSM)**
*   **Question:** Explain Finite State Machine (FSM) with its limitations and applications in lexical analysis.
*(From May 2022 paper.)*

---

### **Chapter 3: Syntax Analysis (Parsing)**

This is a major chapter covering grammars, parsing techniques, and table construction.

#### **Part A: Grammars and Their Properties**

**1. Context-Free Grammars (CFG)**
*   **Question:** What is a Context-Free Grammar (CFG)? Explain the closure properties of context-free languages.
*(From May 2022 paper.)*

**2. Ambiguity, Left Recursion, and Left Factoring**
*   **Question:** Explain the following concepts in a grammar with suitable examples:
    *   a) Ambiguity
    *   b) Left Recursion
    *   c) Left Factoring
*(This combines definition questions from the Dec 2024 paper.)*

**3. Eliminating Left Recursion**
*   **Question:** Eliminate the left recursion from the following grammar:
    ```
    S -> Aa | b
    A -> Ac | Sd | ε 
    ```
*(This is a practical problem from the May 2024 paper.)*

**4. Derivations and Parse Trees**
*   **Question:** Consider the following grammar for list structures:
    ```
    S -> a | ^ | (T)
    T -> T, S | S
    ```
    For the input string `( (a, a), ^(a) ), a` find the:
    *   a) Leftmost Derivation (LMD)
    *   b) Rightmost Derivation (RMD)
    *   c) Parse Tree
*(From Dec 2024 paper. Note: The string in the original paper had a syntax error, corrected here to be derivable.)*

#### **Part B: Parsing Techniques and Concepts**

**5. Top-Down vs. Bottom-Up Parsing**
*   **Question:** Differentiate between Top-Down and Bottom-Up parsing. Give an example of each. Also, explain backtracking and non-backtracking parsers.
*(This combines similar questions from May 2024, May 2023, and Dec 2024 papers.)*

**6. FIRST and FOLLOW Sets**
*   **Question:** For the following grammar, find the FIRST and FOLLOW sets for each non-terminal:
    ```
    S -> aAB | bA | ε
    A -> aAb | ε
    B -> bB | ε
    ```
*(From Dec 2024 paper.)*

**7. LL(1) Parsing**
*   **Question:**
    *   a) What is an LL(1) grammar?
    *   b) Check whether the following grammar is LL(1) or not:
        ```
        S -> i E t S S' | a
        S' -> e S | ε
        E -> b
        ```
*(From May 2022 and Dec 2020 papers.)*

**8. Recursive Descent Parsing**
*   **Question:** Construct a Recursive Descent Parser with backtracking for the grammar:
    ```
    S -> aSbS | bSaS | ε
    ```
*(From May 2023 paper.)*

**9. Shift-Reduce Parsing Conflicts**
*   **Question:** What are the various conflicts that can occur during shift-reduce parsing?
*(From May 2024 paper.)*

#### **Part C: Bottom-Up Parser Construction**

**10. SLR Parser**
*   **Question:**
    *   a) Explain the SLR Parser.
    *   b) Compute the LR(0) items and construct the SLR parsing table for the following grammars:
        *   **Grammar 1:**
            ```
            E -> E + T | T
            T -> T * F | F
            F -> F^ | a | b
            ```
        *   **Grammar 2:**
            ```
            S -> L = R | R
            L -> *R | id
            R -> L
            ```
*(This combines questions from May 2022, May 2023, and Dec 2024 papers.)*

**11. LALR Parser**
*   **Question:** Construct the LALR parsing table for the grammar below. Verify whether the input string `id + id * id` is accepted by the grammar.
    ```
    E -> E + T | T
    T -> T * F | F
    F -> (E) | id
    ```
*(From May 2024 paper.)*

---

### **Chapter 4: Syntax-Directed Translation & Intermediate Code Generation**

This chapter deals with attaching semantic rules to productions and generating intermediate code.

**1. Attributes (Synthesized and Inherited)**
*   **Question:**
    *   a) Differentiate between Synthesized and Inherited attributes with a suitable example.
    *   b) Define S-attributed SDT and L-attributed SDT.
    *   c) Define the following with an example: Annotated Parse Tree and Dependency Graph.
*(This combines multiple related questions from Dec 2024 and May 2024 papers.)*

**2. Syntax-Directed Translation (SDT) Schemes**
*   **Question 2a (Evaluation):** For the grammar below with the given translation rules, compute `E.value` for the expression `2 & 3 & 5 # 6 & 4`. Show the annotated parse tree.
    ```
    E -> E1 # T      { E.value = E1.value * T.value }
    E -> T           { E.value = T.value }
    T -> T1 & F      { T.value = T1.value + F.value }
    T -> F           { T.value = F.value }
    F -> num         { F.value = num.value }
    ```
*   **Question 2b (Translation):** Construct a Syntax Directed Translation Scheme that translates arithmetic expressions from infix into postfix notation.
*(From Dec 2024 and Dec 2020 papers.)*

**3. Three-Address Code (TAC)**
*   **Question:**
    *   a) Explain Three-Address Code.
    *   b) Generate the three-address code for the following code segments:
        *   `while (a < c AND b < d) do`
            `  if (a = 1) then c = c + 1`
            `  else while (a <= 4) do a = a + 3`
        *   `P := (K + Y) + (K - C)`
*(This combines TAC questions from May 2023, Dec 2024, and May 2024.)*

**4. Backpatching**
*   **Question:** Explain the concept of Backpatching with an example.
*(From May 2022 and Dec 2020 papers.)*

**5. Directed Acyclic Graph (DAG)**
*   **Question:**
    *   a) What is a DAG? How is it useful for transformations on basic blocks?
    *   b) Write an algorithm to construct a DAG from a basic block.
    *   c) Construct a DAG for the following statements:
        *   `a = (a * b + c) - (a * b + c)`
        *   `a = a * (b - c) + (b - c) * d`
        *   `D := B * C; E := A + B; B := B * C; A := E - D;`
*(This combines all DAG-related questions into one comprehensive problem set.)*

**6. Issues in Code Generation**
*   **Question:** Discuss the primary issues in the design of a code generator.
*(From Dec 2020 paper.)*

---

### **Chapter 5: Runtime Environment & Symbol Table**

This chapter covers memory management during program execution and symbol table organization.

**1. Storage Allocation Strategies**
*   **Question:**
    *   a) Discuss the various storage allocation strategies in detail (Static, Stack, Heap).
    *   b) How is Stack storage allocation different from Heap allocation? Describe their merits and demerits.
    *   c) Explain Dynamic Storage Allocation.
*(Combines questions from May 2024, May 2023, and May 2024's short notes.)*

**2. Symbol Table Management**
*   **Question:** Explain the symbol table management system in a compiler.
*(From Dec 2020 paper.)*

**3. Type Systems**
*   **Question:**
    *   a) Explain in brief about the equivalence of type expressions.
    *   b) Explain different Polymorphic functions with suitable examples.
*(From May 2023 paper.)*

---

### **Chapter 6: Code Optimization**

This final chapter focuses on techniques to make the generated code faster and smaller.

**1. Basics of Optimization**
*   **Question:** What are the principal sources of optimization? Explain the three main areas of code optimization in detail. Differentiate between local and global transformations.
*(Combines questions from Dec 2024 and May 2023.)*

**2. Basic Blocks and Flow Graphs**
*   **Question:**
    *   a) Explain Basic Blocks and Control Flow Graphs.
    *   b) What are the steps involved in partitioning a sequence of three-address statements into basic blocks?
    *   c) Explain data flow analysis of a structured flow graph.
*(Combines questions from Dec 2020, May 2022 and May 2023.)*

**3. Optimization Techniques (Most Frequently Asked)**
*   **Question:** Explain the following code optimization techniques with suitable examples:
    *   a) Common Sub-expression Elimination
    *   b) Copy Propagation
    *   c) Loop Invariant Computations (Code Motion)
    *   d) Strength Reduction
    *   e) Peephole Optimization
    *   f) Loop Optimization
    *   g) Register Allocation
*(This is a comprehensive list compiled from short notes and detailed questions across all papers.)*

**4. Error Handling**
*   **Question:** Explain the various errors encountered in different phases of a compiler. Discuss Error Detection and Recovery strategies.
*(From Dec 2020 and May 2024 papers.)*