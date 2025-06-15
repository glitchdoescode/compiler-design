# LL(1) Grammars

This document defines what an LL(1) grammar is and then checks if a given grammar satisfies the LL(1) condition.

---

## a) What is an LL(1) grammar?

An **LL(1) grammar** is a type of context-free grammar that can be parsed by an LL(1) parser. The name "LL(1)" stands for:

*   **L (first L):** The input is scanned from **L**eft to right.
*   **L (second L):** The parser produces a **L**eftmost derivation.
*   **(1):** The parser uses **1** token of lookahead to make parsing decisions.

A grammar is formally defined as LL(1) if, for any non-terminal `A` with two or more distinct productions `A -> α` and `A -> β`, the following conditions hold:

**Condition 1: Disjoint FIRST Sets**
*   The sets of terminals that can be derived at the beginning of `α` and `β` must be disjoint. In other words, `FIRST(α) ∩ FIRST(β) = ∅`.
*   This condition ensures that if we see a token `t` in the input, at most one of `α` or `β` can produce a string starting with `t`.

**Condition 2: Handling Nullable Productions (ε)**
*   If one of the productions can derive the empty string (ε), say `β -> ε`, then the set of terminals that can begin `α` must be disjoint from the set of terminals that can *follow* the non-terminal `A`.
*   Formally, if `ε` is in `FIRST(β)`, then `FIRST(α) ∩ FOLLOW(A) = ∅`.
*   This condition resolves ambiguity when a production for `A` could derive nothing (`ε`). If the current input token is something that can follow `A` in a valid sentence, the parser should choose the `A -> ε` production. If the current input token can be at the start of another production for `A`, it should choose that one. This condition ensures there's no overlap, so the parser can make a unique choice.

In short, for any non-terminal, a parser looking at the next single input token must be able to uniquely decide which production rule to apply. If it can, the grammar is LL(1). LL(1) grammars are a subset of context-free grammars and are important because they can be parsed efficiently in linear time by non-backtracking predictive parsers.

---

## b) Check whether the given grammar is LL(1) or not

**Grammar:**
```
1. S -> i E t S S' | a
2. S'-> e S | ε
3. E -> b
```

The non-terminals are {S, S', E}. The terminals are {i, t, a, e, b}. S is the start symbol.

To check if this grammar is LL(1), we must compute the FIRST and FOLLOW sets for each non-terminal and then check the two conditions for every non-terminal that has more than one production. In this grammar, only `S` and `S'` have multiple productions.

### Step 1: Compute FIRST Sets

*   **FIRST(E):**
    *   From `E -> b`: 'b' is a terminal.
    *   `FIRST(E) = {b}`

*   **FIRST(S'):**
    *   From `S' -> e S`: 'e' is a terminal.
    *   From `S' -> ε`: ε is a production.
    *   `FIRST(S') = {e, ε}`

*   **FIRST(S):**
    *   From `S -> i E t S S'`: 'i' is a terminal.
    *   From `S -> a`: 'a' is a terminal.
    *   `FIRST(S) = {i, a}`

**Summary of FIRST Sets:**
*   FIRST(S) = {i, a}
*   FIRST(S') = {e, ε}
*   FIRST(E) = {b}

### Step 2: Compute FOLLOW Sets

*   **FOLLOW(S):**
    *   S is the start symbol, so `$` is in `FOLLOW(S)`.
    *   In `S' -> e S`, S is at the end, so `FOLLOW(S')` is in `FOLLOW(S)`.
    *   We need `FOLLOW(S')` first.

*   **FOLLOW(S'):**
    *   In `S -> i E t S S'`, S' is at the end, so `FOLLOW(S)` is in `FOLLOW(S')`.

*   **FOLLOW(E):**
    *   In `S -> i E t S S'`, E is followed by the terminal 't'.
    *   `FOLLOW(E) = {t}`

Now we resolve the circular dependency between `FOLLOW(S)` and `FOLLOW(S')`.
*   Initially: `FOLLOW(S) = {$}` and `FOLLOW(S') = {}`.
*   **Pass 1:**
    *   `FOLLOW(S')` gets `FOLLOW(S)`, so `FOLLOW(S') = {$}`.
    *   `FOLLOW(S)` gets `FOLLOW(S')`, so `FOLLOW(S)` is still `{$}`.
*   **Pass 2:**
    *   `FOLLOW(S')` gets `FOLLOW(S)`, no change.
    *   `FOLLOW(S)` gets `FOLLOW(S')`, no change.

The sets have stabilized.

**Summary of FOLLOW Sets:**
*   FOLLOW(S) = {$}
*   FOLLOW(S') = {$}
*   FOLLOW(E) = {t}

### Step 3: Check LL(1) Conditions

We only need to check the conditions for non-terminals `S` and `S'`, as they are the only ones with multiple productions.

**For non-terminal S:**
The productions are `S -> α` where `α = i E t S S'` and `S -> β` where `β = a`.

*   `FIRST(α) = FIRST(i E t S S') = {i}`
*   `FIRST(β) = FIRST(a) = {a}`
*   **Check Condition 1:** `FIRST(α) ∩ FIRST(β) = {i} ∩ {a} = ∅`.
*   This condition holds for S. Neither production is nullable, so Condition 2 is not applicable.

**For non-terminal S':**
The productions are `S' -> α` where `α = e S` and `S' -> β` where `β = ε`.

*   `FIRST(α) = FIRST(e S) = {e}`
*   `FIRST(β) = FIRST(ε) = {ε}`
*   One production derives `ε`, so we must check Condition 2.
*   **Check Condition 2:** `FIRST(α) ∩ FOLLOW(S') = ∅` ?
    *   `FIRST(α) = {e}`
    *   `FOLLOW(S') = {$}`
    *   `{e} ∩ {$} = ∅`.
*   This condition holds for S'. Condition 1 is trivially true since `FIRST(β)` does not contain any terminals.

### Conclusion

Since both conditions for an LL(1) grammar are satisfied for all non-terminals with multiple productions (`S` and `S'`), **the given grammar is LL(1)**. 