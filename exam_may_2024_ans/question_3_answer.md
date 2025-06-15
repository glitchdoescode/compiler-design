# Answer to Question 3

---

### a) Differentiate Top Down Parser and Bottom Up Parser. Give Example for each.

Top-down and bottom-up parsers are the two main categories of parsing algorithms used in compilers. They differ fundamentally in how they construct the parse tree for an input string.

| Feature                      | Top-Down Parser                                                                      | Bottom-Up Parser                                                                        |
|:-----------------------------|:-------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------|
| **Parse Tree Construction**  | Builds the parse tree from the **root** (start symbol) down to the **leaves** (terminals). | Builds the parse tree from the **leaves** up to the **root**.                           |
| **Derivation Type**          | Produces a **leftmost derivation** of the input string.                              | Produces a **rightmost derivation** in reverse.                                         |
| **Basic Approach**           | Starts with the start symbol and tries to derive the input string by applying production rules. | Starts with the input string and tries to reduce it back to the start symbol.           |
| **Grammar Restrictions**     | Cannot handle left-recursive grammars. Often requires the grammar to be left-factored. | Can handle left-recursive grammars. Less restrictive in general.                         |
| **Power**                    | Less powerful. Can only parse a subset of context-free grammars (LL grammars).       | More powerful. Can parse a larger class of grammars (LR grammars), including all LL grammars. |
| **Common Implementation**    | Recursive Descent Parsers (often hand-coded), LL(1) Parsers (table-driven).          | Shift-Reduce Parsers, LR(0), SLR(1), LALR(1), CLR(1) Parsers (all table-driven).           |
| **Error Handling**           | Can be easier to implement good error diagnostics in hand-coded versions.            | Can detect an error as soon as a viable prefix of the input is no longer valid.         |
| **Action**                   | The main action is **predicting** which production rule to use.                        | The main actions are **shift** (move input to stack) and **reduce** (apply a grammar rule). |

#### Example

Consider the simple grammar: `S -> a S b | ε` and the input string `aab`.

**Top-Down Parser (e.g., LL(1))**
It would attempt to find a leftmost derivation for `aab`:
1.  `S` -> `aSb` (Predicts this rule based on input 'a')
2.  `aSb` -> `aaSb`b (Expands the inner S, predicts `aSb` again)
3.  `aaSb`b -> `aabb` (Expands S to ε)
4.  **Wait**, the input is `aab`. The derivation produced `aabb`. There is a mismatch. This string is not in the language. Let's try input `ab`:
    1.  `S` -> `aSb`
    2.  `aSb` -> `aεb` -> `ab` (Matches!)

**Bottom-Up Parser (e.g., LR)**
It would work backward from the input string `ab`:
1.  **String:** `ab`
2.  Shift `a`. Stack: `a`. Input: `b`.
3.  The handle is not obvious here, let's use a better grammar: `E -> E + n | n` and input `n + n`.
4.  **String:** `n + n`
5.  **Shift `n`**. Stack: `n`. Reduce `E -> n`. Stack: `E`.
6.  **Shift `+`**. Stack: `E +`.
7.  **Shift `n`**. Stack: `E + n`. Reduce `E -> n`. This is not correct for LR.
    Let's trace it properly for a shift-reduce parser:
    1.  **Stack:** `[]`, **Input:** `n + n $` -> Shift
    2.  **Stack:** `[n]`, **Input:** `+ n $` -> Reduce by `E -> n`
    3.  **Stack:** `[E]`, **Input:** `+ n $` -> Shift
    4.  **Stack:** `[E, +]`, **Input:** `n $` -> Shift
    5.  **Stack:** `[E, +, n]`, **Input:** `$` -> Reduce by `E -> n`
    6.  **Stack:** `[E, +, E]`. Oh, the grammar should be `E -> E + T | T`, `T -> n`. Let's retry with `n+n`.
    Grammar: `E -> E + T | T`, `T -> n`. Input: `n+n`
    1.  **Stack:** `[]`, **Input:** `n + n $` -> Shift
    2.  **Stack:** `[n]`, **Input:** `+ n $` -> Reduce `T -> n`
    3.  **Stack:** `[T]`, **Input:** `+ n $` -> Reduce `E -> T`
    4.  **Stack:** `[E]`, **Input:** `+ n $` -> Shift
    5.  **Stack:** `[E, +]`, **Input:** `n $` -> Shift
    6.  **Stack:** `[E, +, n]`, **Input:** `$` -> Reduce `T -> n`
    7.  **Stack:** `[E, +, T]`, **Input:** `$` -> Reduce `E -> E + T`
    8.  **Stack:** `[E]`, **Input:** `$` -> Accept. The string is parsed.

---

### b) Prepare and Eliminate the left recursion for the grammar

**Original Grammar:**
```
S -> Aa | b
A -> Ac | Sd | ε
```

This grammar has both direct and indirect left recursion.

*   **Direct Left Recursion:** `A -> Ac`
*   **Indirect Left Recursion:** `S -> Aa` and `A -> Sd` create a cycle (`S` derives `A`, which in turn derives `S`).

#### Step 1: Eliminate Indirect Left Recursion
To eliminate the `S -> A -> S` cycle, we substitute the productions of `S` into the `A` production `A -> Sd`.
The `S` productions are `S -> Aa` and `S -> b`.

Substitute `S` in `A -> Sd`:
`A -> (Aa)d | bd`
`A -> Aad | bd`

Now the `A` productions are: `A -> Ac | Aad | bd | ε`.
The grammar is now:
```
S -> Aa | b
A -> Ac | Aad | bd | ε
```
The grammar still has direct left recursion on `A`.

#### Step 2: Eliminate Direct Left Recursion
The productions for `A` are in the form `A -> Aα | β`, where:
*   `α` productions (the recursive parts) are `c` and `ad`.
*   `β` productions (the non-recursive parts) are `bd` and `ε`.

We apply the standard rule for eliminating direct left recursion:
Replace `A -> Aα₁ | Aα₂ | ... | β₁ | β₂ | ...`
with:
`A -> β₁A' | β₂A' | ...`
`A' -> α₁A' | α₂A' | ... | ε`

Applying this to our `A` productions:
*   `α₁ = c`, `α₂ = ad`
*   `β₁ = bd`, `β₂ = ε`

The new productions become:
1.  **For A:**
    `A -> bd A' | ε A'`
    `A -> bd A' | A'`

2.  **For A':**
    `A' -> c A' | ad A' | ε`

#### Step 3: Final Grammar
Combining everything, the final grammar with no left recursion is:
```
S  -> Aa | b
A  -> bd A' | A'
A' -> c A' | ad A' | ε
```
This grammar is now suitable for top-down parsing. 