# SLR Parser and Parsing Table Construction

This document explains SLR parsers and constructs SLR parsing tables for the given grammars.

---

## a) SLR Parser Explanation

### What is SLR Parser?

**SLR (Simple LR)** parser is a type of bottom-up parser that belongs to the LR family of parsers. It is simpler than canonical LR and LALR parsers but more powerful than operator precedence parsers.

### Key Characteristics

1. **LR(0) Items:** Uses LR(0) items to construct the parsing automaton
2. **FOLLOW Sets:** Uses FOLLOW sets to resolve conflicts
3. **Simple Construction:** Easier to construct than canonical LR parsers
4. **Limited Power:** Cannot handle all LR(1) grammars

### SLR Parser Components

1. **LR(0) Items:** Productions with a dot (•) indicating parsing progress
2. **Canonical Collection:** Set of all LR(0) item sets (states)
3. **Parsing Table:** ACTION and GOTO tables for parsing decisions
4. **Parsing Stack:** Stores states and symbols during parsing

### LR(0) Items

An **LR(0) item** is a production with a dot (•) somewhere in the right-hand side, indicating how much of the production has been seen.

**Example:** For production A → XYZ, the LR(0) items are:
- A → •XYZ (nothing seen yet)
- A → X•YZ (X has been seen)
- A → XY•Z (X and Y have been seen)
- A → XYZ• (everything has been seen - ready to reduce)

### Closure Operation

**CLOSURE(I)** computes the closure of a set of items I:

1. Start with I
2. For each item A → α•Bβ in the closure:
   - For each production B → γ:
     - Add B → •γ to the closure (if not already present)
3. Repeat until no new items can be added

### GOTO Operation

**GOTO(I, X)** computes the set of items reachable from item set I on symbol X:

1. For each item A → α•Xβ in I:
   - Add A → αX•β to the result
2. Return CLOSURE of the result

### SLR Parsing Table Construction

1. **Construct Canonical Collection:** Build all LR(0) item sets
2. **Build ACTION Table:**
   - If A → α•aβ is in Iᵢ and GOTO(Iᵢ, a) = Iⱼ, then ACTION[i, a] = shift j
   - If A → α• is in Iᵢ and A ≠ S', then ACTION[i, a] = reduce A → α for all a ∈ FOLLOW(A)
   - If S' → S• is in Iᵢ, then ACTION[i, $] = accept
3. **Build GOTO Table:**
   - If GOTO(Iᵢ, A) = Iⱼ, then GOTO[i, A] = j

---

## b) Grammar 1: Expression Grammar

### Given Grammar
```
E → E + T | T
T → T * F | F
F → F^ | a | b
```

### Step 1: Augment the Grammar
```
E' → E
E → E + T | T
T → T * F | F
F → F^ | a | b
```

### Step 2: Construct LR(0) Items

**All possible LR(0) items:**
```
E' → •E        E' → E•
E → •E + T     E → E• + T     E → E +• T     E → E + T•
E → •T         E → T•
T → •T * F     T → T• * F     T → T *• F     T → T * F•
T → •F         T → F•
F → •F^        F → F•^        F → F^•
F → •a         F → a•
F → •b         F → b•
```

### Step 3: Construct Canonical Collection

