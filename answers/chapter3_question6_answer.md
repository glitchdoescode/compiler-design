# FIRST and FOLLOW Sets

This document calculates the FIRST and FOLLOW sets for each non-terminal in the given grammar.

---

## Given Grammar

```
S → aAB | bA | ε
A → aAb | ε
B → bB | ε
```

---

## FIRST Sets

The FIRST set of a non-terminal A is the set of terminals that can appear as the first symbol of strings derived from A.

### Rules for Computing FIRST Sets

1. If X is a terminal, then FIRST(X) = {X}
2. If X → ε is a production, then add ε to FIRST(X)
3. If X → Y₁Y₂...Yₖ is a production:
   - Add FIRST(Y₁) - {ε} to FIRST(X)
   - If ε ∈ FIRST(Y₁), add FIRST(Y₂) - {ε} to FIRST(X)
   - Continue until Yᵢ where ε ∉ FIRST(Yᵢ) or all symbols are processed
   - If ε ∈ FIRST(Yᵢ) for all i = 1 to k, then add ε to FIRST(X)

### Computing FIRST Sets

**FIRST(S):**
- From S → aAB: Add {a} to FIRST(S)
- From S → bA: Add {b} to FIRST(S)  
- From S → ε: Add {ε} to FIRST(S)

**FIRST(S) = {a, b, ε}**

**FIRST(A):**
- From A → aAb: Add {a} to FIRST(A)
- From A → ε: Add {ε} to FIRST(A)

**FIRST(A) = {a, ε}**

**FIRST(B):**
- From B → bB: Add {b} to FIRST(B)
- From B → ε: Add {ε} to FIRST(B)

**FIRST(B) = {b, ε}**

---

## FOLLOW Sets

The FOLLOW set of a non-terminal A is the set of terminals that can appear immediately to the right of A in some sentential form.

### Rules for Computing FOLLOW Sets

1. Add $ to FOLLOW(S) where S is the start symbol
2. If A → αBβ is a production, then add FIRST(β) - {ε} to FOLLOW(B)
3. If A → αBβ is a production and ε ∈ FIRST(β), then add FOLLOW(A) to FOLLOW(B)
4. If A → αB is a production, then add FOLLOW(A) to FOLLOW(B)

### Computing FOLLOW Sets

**FOLLOW(S):**
- S is the start symbol, so add $ to FOLLOW(S)
- No other productions have S on the right-hand side

**FOLLOW(S) = {$}**

**FOLLOW(A):**
- From S → aAB: A is followed by B
  - Add FIRST(B) - {ε} = {b, ε} - {ε} = {b} to FOLLOW(A)
  - Since ε ∈ FIRST(B), add FOLLOW(S) = {$} to FOLLOW(A)
- From S → bA: A is at the end
  - Add FOLLOW(S) = {$} to FOLLOW(A)
- From A → aAb: A is followed by b
  - Add {b} to FOLLOW(A)

**FOLLOW(A) = {b, $}**

**FOLLOW(B):**
- From S → aAB: B is at the end
  - Add FOLLOW(S) = {$} to FOLLOW(B)
- From B → bB: B is at the end
  - Add FOLLOW(B) to FOLLOW(B) (no new information)

**FOLLOW(B) = {$}**

---

## Summary

| Non-terminal | FIRST Set | FOLLOW Set |
|--------------|-----------|------------|
| S | {a, b, ε} | {$} |
| A | {a, ε} | {b, $} |
| B | {b, ε} | {$} |

---

## Verification

Let's verify our results with some example derivations:

**Example 1:** S ⇒ aAB ⇒ aεB ⇒ aB ⇒ abB ⇒ abε ⇒ ab
- First symbol: a ✓ (a ∈ FIRST(S))
- A is followed by B, and B can start with b or ε, so A can be followed by b or $ ✓

**Example 2:** S ⇒ bA ⇒ baAb ⇒ baaAbb ⇒ baaεbb ⇒ baabb
- First symbol: b ✓ (b ∈ FIRST(S))
- A is followed by b ✓ (b ∈ FOLLOW(A))

**Example 3:** S ⇒ ε
- First symbol: ε ✓ (ε ∈ FIRST(S))

**Example 4:** S ⇒ aAB ⇒ aaAbB ⇒ aaεbB ⇒ aabB ⇒ aabε ⇒ aab
- A is followed by b ✓ (b ∈ FOLLOW(A))
- B is followed by end of string ✓ ($ ∈ FOLLOW(B))

All verifications confirm our FIRST and FOLLOW sets are correct.

---

## Applications

FIRST and FOLLOW sets are used in:

1. **LL(1) Parser Construction:** To build predictive parsing tables
2. **Error Recovery:** To determine valid continuation points
3. **Grammar Analysis:** To check if a grammar is LL(1)
4. **Syntax Error Detection:** To identify unexpected tokens

The computed sets show that this grammar can potentially be used for LL(1) parsing, as we can distinguish between alternatives based on the first symbol of input. 