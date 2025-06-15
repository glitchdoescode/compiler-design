# FIRST and FOLLOW Sets for Grammar

This document provides the calculation of FIRST and FOLLOW sets for each non-terminal in the following grammar:

```
S -> aAB | bA | ε
A -> aAb | ε
B -> bB | ε
```

The non-terminals are S, A, B. The terminals are a, b. ε represents the empty string. S is the start symbol.

---

## 1. Calculation of FIRST Sets

The FIRST set of a non-terminal X, denoted FIRST(X), is the set of terminals that can begin a string derived from X. If X can derive ε, then ε is also in FIRST(X).

**Rules for computing FIRST sets:**
1.  If X is a terminal, then FIRST(X) = {X}.
2.  If X -> ε is a production, then add ε to FIRST(X).
3.  If X is a non-terminal and X -> Y1 Y2 ... Yk is a production:
    *   Add FIRST(Y1) - {ε} to FIRST(X).
    *   If ε is in FIRST(Y1), then add FIRST(Y2) - {ε} to FIRST(X).
    *   ...
    *   If ε is in FIRST(Y_i-1) for all i > 1, then add FIRST(Y_i) - {ε} to FIRST(X).
    *   If ε is in FIRST(Y_i) for all i from 1 to k, then add ε to FIRST(X).

**Computing FIRST sets for the given grammar:**

*   **FIRST(B):**
    *   From `B -> bB`: 'b' is a terminal, so 'b' is in FIRST(B).
    *   From `B -> ε`: ε is in FIRST(B).
    *   Therefore, **FIRST(B) = {b, ε}**

*   **FIRST(A):**
    *   From `A -> aAb`: 'a' is a terminal, so 'a' is in FIRST(A).
    *   From `A -> ε`: ε is in FIRST(A).
    *   Therefore, **FIRST(A) = {a, ε}**

*   **FIRST(S):**
    *   From `S -> aAB`: 'a' is a terminal, so 'a' is in FIRST(S).
    *   From `S -> bA`: 'b' is a terminal, so 'b' is in FIRST(S).
    *   From `S -> ε`: ε is in FIRST(S).
    *   Therefore, **FIRST(S) = {a, b, ε}**

**Summary of FIRST Sets:**
*   FIRST(S) = {a, b, ε}
*   FIRST(A) = {a, ε}
*   FIRST(B) = {b, ε}

---

## 2. Calculation of FOLLOW Sets

The FOLLOW set of a non-terminal X, denoted FOLLOW(X), is the set of terminals that can appear immediately to the right of X in some sentential form. If X can be the rightmost symbol in some sentential form, then $ (the end-of-input marker) is in FOLLOW(X). ε is never in a FOLLOW set.

**Rules for computing FOLLOW sets:**
1.  Place $ in FOLLOW(S), where S is the start symbol.
2.  If there is a production A -> αBβ, then add FIRST(β) - {ε} to FOLLOW(B).
3.  If there is a production A -> αB, or a production A -> αBβ where ε is in FIRST(β), then add FOLLOW(A) to FOLLOW(B).

These rules are applied iteratively until no more changes occur in any FOLLOW set.

**Initial FOLLOW sets:**
*   FOLLOW(S) = {$}
*   FOLLOW(A) = {}
*   FOLLOW(B) = {}

**Iterative Calculation:**

**Applying rules to productions:**

1.  **S -> aAB**
    *   For non-terminal A: β is B.
        *   Add FIRST(B) - {ε} = {b} to FOLLOW(A). So, FOLLOW(A) becomes {b}.
        *   Since ε ∈ FIRST(B), add FOLLOW(S) = {$} to FOLLOW(A). So, FOLLOW(A) becomes {b, $}.
    *   For non-terminal B: B is at the end (A -> αB form, where A=S, α=aA).
        *   Add FOLLOW(S) = {$} to FOLLOW(B). So, FOLLOW(B) becomes {$}.

2.  **S -> bA**
    *   For non-terminal A: A is at the end (A -> αB form, where A=S, α=b).
        *   Add FOLLOW(S) = {$} to FOLLOW(A). FOLLOW(A) is {b, $} (no change).

3.  **S -> ε**
    *   This production does not directly contribute to FOLLOW sets of other non-terminals based on these rules.

4.  **A -> aAb**
    *   For non-terminal A (the one on the RHS): β is 'b'.
        *   Add FIRST(b) - {ε} = {b} to FOLLOW(A). FOLLOW(A) is {b, $} (no change).
    *   (Note: 'b' is a terminal, we are finding FOLLOW sets for non-terminals).

5.  **A -> ε**
    *   This production does not directly contribute to FOLLOW sets of other non-terminals.

6.  **B -> bB**
    *   For non-terminal B (the one on the RHS): B is at the end (A -> αB form, where A=B_LHS, α=b).
        *   Add FOLLOW(B_LHS) to FOLLOW(B_RHS). Since FOLLOW(B) is currently {$}, this rule implies FOLLOW(B) includes elements already in FOLLOW(B), so no new terminals are added by this step alone unless FOLLOW(B) changes due to other rules.

**Current sets after one pass:**
*   FOLLOW(S) = {$}
*   FOLLOW(A) = {b, $}
*   FOLLOW(B) = {$}

Further iterations will not change these sets as all rules have been applied and their consequences incorporated.

**Summary of FOLLOW Sets:**
*   FOLLOW(S) = {$}
*   FOLLOW(A) = {b, $}
*   FOLLOW(B) = {$} 