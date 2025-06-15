# Answer to Question 1

---

### (a) What is a translator? Compare compiler and interpreter in terms of translation method, memory requirement and speed.

A **translator** is a type of system software that converts a program written in one programming language (the source language) into an equivalent program in another language (the target language). The target language can be another high-level language, machine code for a specific CPU, or an intermediate representation. Compilers and interpreters are the two primary types of translators.

**Comparison: Compiler vs. Interpreter**

| Feature                 | Compiler                                                                          | Interpreter                                                                      |
|:------------------------|:----------------------------------------------------------------------------------|:---------------------------------------------------------------------------------|
| **Translation Method**  | Translates the entire source program into machine code in one go, before execution. The resulting object code is saved separately. | Translates the source program line by line, executing each line as it is translated. No separate object file is created. |
| **Memory Requirement**  | **Higher.** A compiler needs memory to generate an intermediate representation and the final object code, which itself requires disk space. | **Lower.** An interpreter only needs enough memory to process the current line or statement. |
| **Speed of Execution**  | **Faster.** The object code is already in the machine's native language, so it runs very quickly when executed. The translation overhead is a one-time cost. | **Slower.** The program is translated line by line *every time it runs*. This translation overhead makes execution slower, especially in loops. |
| **Error Detection**     | Detects all syntax and static semantic errors in the entire program at once, after the compilation scan. It generates a list of all errors. | Detects errors line by line. It stops execution as soon as the first error is encountered, making it harder to see all errors at once. |
| **Platform Dependency** | The generated object code is specific to a target machine architecture. The source code must be re-compiled for each different platform. | Interpreted programs are more portable. The same source code can be run on any machine that has the appropriate interpreter. |
| **Development Cycle**   | The "edit-compile-run" cycle can be slower because compilation is a distinct, and sometimes lengthy, step. | Faster for development and debugging because code can be tested immediately without a separate compilation step. |
| **Example Languages**   | C, C++, C#, Java (compiles to bytecode)                                           | Python, JavaScript, Ruby, PHP                                                    |

---

### (b) Explain LEX tool in brief.

**LEX (Lexical Analyzer Generator)** is a standard tool on Unix-like systems for generating a lexical analyzer (also known as a scanner or tokenizer) from a set of regular expressions.

*   **Functionality:** It takes a source file, typically with a `.l` extension, which contains a set of rules. Each rule consists of a regular expression pattern and a corresponding action written in C. LEX processes this file and generates a C source file (usually named `lex.yy.c`) containing a function called `yylex()`.
*   **The `yylex()` function:** When compiled and linked with a parser, this function reads an input stream of characters, identifies sequences of characters (lexemes) that match the provided regular expressions, and executes the associated C code (the action). The action typically involves returning a token value to the parser.
*   **Structure of a LEX file:** A LEX program is divided into three sections, separated by `%%`:
    1.  **Definitions:** Contains declarations of variables, manifest constants, and regular definitions (aliases for common patterns).
    2.  **Rules:** The core of the program. Each line has a `pattern` (a regular expression) and an `{ action }` (a block of C code).
    3.  **User Subroutines:** Contains any additional C functions that are needed by the actions in the rules section.

In summary, LEX automates the tedious and error-prone task of writing a lexical analyzer by hand. The programmer only needs to specify the patterns for tokens, and LEX generates the efficient C code to recognize them.

---

### (c) What is left factoring in grammar? Explain with an example.

**Left factoring** is a grammar transformation technique used to make a grammar suitable for top-down parsing (e.g., LL(1) parsing). It is applied to eliminate non-determinism that occurs when two or more productions for the same non-terminal start with the same sequence of symbols (a common prefix).

A top-down parser cannot decide which production to choose by looking at the next input token if multiple productions start with the same symbol. Left factoring resolves this by "factoring out" the common prefix and creating a new non-terminal to represent the parts of the productions that differ.

**General Rule:**
A grammar with productions of the form:
`A -> αβ₁ | αβ₂ | ... | αβₙ`

where `α` is the common prefix, is transformed into:
```
A  -> αA'
A' -> β₁ | β₂ | ... | βₙ
```

**Example:**
Consider the following grammar for a conditional statement:
`stmt -> if expr then stmt | if expr then stmt else stmt`

This grammar is not suitable for a predictive parser because when the parser sees the token `if`, it doesn't know which of the two productions to choose. The common prefix is `if expr then stmt`.

**Applying Left Factoring:**
1.  **Identify the common prefix `α`:**
    *   `α = if expr then stmt`
2.  **Identify the differing parts `β`:**
    *   `β₁ = else stmt`
    *   `β₂ = ε` (the empty string, for the first rule)
3.  **Create a new non-terminal, e.g., `stmt_tail`:**
4.  **Rewrite the grammar:**
    ```
    stmt      -> if expr then stmt stmt_tail
    stmt_tail -> else stmt | ε
    ```

After left factoring, when the parser sees an `if`, it can confidently choose the single `stmt` production. After parsing `if expr then stmt`, it then calls the procedure for `stmt_tail` to see if an `else` part exists. This removes the non-determinism. 