# Eliminating Left Recursion

This document provides a step-by-step solution for eliminating left recursion from the given grammar.

**Original Grammar:**
```
S -> Aa | b
A -> Ac | Sd | ε 
```

---

## Analysis of the Grammar

First, we need to identify the type of left recursion present.

1.  **Direct Left Recursion:** The production `A -> Ac` is a direct left recursion because the non-terminal `A` on the left-hand side is also the leftmost symbol on the right-hand side.
2.  **Indirect Left Recursion:** The grammar also contains an indirect (or mutual) left recursion between `S` and `A`.
    *   `S` produces a string starting with `A` (from `S -> Aa`).
    *   `A` produces a string starting with `S` (from `A -> Sd`).
    *   This creates a cycle `S => Aa => Sda`, which shows that `S` derives a string starting with itself through `A`.

To eliminate left recursion from a grammar, we must resolve both indirect and direct forms.

---

## Step 1: Eliminate Indirect Left Recursion

The standard algorithm to eliminate left recursion requires that the non-terminals be ordered, say A₁, A₂, ..., Aₙ. We then process them in order. For the `i`-th non-terminal, we substitute in the productions for all preceding non-terminals Aⱼ (where j < i) to reveal any hidden direct left recursion.

Let's establish an order for our non-terminals: `S, A`.

**Processing S (i=1):**
The productions for `S` are `S -> Aa | b`. They do not have any left recursion yet. We leave them as they are.

**Processing A (i=2):**
The productions for `A` are `A -> Ac | Sd | ε`.
The production `A -> Sd` contains `S`, a preceding non-terminal. We must substitute the productions of `S` into this rule.

Substitute `S -> Aa | b` into `A -> Sd`:
*   `A -> (Aa)d | (b)d`
*   `A -> Aad | bd`

Now, the set of productions for `A` becomes:
```
A -> Ac | Aad | bd | ε
```

After this substitution, we can see that the grammar now has two direct left-recursive productions for `A`: `A -> Ac` and `A -> Aad`.

The grammar, after resolving the indirect recursion, is:
```
S -> Aa | b
A -> Ac | Aad | bd | ε
```

---

## Step 2: Eliminate Direct Left Recursion

Now we eliminate the direct left recursion from the `A` productions. The general form for eliminating direct left recursion from `A -> Aα₁ | Aα₂ | ... | β₁ | β₂ | ...` is:
```
A  -> β₁A' | β₂A' | ...
A' -> α₁A' | α₂A' | ... | ε
```

For our `A` productions: `A -> Ac | Aad | bd | ε`
*   The recursive parts (`Aα`) are: `Ac` and `Aad`. So, `α₁ = c` and `α₂ = ad`.
*   The non-recursive parts (`β`) are: `bd` and `ε`. So, `β₁ = bd` and `β₂ = ε`.

Applying the transformation rule:

**New productions for `A`:**
`A -> β₁A' | β₂A'`
*   `A -> bdA'`
*   `A -> εA'` which simplifies to `A -> A'`
So, the `A` productions are: `A -> bdA' | A'`

**New productions for `A'`:**
`A' -> α₁A' | α₂A' | ε`
*   `A' -> cA' | adA' | ε`

---

## Final Resulting Grammar

Combining everything, the final grammar with all left recursion eliminated is:

```
S  -> Aa | b
A  -> bdA' | A'
A' -> cA' | adA' | ε
```

This new grammar is equivalent to the original grammar (generates the same language) but is not left-recursive, making it suitable for use with top-down parsing algorithms. 