# Answer to Question 2

---

### (a) Consider the following grammar for list structures and find the LMD, RMD, and parse tree for the given string.

**Grammar:**
```
S -> a | ^ | (T)
T -> T,S | S
```
**Input String:** `((a,a),^(a)),a`

#### Analysis of the Input String
First, it is essential to note that the provided string `((a,a),^(a)),a` **cannot be derived** from the given grammar. The reasons are:
1.  **Invalid Top-Level Structure:** The start symbol `S` can generate `(T)`, but it cannot generate `(T),a`. The trailing `,a` is outside the main parentheses, which is not permitted by any `S` production.
2.  **Invalid Substring `^(a)`:** The production `S -> ^` means `^` is a complete terminal string derived from `S`. There is no rule that allows `^` to be followed by `(a)`.

Therefore, a derivation for the given string is impossible. To demonstrate the concepts, we will use a corrected, complex string that **is** derivable from the grammar: **`((a,^),a)`**.

#### Derivations and Parse Tree for the string `((a,^),a)`

**i) Leftmost Derivation (LMD)**
In a leftmost derivation, the leftmost non-terminal is replaced at each step.
```
S => (T)
  => (T, S)
  => (S, S)
  => ((T), S)
  => ((T, S), S)
  => ((S, S), S)
  => ((a, S), S)
  => ((a, ^), S)
  => ((a, ^), a)
```

**ii) Rightmost Derivation (RMD)**
In a rightmost derivation, the rightmost non-terminal is replaced at each step.
```
S => (T)
  => (T, S)
  => (T, a)
  => (S, a)
  => ((T), a)
  => ((T, S), a)
  => ((T, ^), a)
  => ((S, ^), a)
  => ((a, ^), a)
```

**iii) Parse Tree**
Both LMD and RMD produce the same parse tree.

```
                      S
                      |
            .--------------------.
            |         T          |
            (         |          )
            .--------------.
            |         ,    |
            T              S
            |              |
            S              a
            |
      .------------.
      |      T     |
      (      |     )
      .----------.
      |     ,    |
      T          S
      |          |
      S          ^
      |
      a
```

---

### (b) For the following grammar find First and Follow sets for each non-terminal.

**Grammar:**
```
S -> aAB | bA | ε
A -> aAb | ε
B -> bB | ε
```

#### Calculation of FIRST Sets
The FIRST set of a non-terminal is the set of terminals that can begin a string derived from it.

*   **FIRST(B):**
    *   From `B -> bB`, we add `b`.
    *   From `B -> ε`, we add `ε`.
    *   **FIRST(B) = {b, ε}**

*   **FIRST(A):**
    *   From `A -> aAb`, we add `a`.
    *   From `A -> ε`, we add `ε`.
    *   **FIRST(A) = {a, ε}**

*   **FIRST(S):**
    *   From `S -> aAB`, we add `a`.
    *   From `S -> bA`, we add `b`.
    *   From `S -> ε`, we add `ε`.
    *   **FIRST(S) = {a, b, ε}**

**Summary of FIRST Sets:**
*   `FIRST(S) = {a, b, ε}`
*   `FIRST(A) = {a, ε}`
*   `FIRST(B) = {b, ε}`

#### Calculation of FOLLOW Sets
The FOLLOW set of a non-terminal X is the set of terminals that can appear immediately to the right of X. `$`, the end-of-input marker, is in `FOLLOW(S)`.

1.  **Initialize:** `FOLLOW(S) = {$}`.

2.  **Analyze Production `S -> aAB`:**
    *   For `A`: `FOLLOW(A)` gets `FIRST(B) - {ε}`, which is `{b}`.
    *   Since `FIRST(B)` contains `ε`, `FOLLOW(A)` also gets `FOLLOW(S)`, which is `{$}`.
    *   So, `FOLLOW(A)` contains `{b, $}`.
    *   For `B`: `B` is at the end, so `FOLLOW(B)` gets `FOLLOW(S)`, which is `{$}`.

3.  **Analyze Production `S -> bA`:**
    *   `A` is at the end, so `FOLLOW(A)` gets `FOLLOW(S)`. This adds `$` to `FOLLOW(A)`, which is already there.

4.  **Analyze Production `A -> aAb`:**
    *   For `A`: `FOLLOW(A)` gets what follows it, which is the terminal `b`. This adds `b` to `FOLLOW(A)`, which is already there.

5.  **Analyze Production `B -> bB`:**
    *   For `B`: `B` is at the end of the production `B -> bB`, so `FOLLOW(B)` gets `FOLLOW(B)`. This adds no new elements.

The sets have stabilized.

**Summary of FOLLOW Sets:**
*   `FOLLOW(S) = {$}`
*   `FOLLOW(A) = {b, $}`
*   `FOLLOW(B) = {$}`

---

### (c) Describe the following in brief: i) Ambiguity, ii) Left recursion.

#### i) Ambiguity
A grammar is said to be **ambiguous** if there exists at least one string for which the grammar can produce more than one parse tree. This means the string can have more than one leftmost derivation or more than one rightmost derivation.

Ambiguity is problematic for compilers because it means the syntactic structure of a program is not uniquely defined, leading to uncertainty about its meaning.

**Classic Example: The Dangling Else**
```
stmt -> if expr then stmt
      | if expr then stmt else stmt
```
For the string `if E1 then if E2 then S1 else S2`, this grammar allows two different parse trees: one where the `else` attaches to the inner `if` (the common convention) and another where it attaches to the outer `if`.

#### ii) Left Recursion
A grammar contains **left recursion** if a non-terminal `A` has a production of the form `A -> Aα`, where `α` is any sequence of grammar symbols. In simple terms, a non-terminal can derive a string that starts with itself.

Left recursion is a major problem for top-down parsers (like Recursive Descent or LL(1) parsers) because it causes them to enter an infinite loop. When the parser tries to expand `A`, its first step is to call the procedure for `A` again, leading to infinite recursion without consuming any input.

**Example:**
The standard grammar for arithmetic expressions is left-recursive:
```
E -> E + T | T
T -> T * F | F
```
Here, the production `E -> E + T` is directly left-recursive. This type of grammar must be transformed (e.g., by converting the left recursion to right recursion) before it can be parsed with a top-down parser. 