**I₀ = CLOSURE({E' → •E})**
```
E' → •E
E → •E + T
E → •T
T → •T * F
T → •F
F → •F^
F → •a
F → •b
```

**I₁ = GOTO(I₀, E)**
```
E' → E•
E → E• + T
```

**I₂ = GOTO(I₀, T)**
```
E → T•
T → T• * F
```

**I₃ = GOTO(I₀, F)**
```
T → F•
F → F•^
```

**I₄ = GOTO(I₀, a)**
```
F → a•
```

**I₅ = GOTO(I₀, b)**
```
F → b•
```

**I₆ = GOTO(I₁, +)**
```
E → E +• T
T → •T * F
T → •F
F → •F^
F → •a
F → •b
```

**I₇ = GOTO(I₂, *)**
```
T → T *• F
F → •F^
F → •a
F → •b
```

**I₈ = GOTO(I₃, ^)**
```
F → F^•
```

**I₉ = GOTO(I₆, T)**
```
E → E + T•
T → T• * F
```

**I₁₀ = GOTO(I₆, F)**
```
T → F•
F → F•^
```

**I₁₁ = GOTO(I₆, a)**
```
F → a•
```

**I₁₂ = GOTO(I₆, b)**
```
F → b•
```

**I₁₃ = GOTO(I₇, F)**
```
T → T * F•
F → F•^
```

**I₁₄ = GOTO(I₇, a)**
```
F → a•
```

**I₁₅ = GOTO(I₇, b)**
```
F → b•
```

**I₁₆ = GOTO(I₉, *)**
```
T → T *• F
F → •F^
F → •a
F → •b
```

**I₁₇ = GOTO(I₁₀, ^)**
```
F → F^•
```

**I₁₈ = GOTO(I₁₃, ^)**
```
F → F^•
```

**I₁₉ = GOTO(I₁₆, F)**
```
T → T * F•
F → F•^
```

**I₂₀ = GOTO(I₁₆, a)**
```
F → a•
```

**I₂₁ = GOTO(I₁₆, b)**
```
F → b•
```

**I₂₂ = GOTO(I₁₉, ^)**
```
F → F^•
```

### Step 4: Compute FOLLOW Sets

**FOLLOW(E) = {$, +}**
**FOLLOW(T) = {$, +, *}**
**FOLLOW(F) = {$, +, *, ^}**

### Step 5: Construct SLR Parsing Table

| State | a | b | + | * | ^ | $ | E | T | F |
|-------|---|---|---|---|---|---|---|---|---|
| 0 | s4 | s5 | | | | | 1 | 2 | 3 |
| 1 | | | s6 | | | acc | | | |
| 2 | | | r2 | s7 | | r2 | | | |
| 3 | | | r4 | r4 | s8 | r4 | | | |
| 4 | | | r6 | r6 | r6 | r6 | | | |
| 5 | | | r7 | r7 | r7 | r7 | | | |
| 6 | s11 | s12 | | | | | | 9 | 10 |
| 7 | s14 | s15 | | | | | | | 13 |
| 8 | | | r5 | r5 | r5 | r5 | | | |
| 9 | | | r1 | s16 | | r1 | | | |
| 10 | | | r4 | r4 | s17 | r4 | | | |
| 11 | | | r6 | r6 | r6 | r6 | | | |
| 12 | | | r7 | r7 | r7 | r7 | | | |
| 13 | | | r3 | r3 | s18 | r3 | | | |
| 14 | | | r6 | r6 | r6 | r6 | | | |
| 15 | | | r7 | r7 | r7 | r7 | | | |
| 16 | s20 | s21 | | | | | | | 19 |
| 17 | | | r5 | r5 | r5 | r5 | | | |
| 18 | | | r5 | r5 | r5 | r5 | | | |
| 19 | | | r3 | r3 | s22 | r3 | | | |
| 20 | | | r6 | r6 | r6 | r6 | | | |
| 21 | | | r7 | r7 | r7 | r7 | | | |
| 22 | | | r5 | r5 | r5 | r5 | | | |

**Legend:**
- sn = shift and go to state n
- rn = reduce by production n
- acc = accept
- Numbers in E, T, F columns are GOTO states

**Productions:**
1. E → E + T
2. E → T
3. T → T * F
4. T → F
5. F → F^
6. F → a
7. F → b

---

## Grammar 2: Assignment Grammar

### Given Grammar
```
S → L = R | R
L → *R | id
R → L
```

### Step 1: Augment the Grammar
```
S' → S
S → L = R | R
L → *R | id
R → L
```

### Step 2: Construct LR(0) Items and Canonical Collection

**I₀ = CLOSURE({S' → •S})**
```
S' → •S
S → •L = R
S → •R
L → •*R
L → •id
R → •L
```

**I₁ = GOTO(I₀, S)**
```
S' → S•
```

**I₂ = GOTO(I₀, L)**
```
S → L• = R
R → L•
```

**I₃ = GOTO(I₀, R)**
```
S → R•
```

**I₄ = GOTO(I₀, *)**
```
L → *•R
R → •L
L → •*R
L → •id
```

**I₅ = GOTO(I₀, id)**
```
L → id•
```

**I₆ = GOTO(I₂, =)**
```
S → L =• R
R → •L
L → •*R
L → •id
```

**I₇ = GOTO(I₄, R)**
```
L → *R•
```

**I₈ = GOTO(I₄, L)**
```
R → L•
```

**I₉ = GOTO(I₄, *)**
```
L → *•R
R → •L
L → •*R
L → •id
```

**I₁₀ = GOTO(I₄, id)**
```
L → id•
```

**I₁₁ = GOTO(I₆, R)**
```
S → L = R•
```

**I₁₂ = GOTO(I₆, L)**
```
R → L•
```

**I₁₃ = GOTO(I₆, *)**
```
L → *•R
R → •L
L → •*R
L → •id
```

**I₁₄ = GOTO(I₆, id)**
```
L → id•
```

### Step 3: Compute FOLLOW Sets

**FOLLOW(S) = {$}**
**FOLLOW(L) = {=, $}**
**FOLLOW(R) = {$}**

### Step 4: Construct SLR Parsing Table

| State | * | id | = | $ | S | L | R |
|-------|---|----|----|---|---|---|---|
| 0 | s4 | s5 | | | 1 | 2 | 3 |
| 1 | | | | acc | | | |
| 2 | | | s6 | r4 | | | |
| 3 | | | | r2 | | | |
| 4 | s9 | s10 | | | | 8 | 7 |
| 5 | | | r3 | r3 | | | |
| 6 | s13 | s14 | | | | 12 | 11 |
| 7 | | | r3 | r3 | | | |
| 8 | | | r4 | r4 | | | |
| 9 | s9 | s10 | | | | 8 | 7 |
| 10 | | | r3 | r3 | | | |
| 11 | | | | r1 | | | |
| 12 | | | r4 | r4 | | | |
| 13 | s13 | s14 | | | | 12 | 11 |
| 14 | | | r3 | r3 | | | |

**Productions:**
1. S → L = R
2. S → R
3. L → *R
4. L → id
5. R → L

**Note:** This grammar has a reduce-reduce conflict in state 2 on lookahead $. Both R → L and the implicit reduction could apply. This indicates the grammar is not SLR(1).

---

## Analysis and Conflicts

### Grammar 1 Analysis
The expression grammar is **SLR(1)** and can be parsed successfully. The precedence of operators is encoded in the grammar structure (^ has highest precedence, then *, then +).

### Grammar 2 Analysis
The assignment grammar has a **reduce-reduce conflict** and is **not SLR(1)**. The conflict occurs because:
- In state 2, when we see $, we could reduce L to R (using R → L) or reduce L = R to S
- The FOLLOW sets overlap, making it impossible to decide which reduction to perform

This grammar would require a more powerful parser (like LALR or canonical LR) or grammar modification to resolve the conflict.

---

## Summary

**SLR Parser Advantages:**
- Simple to construct
- Efficient parsing
- Good for many practical grammars

**SLR Parser Limitations:**
- Cannot handle all LR(1) grammars
- May have conflicts that more powerful parsers can resolve
- Less powerful than LALR and canonical LR parsers

The SLR parsing technique provides a good balance between simplicity and parsing power, making it suitable for many programming language constructs. 