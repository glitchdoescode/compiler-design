# LL(1) Grammar Analysis

This document explains LL(1) grammars and analyzes whether the given grammar is LL(1).

---

## a) What is an LL(1) Grammar?

### Definition

An **LL(1) grammar** is a context-free grammar that can be parsed by an LL(1) parser. The notation "LL(1)" stands for:
- **L**: Left-to-right scanning of input
- **L**: Leftmost derivation
- **1**: One symbol of lookahead

### Characteristics of LL(1) Grammars

1. **Deterministic Parsing:** For any non-terminal and any input symbol, there is at most one production rule that can be applied.

2. **No Left Recursion:** LL(1) grammars cannot have left recursion (direct or indirect).

3. **Left Factored:** All common prefixes must be factored out.

4. **Unambiguous:** LL(1) grammars are always unambiguous.

### Formal Conditions for LL(1)

A grammar G is LL(1) if and only if for every non-terminal A with productions A → α₁ | α₂ | ... | αₙ, the following conditions hold:

1. **FIRST sets are disjoint:** FIRST(αᵢ) ∩ FIRST(αⱼ) = ∅ for all i ≠ j

2. **FIRST-FOLLOW condition:** If ε ∈ FIRST(αᵢ) for some i, then:
   - FIRST(αⱼ) ∩ FOLLOW(A) = ∅ for all j ≠ i
   - At most one αᵢ can derive ε

### LL(1) Parsing Table

An LL(1) parser uses a parsing table M[A, a] where:
- A is a non-terminal
- a is a terminal or $
- M[A, a] contains the production to use when A is on top of stack and a is the current input symbol

**Table Construction Rules:**
1. For each production A → α:
   - For each terminal a ∈ FIRST(α), add A → α to M[A, a]
   - If ε ∈ FIRST(α), then for each b ∈ FOLLOW(A), add A → α to M[A, b]

2. If there are multiple entries for any M[A, a], the grammar is not LL(1)

---

## b) Checking if the Given Grammar is LL(1)

### Given Grammar

```
S → i E t S S' | a
S' → e S | ε
E → b
```

Where:
- i = if
- t = then  
- e = else
- a, b = other terminals

### Step 1: Check for Left Recursion

**Analysis:** None of the productions have left recursion.
- S → i E t S S' (starts with terminal i)
- S → a (starts with terminal a)
- S' → e S (starts with terminal e)
- S' → ε (epsilon production)
- E → b (starts with terminal b)

✓ **No left recursion found.**

### Step 2: Compute FIRST Sets

**FIRST(S):**
- From S → i E t S S': Add {i}
- From S → a: Add {a}
- **FIRST(S) = {i, a}**

**FIRST(S'):**
- From S' → e S: Add {e}
- From S' → ε: Add {ε}
- **FIRST(S') = {e, ε}**

**FIRST(E):**
- From E → b: Add {b}
- **FIRST(E) = {b}**

### Step 3: Compute FOLLOW Sets

**FOLLOW(S):**
- S is the start symbol: Add {$}
- From S → i E t S S': S is followed by S'
  - Add FIRST(S') - {ε} = {e} to FOLLOW(S)
  - Since ε ∈ FIRST(S'), add FOLLOW(S) to FOLLOW(S) (no new info)
- From S' → e S: S is at the end
  - Add FOLLOW(S') to FOLLOW(S)
- **FOLLOW(S) = {$, e} ∪ FOLLOW(S')**

**FOLLOW(S'):**
- From S → i E t S S': S' is at the end
  - Add FOLLOW(S) = {$, e} ∪ FOLLOW(S') to FOLLOW(S')
- This creates a dependency. Let's solve it:
- **FOLLOW(S') = {$, e}** (since S' appears at the end of S productions)
- Therefore: **FOLLOW(S) = {$, e}**

**FOLLOW(E):**
- From S → i E t S S': E is followed by t
  - Add {t} to FOLLOW(E)
- **FOLLOW(E) = {t}**

### Step 4: Check LL(1) Conditions

**For non-terminal S:**
Productions: S → i E t S S' | a

- FIRST(i E t S S') = {i}
- FIRST(a) = {a}
- FIRST(i E t S S') ∩ FIRST(a) = {i} ∩ {a} = ∅ ✓

**For non-terminal S':**
Productions: S' → e S | ε

- FIRST(e S) = {e}
- FIRST(ε) = {ε}
- FIRST(e S) ∩ FIRST(ε) = {e} ∩ {ε} = ∅ ✓

Since ε ∈ FIRST(ε), check FIRST-FOLLOW condition:
- FIRST(e S) ∩ FOLLOW(S') = {e} ∩ {$, e} = {e} ≠ ∅ ✗

**For non-terminal E:**
Productions: E → b

- Only one production, so trivially satisfies LL(1) conditions ✓

### Step 5: Construct LL(1) Parsing Table

| Non-terminal | i | a | e | b | t | $ |
|--------------|---|---|---|---|---|---|
| S | S → i E t S S' | S → a | | | | |
| S' | | | S' → e S, S' → ε | | | S' → ε |
| E | | | | E → b | | |

**Conflict detected:** M[S', e] has two entries:
- S' → e S (from FIRST(e S))
- S' → ε (from FOLLOW(S') since ε ∈ FIRST(ε))

### Conclusion

**The grammar is NOT LL(1)** because:

1. There is a FIRST-FOLLOW conflict for non-terminal S'
2. FIRST(e S) ∩ FOLLOW(S') = {e} ≠ ∅
3. The parsing table has multiple entries for M[S', e]

### The Problem: Dangling Else

This grammar represents the classic "dangling else" problem in programming languages. The ambiguity arises because when we see an "else", we don't know whether:
- It belongs to the current if statement (S' → e S)
- We should finish the current if statement without an else (S' → ε)

### Making it LL(1)

To make this grammar LL(1), we could:

1. **Eliminate the ambiguity** by requiring explicit delimiters
2. **Use precedence rules** (else matches the nearest unmatched if)
3. **Restructure the grammar** to avoid the conflict

**Example restructured grammar:**
```
S → i E t S S' | a
S' → e S | ε
```

Could be changed to:
```
S → MatchedS | UnmatchedS
MatchedS → i E t MatchedS e MatchedS | a
UnmatchedS → i E t S | i E t MatchedS e UnmatchedS
```

This eliminates the ambiguity but makes the grammar more complex.

---

## Summary

- **LL(1) Definition:** A grammar that can be parsed deterministically with one symbol of lookahead
- **Given Grammar:** NOT LL(1) due to FIRST-FOLLOW conflict in S' productions
- **Reason:** The dangling else problem creates ambiguity that cannot be resolved with one symbol of lookahead 