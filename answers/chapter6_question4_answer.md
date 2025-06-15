# Compiler Error Handling

A robust compiler must be able to detect errors in the source code, report them to the user in a clear and meaningful way, and, if possible, recover from them to continue checking the rest of the program. This document discusses the types of errors that occur in each phase of compilation and the strategies used for detection and recovery.

---

## 1. Errors Encountered in Different Compiler Phases

Errors can be detected at almost every phase of the compilation process.

### a) Lexical Errors
These errors occur during the lexical analysis phase when the scanner is unable to group the sequence of characters into a valid token.
*   **Examples:**
    *   An illegal character that is not part of the language's alphabet (e.g., `$` in a language that doesn't use it).
    *   A malformed token, like a number with multiple decimal points (`3.14.15`) or an identifier that starts with a number (`9var`).
    *   An unterminated string or character literal (e.g., `"hello world` without the closing quote).
*   **Detection:** The lexical analyzer detects these when its predefined patterns for tokens do not match.

### b) Syntax Errors
These are grammatical errors that occur during the parsing phase. The sequence of tokens does not conform to the context-free grammar rules of the language.
*   **Examples:**
    *   Missing semicolons (`a = b + c \n d = e`).
    *   Unbalanced parentheses or braces (`if (a > b { ... `).
    *   An operator with a missing operand (`a * ;`).
    *   Keywords in the wrong place (`else { ... }` without a preceding `if`).
*   **Detection:** The parser detects these when it is unable to match the stream of tokens to any valid production rule.

### c) Semantic Errors
These errors occur during the semantic analysis phase and relate to the meaning of the program, rather than its syntax. The code is grammatically correct but violates the language's semantic rules.
*   **Examples:**
    *   **Type Mismatch:** Trying to add a string to an integer (`"hello" + 5`).
    *   **Undeclared Variable:** Using a variable that has not been declared.
    *   **Multiple Declarations:** Declaring the same variable twice in the same scope.
    *   **Incorrect Function Arguments:** Calling a function with the wrong number or types of arguments.
    *   **Access Violations:** Trying to access a `private` member from outside its class.
*   **Detection:** The semantic analyzer detects these using the information stored in the symbol table and the type rules of the language.

### d) Logical Errors
These are errors in the programmer's logic. The program is syntactically and semantically correct, compiles without errors, but does not produce the intended result.
*   **Examples:**
    *   Using `=` (assignment) instead of `==` (comparison) in a condition.
    *   An infinite loop.
    *   An incorrect algorithm for a calculation.
*   **Detection:** The compiler **cannot** detect logical errors, as it does not understand the programmer's intent. These errors must be found through testing and debugging by the programmer.

---

## 2. Error Detection and Recovery Strategies

A good compiler should not give up after finding the first error. It should attempt to recover and continue analyzing the code to find as many errors as possible in a single compilation run.

### Error Detection
Detection is the process of discovering an error. The goal is to detect the error as soon as it occurs.
*   The parser's state machine immediately detects a syntax error when no valid shift or reduce action is possible.
*   The lexical analyzer detects an error when no rule matches the input characters.
*   The semantic analyzer detects an error when a rule of the language is violated (e.g., a type check fails).
Clear error messages are crucial, ideally including the file name, line number, and a hint about the nature of the error.

### Error Recovery Strategies

Recovery is the process of adjusting the parser's state to continue checking the rest of the program after an error has been found.

1.  **Panic Mode Recovery:**
    *   **Strategy:** This is the simplest and most common recovery method. When the parser detects an error, it discards input tokens one by one until a "synchronizing token" is found. Synchronizing tokens are typically delimiters, such as a semicolon `;`, a closing brace `}`, or a keyword that starts a new statement (`for`, `if`).
    *   **Pros:** Easy to implement and guaranteed not to get into an infinite loop.
    *   **Cons:** Can skip a large amount of input without checking it for other errors. The choice of synchronizing tokens is critical.

2.  **Phrase-Level Recovery:**
    *   **Strategy:** When an error is found, the parser performs a local correction on the remaining input. For example, it might replace a comma with a semicolon, delete an extraneous semicolon, or insert a missing one.
    *   **Pros:** Can correct small, common errors and avoid skipping large code sections.
    *   **Cons:** Much more complex to implement. It can be difficult to decide on the correct local repair, and a bad repair can lead to a cascade of further "spurious" errors.

3.  **Error Productions:**
    *   **Strategy:** The compiler designer anticipates common errors and adds special "error productions" to the grammar that match them. For example, to handle a missing semicolon, one could add a rule like `statement -> expression error_semicolon`. When this rule is used to reduce, the parser can report the specific error and continue.
    *   **Pros:** Allows for highly specific error messages and controlled recovery for common mistakes.
    *   **Cons:** Can significantly complicate the grammar and cannot account for all possible errors.

4.  **Global Correction:**
    *   **Strategy:** This is a theoretical approach where the compiler attempts to find the minimum number of changes (insertions, deletions, modifications) to the source code to transform it into a syntactically correct program.
    *   **Pros:** Would be the ideal solution if it were practical.
    *   **Cons:** Extremely complex and computationally expensive (NP-hard). It is too slow for use in production compilers. 