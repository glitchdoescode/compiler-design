# Context-Free Grammars and Closure Properties

This document provides a definition for Context-Free Grammars (CFGs) and explains the closure properties of the languages they generate, known as context-free languages.

---

## 1. Context-Free Grammar (CFG)

A **Context-Free Grammar (CFG)** is a formal grammar used to describe a context-free language. It is a powerful tool for modeling the syntax of most programming languages. A CFG is "context-free" because any of its production rules can be applied regardless of the context of a non-terminal symbol.

A CFG is formally defined as a 4-tuple G = (V, T, P, S):

1.  **V (or N):** A finite set of **non-terminal symbols**. These are variables or placeholders that represent sets of strings. They are typically written as uppercase letters.
    *   *Example:* `{S, A, B, Expression, Statement}`

2.  **T (or Σ):** A finite set of **terminal symbols**. These are the basic symbols or tokens from which strings of the language are formed (the alphabet). They are typically written as lowercase letters, digits, or punctuation. `V` and `T` are disjoint sets (`V ∩ T = ∅`).
    *   *Example:* `{a, b, 0, 1, if, then, +, *, (, )}`

3.  **P:** A finite set of **production rules**. These rules define how non-terminals can be replaced by sequences of terminals and/or other non-terminals. Each rule is of the form:
    `A -> α`
    where `A` is a non-terminal (`A ∈ V`) and `α` is a string of symbols from `(V ∪ T)*` (a sequence of zero or more terminals and non-terminals). The `->` means "can be replaced by".
    *   *Example:* `S -> aSb`, `E -> E + T`, `A -> ε` (where ε is the empty string).

4.  **S:** The **start symbol**. This is a special non-terminal (`S ∈ V`) from which all strings in the language must be derived.

**Example of a CFG:**
Consider a grammar for simple arithmetic expressions:
*   V = {E, T, F} (Expression, Term, Factor)
*   T = {id, +, *, (, )}
*   P = {
        E -> E + T | T,
        T -> T * F | F,
        F -> (E) | id
    }
*   S = E

This grammar can generate strings like `id`, `id + id`, `id * id`, `(id + id) * id`, etc.

---

## 2. Closure Properties of Context-Free Languages (CFLs)

A set of languages is "closed" under an operation if applying that operation to any language(s) in the set results in a language that is also in the set. Context-free languages are closed under certain operations but not others.

### Operations Under Which CFLs ARE Closed

1.  **Union:** If L1 and L2 are two context-free languages, then their union L1 ∪ L2 is also a context-free language.
    *   **Proof Idea:** Let G1 = (V1, T1, P1, S1) and G2 = (V2, T2, P2, S2) be the CFGs for L1 and L2. We can construct a new grammar G for L1 ∪ L2 by combining them. We create a new start symbol S, and add the productions `S -> S1 | S2`. We must ensure the non-terminal sets V1 and V2 are disjoint (by renaming if necessary). The resulting grammar G generates the union of the two languages.

2.  **Concatenation:** If L1 and L2 are two context-free languages, then their concatenation L1 • L2 (the set of strings formed by taking a string from L1 and appending a string from L2) is also a context-free language.
    *   **Proof Idea:** Similar to union, let G1 and G2 be the grammars for L1 and L2. Create a new start symbol S and add the production `S -> S1S2`. Again, we ensure V1 and V2 are disjoint. The resulting grammar G generates the concatenation of the languages.

3.  **Kleene Star (or Kleene Closure):** If L is a context-free language, then L* (the set of strings formed by concatenating zero or more strings from L) is also a context-free language.
    *   **Proof Idea:** Let G = (V, T, P, S) be the grammar for L. We create a new grammar G' with a new start symbol S'. We add the productions `S' -> S'S | ε`. This allows for the derivation of any number of S instances, concatenated together, or the empty string.

4.  **Homomorphism:** A homomorphism is a function that replaces each symbol in a string with another string. CFLs are closed under homomorphism.

5.  **Inverse Homomorphism:** CFLs are also closed under inverse homomorphism.

6.  **Reversal:** If L is a CFL, its reversal Lᴿ (the set of all strings in L, but spelled backward) is also a CFL.
    *   **Proof Idea:** To get a grammar for Lᴿ, we simply reverse the right-hand side of every production in the original grammar for L.

### Operations Under Which CFLs are NOT Closed

1.  **Intersection:** If L1 and L2 are two context-free languages, their intersection L1 ∩ L2 is **not** necessarily context-free.
    *   **Counterexample:** Consider the languages:
        *   L1 = {aⁿbⁿcᵐ | n, m ≥ 0} (equal a's and b's). This is a CFL.
        *   L2 = {aᵐbⁿcⁿ | n, m ≥ 0} (equal b's and c's). This is also a CFL.
        *   Their intersection is L1 ∩ L2 = {aⁿbⁿcⁿ | n ≥ 0}. This language is a well-known example of a **context-sensitive language** that is not context-free, as a pushdown automaton cannot simultaneously track counts for three different symbols.

2.  **Complementation:** If L is a context-free language, its complement (the set of all strings over the alphabet Σ* that are not in L) is **not** necessarily context-free.
    *   **Proof Idea:** This can be shown using De Morgan's laws. If CFLs were closed under complementation, they would also have to be closed under intersection, because L1 ∩ L2 = ¬(¬L1 ∪ ¬L2). Since we know CFLs are closed under union but not intersection, they cannot be closed under complementation.

3.  **Difference:** If L1 and L2 are CFLs, their difference L1 - L2 (L1 ∩ ¬L2) is **not** necessarily context-free.
    *   **Proof Idea:** This follows directly from the lack of closure under intersection and complementation. 