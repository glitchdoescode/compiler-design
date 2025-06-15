# LALR Parser Construction and String Verification

This document constructs the LALR parsing table for the given grammar and verifies whether the input string `id + id * id` is accepted.

---

## Given Grammar

```
E → E + T | T
T → T * F | F
F → (E) | id
```

---

## LALR Parser Overview

**LALR (Look-Ahead LR)** parser is a type of LR parser that:
- Uses LR(1) items but merges states with identical cores
- More powerful than SLR but less complex than canonical LR
- Suitable for most programming languages
- Used by tools like YACC and Bison

---

## Step 1: Augment the Grammar

```
E' → E
E → E + T | T
T → T * F | F
F → (E) | id
```

---

## Step 2: Construct LR(1) Items

An LR(1) item has the form [A → α•β, a] where:
- A → α•β is an LR(0) item
- a is a lookahead symbol

### Initial Item Set

**I₀ = CLOSURE({[E' → •E, $]})**

Computing closure:
```
[E' → •E, $]
[E → •E + T, $]     (from E' → •E with lookahead $)
[E → •T, $]         (from E' → •E with lookahead $)
[E → •E + T, +]     (from E → •E + T with lookahead $)
[E → •T, +]         (from E → •E + T with lookahead $)
[T → •T * F, $]     (from E → •T with lookaheads $, +)
[T → •F, $]         (from E → •T with lookaheads $, +)
[T → •T * F, +]     (from E → •T with lookaheads $, +)
[T → •F, +]         (from E → •T with lookaheads $, +)
[T → •T * F, *]     (from T → •T * F with lookaheads $, +)
[T → •F, *]         (from T → •T * F with lookaheads $, +)
[F → •(E), $]       (from T → •F with lookaheads $, +, *)
[F → •id, $]        (from T → •F with lookaheads $, +, *)
[F → •(E), +]       (from T → •F with lookaheads $, +, *)
[F → •id, +]        (from T → •F with lookaheads $, +, *)
[F → •(E), *]       (from T → •F with lookaheads $, +, *)
[F → •id, *]        (from T → •F with lookaheads $, +, *)
```

**Simplified I₀:**
```
[E' → •E, $]
[E → •E + T, $]
[E → •T, $]
[E → •E + T, +]
[E → •T, +]
[T → •T * F, $]
[T → •F, $]
[T → •T * F, +]
[T → •F, +]
[T → •T * F, *]
[T → •F, *]
[F → •(E), $]
[F → •id, $]
[F → •(E), +]
[F → •id, +]
[F → •(E), *]
[F → •id, *]
```

---

## Step 3: Construct All LR(1) Item Sets

**I₁ = GOTO(I₀, E)**
```
[E' → E•, $]
[E → E• + T, $]
[E → E• + T, +]
```

**I₂ = GOTO(I₀, T)**
```
[E → T•, $]
[E → T•, +]
[T → T• * F, $]
[T → T• * F, +]
[T → T• * F, *]
```

**I₃ = GOTO(I₀, F)**
```
[T → F•, $]
[T → F•, +]
[T → F•, *]
```

**I₄ = GOTO(I₀, ()**
```
[F → (•E), $]
[F → (•E), +]
[F → (•E), *]
[E → •E + T, )]
[E → •T, )]
[T → •T * F, )]
[T → •F, )]
[T → •T * F, *]
[T → •F, *]
[F → •(E), )]
[F → •id, )]
[F → •(E), *]
[F → •id, *]
```

**I₅ = GOTO(I₀, id)**
```
[F → id•, $]
[F → id•, +]
[F → id•, *]
```

**I₆ = GOTO(I₁, +)**
```
[E → E +• T, $]
[E → E +• T, +]
[T → •T * F, $]
[T → •F, $]
[T → •T * F, +]
[T → •F, +]
[T → •T * F, *]
[T → •F, *]
[F → •(E), $]
[F → •id, $]
[F → •(E), +]
[F → •id, +]
[F → •(E), *]
[F → •id, *]
```

**I₇ = GOTO(I₂, *)**
```
[T → T *• F, $]
[T → T *• F, +]
[T → T *• F, *]
[F → •(E), $]
[F → •id, $]
[F → •(E), +]
[F → •id, +]
[F → •(E), *]
[F → •id, *]
```

**I₈ = GOTO(I₄, E)**
```
[F → (E•), $]
[F → (E•), +]
[F → (E•), *]
[E → E• + T, )]
```

**I₉ = GOTO(I₄, T)**
```
[E → T•, )]
[T → T• * F, )]
[T → T• * F, *]
```

**I₁₀ = GOTO(I₄, F)**
```
[T → F•, )]
[T → F•, *]
```

**I₁₁ = GOTO(I₄, id)**
```
[F → id•, )]
[F → id•, *]
```

**I₁₂ = GOTO(I₆, T)**
```
[E → E + T•, $]
[E → E + T•, +]
[T → T• * F, $]
[T → T• * F, +]
[T → T• * F, *]
```

**I₁₃ = GOTO(I₆, F)**
```
[T → F•, $]
[T → F•, +]
[T → F•, *]
```

**I₁₄ = GOTO(I₆, id)**
```
[F → id•, $]
[F → id•, +]
[F → id•, *]
```

**I₁₅ = GOTO(I₇, F)**
```
[T → T * F•, $]
[T → T * F•, +]
[T → T * F•, *]
```

**I₁₆ = GOTO(I₇, id)**
```
[F → id•, $]
[F → id•, +]
[F → id•, *]
```

**I₁₇ = GOTO(I₈, ))**
```
[F → (E)•, $]
[F → (E)•, +]
[F → (E)•, *]
```

**I₁₈ = GOTO(I₈, +)**
```
[E → E +• T, )]
[T → •T * F, )]
[T → •F, )]
[T → •T * F, *]
[T → •F, *]
[F → •(E), )]
[F → •id, )]
[F → •(E), *]
[F → •id, *]
```

**I₁₉ = GOTO(I₉, *)**
```
[T → T *• F, )]
[T → T *• F, *]
[F → •(E), )]
[F → •id, )]
[F → •(E), *]
[F → •id, *]
```

**I₂₀ = GOTO(I₁₂, *)**
```
[T → T *• F, $]
[T → T *• F, +]
[T → T *• F, *]
[F → •(E), $]
[F → •id, $]
[F → •(E), +]
[F → •id, +]
[F → •(E), *]
[F → •id, *]
```

---

## Step 4: Merge States with Same Core (LALR Construction)

In LALR, we merge states that have the same LR(0) core but different lookaheads.

**Core Analysis:**
- I₅, I₁₁, I₁₄, I₁₆ all have core {F → id•} → Merge into I₅'
- I₃, I₁₀, I₁₃ all have core {T → F•} → Merge into I₃'
- I₇, I₂₀ have same core → Merge into I₇'

**Merged States:**

**I₅' (merged I₅, I₁₁, I₁₄, I₁₆):**
```
[F → id•, $]
[F → id•, +]
[F → id•, *]
[F → id•, )]
```

**I₃' (merged I₃, I₁₀, I₁₃):**
```
[T → F•, $]
[T → F•, +]
[T → F•, *]
[T → F•, )]
```

**I₇' (merged I₇, I₂₀):**
```
[T → T *• F, $]
[T → T *• F, +]
[T → T *• F, *]
[T → T *• F, )]
[F → •(E), $]
[F → •id, $]
[F → •(E), +]
[F → •id, +]
[F → •(E), *]
[F → •id, *]
[F → •(E), )]
[F → •id, )]
```

---

## Step 5: Construct LALR Parsing Table

| State | id | + | * | ( | ) | $ | E | T | F |
|-------|----|----|---|---|---|---|---|---|---|
| 0 | s5 | | | s4 | | | 1 | 2 | 3 |
| 1 | | s6 | | | | acc | | | |
| 2 | | r2 | s7 | | r2 | r2 | | | |
| 3 | | r4 | r4 | | r4 | r4 | | | |
| 4 | s11 | | | s4 | | | 8 | 9 | 10 |
| 5 | | r6 | r6 | | r6 | r6 | | | |
| 6 | s14 | | | s4 | | | | 12 | 13 |
| 7 | s16 | | | s4 | | | | | 15 |
| 8 | | s18 | | | s17 | | | | |
| 9 | | r2 | s19 | | r2 | | | | |
| 10 | | r4 | r4 | | r4 | | | | |
| 11 | | r6 | r6 | | r6 | | | | |
| 12 | | r1 | s20 | | r1 | r1 | | | |
| 13 | | r4 | r4 | | r4 | r4 | | | |
| 14 | | r6 | r6 | | r6 | r6 | | | |
| 15 | | r3 | r3 | | r3 | r3 | | | |
| 16 | | r6 | r6 | | r6 | r6 | | | |
| 17 | | r5 | r5 | | r5 | r5 | | | |
| 18 | s11 | | | s4 | | | | 21 | 10 |
| 19 | s11 | | | s4 | | | | | 22 |
| 20 | s14 | | | s4 | | | | | 15 |
| 21 | | r2 | s19 | | r2 | | | | |
| 22 | | r3 | r3 | | r3 | | | | |

**Productions:**
1. E → E + T
2. E → T
3. T → T * F
4. T → F
5. F → (E)
6. F → id

**Legend:**
- sn = shift and go to state n
- rn = reduce by production n
- acc = accept

---

## Step 6: Verify Input String `id + id * id`

**Parsing Trace:**

| Step | Stack | Input | Action |
|------|-------|-------|--------|
| 1 | 0 | id + id * id $ | shift 5 |
| 2 | 0 id 5 | + id * id $ | reduce F → id |
| 3 | 0 F 3 | + id * id $ | reduce T → F |
| 4 | 0 T 2 | + id * id $ | reduce E → T |
| 5 | 0 E 1 | + id * id $ | shift 6 |
| 6 | 0 E 1 + 6 | id * id $ | shift 14 |
| 7 | 0 E 1 + 6 id 14 | * id $ | reduce F → id |
| 8 | 0 E 1 + 6 F 13 | * id $ | reduce T → F |
| 9 | 0 E 1 + 6 T 12 | * id $ | shift 20 |
| 10 | 0 E 1 + 6 T 12 * 20 | id $ | shift 14 |
| 11 | 0 E 1 + 6 T 12 * 20 id 14 | $ | reduce F → id |
| 12 | 0 E 1 + 6 T 12 * 20 F 15 | $ | reduce T → T * F |
| 13 | 0 E 1 + 6 T 12 | $ | reduce E → E + T |
| 14 | 0 E 1 | $ | accept |

**Result: ACCEPTED** ✓

---

## Analysis

### Parse Tree Generated

```
        E
      / | \
     E  +  T
     |   / | \
     T  T  *  F
     |  |     |
     F  F    id
     |  |
    id id
```

### Operator Precedence

The grammar correctly handles operator precedence:
- `*` has higher precedence than `+` (due to grammar structure)
- Both operators are left-associative
- The expression `id + id * id` is parsed as `id + (id * id)`

### LALR vs SLR

This grammar is LALR(1) and would also be SLR(1). The LALR construction provides:
- More precise lookahead information
- Better conflict resolution
- Ability to handle more grammars than SLR

---

## Summary

1. **Grammar Analysis:** The given grammar is LALR(1) and generates arithmetic expressions with correct precedence
2. **LALR Construction:** Successfully built LALR parsing table with 23 states
3. **String Verification:** Input `id + id * id` is **ACCEPTED** by the grammar
4. **Parse Result:** Correctly parsed with `*` having higher precedence than `+`

The LALR parser successfully handles this expression grammar and provides efficient parsing for arithmetic expressions with proper operator precedence and associativity. 