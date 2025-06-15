# SLR Parsing

This document first explains the concept of an SLR (Simple LR) parser and then demonstrates its application by computing the LR(0) items and constructing the SLR parsing table for two different grammars.

---

## a) SLR Parser Explained

An **SLR (Simple LR) parser** is a type of bottom-up, shift-reduce parser that belongs to the LR parser family (LR stands for **L**eft-to-right scan, **R**ightmost derivation in reverse). It is "simple" because it is one of the easiest LR parsers to implement.

The core of an SLR parser is a **parsing table** that dictates its actions. This table is constructed based on an analysis of the grammar. The parser itself is a state machine. Given its current state (at the top of the stack) and the next input token (the lookahead), it consults the parsing table to decide on one of four actions:

1.  **Shift:** Push the current input token and a new state onto the stack.
2.  **Reduce:** Reduce a handle on the stack according to a grammar rule, pop the corresponding symbols and states, and push the resulting non-terminal and a new state onto the stack.
3.  **Accept:** The input string has been successfully parsed.
4.  **Error:** A syntax error has been found.

### Construction of an SLR Parsing Table

The process involves three main steps:
1.  **Augment the Grammar:** A new start production `S' -> S` is added to the grammar, where `S` is the original start symbol. This helps the parser know when to stop and accept the input.
2.  **Compute LR(0) Items and Canonical Collection:**
    *   An **LR(0) item** is a production from the grammar with a dot (`.`) placed somewhere on the right-hand side. The dot indicates how much of the production we have seen at a given point in the parse. For example, for the production `E -> E + T`, the LR(0) items are `E -> .E + T`, `E -> E. + T`, `E -> E +. T`, and `E -> E + T.`.
    *   The parser's states are sets of these LR(0) items. We compute the **canonical collection of LR(0) item sets** (also called LR(0) states) using two operations:
        *   **CLOSURE(I):** For a set of items `I`, the closure is computed by repeatedly adding new items. If `A -> α.Bβ` is in `CLOSURE(I)` and `B -> γ` is a production, then the item `B -> .γ` is also added. This process continues until no more new items can be added.
        *   **GOTO(I, X):** For an item set `I` and a grammar symbol `X`, `GOTO(I, X)` is the closure of the set of all items `A -> αX.β` such that `A -> α.Xβ` is in `I`. The GOTO function defines the transitions between states.
3.  **Construct the Parsing Table:** The table has two parts: `ACTION` and `GOTO`.
    *   The `ACTION` table determines the shift/reduce action for a given state and terminal (including the end-of-input marker `$`).
    *   The `GOTO` table determines the next state after a reduction for a given state and non-terminal.

**Rules for Filling the SLR Parsing Table:**
For each state `Iᵢ` in the canonical collection:
1.  **Shift Action:** If `A -> α.aβ` is in `Iᵢ` and `GOTO(Iᵢ, a) = Iⱼ` (where `a` is a terminal), set `ACTION[i, a] = Shift j`.
2.  **Reduce Action:** If `A -> α.` is in `Iᵢ` (a "reduce item"), then for every terminal `a` in `FOLLOW(A)`, set `ACTION[i, a] = Reduce by A -> α`. **This is the key step for SLR:** it uses the FOLLOW set to decide where to place reduce actions.
3.  **Accept Action:** If `S' -> S.` is in `Iᵢ`, set `ACTION[i, $] = Accept`.
4.  **GOTO Entry:** If `GOTO(Iᵢ, A) = Iⱼ` (where `A` is a non-terminal), set `GOTO[i, A] = j`.
5.  All other entries are marked as `Error`.

If any table entry has more than one action defined (e.g., a shift and a reduce, or two different reduces), the grammar is not SLR(1).

---

## b) Computation for Grammars

### Grammar 1

**Augmented Grammar:**
```
0. E' -> E
1. E -> E + T
2. E -> T
3. T -> T * F
4. T -> F
5. F -> F^
6. F -> a
7. F -> b
```

**FOLLOW Sets (needed for the table):**
*   FOLLOW(E') = {$}
*   FOLLOW(E) = {+, $}
*   FOLLOW(T) = {+, *, $}
*   FOLLOW(F) = {+, *, ^, $}

**LR(0) Canonical Collection:**

*   **I₀ = CLOSURE({E' -> .E})**
    *   `E' -> .E`
    *   `E -> .E + T`
    *   `E -> .T`
    *   `T -> .T * F`
    *   `T -> .F`
    *   `F -> .F^`
    *   `F -> .a`
    *   `F -> .b`

*   **I₁ = GOTO(I₀, E)** = `CLOSURE({E' -> E., E -> E. + T})` = `{ E' -> E., E -> E. + T }`
*   **I₂ = GOTO(I₀, T)** = `CLOSURE({E -> T., T -> T. * F})` = `{ E -> T., T -> T. * F }`
*   **I₃ = GOTO(I₀, F)** = `CLOSURE({T -> F., F -> F.^})` = `{ T -> F., F -> F.^ }`
*   **I₄ = GOTO(I₀, a)** = `CLOSURE({F -> a.})` = `{ F -> a. }`
*   **I₅ = GOTO(I₀, b)** = `CLOSURE({F -> b.})` = `{ F -> b. }`

*   **I₆ = GOTO(I₁, +)** = `CLOSURE({E -> E + .T})`
    *   `E -> E + .T`
    *   `T -> .T * F`
    *   `T -> .F`
    *   `F -> .F^`
    *   `F -> .a`
    *   `F -> .b`

*   **I₇ = GOTO(I₂, *)** = `CLOSURE({T -> T * .F})`
    *   `T -> T * .F`
    *   `F -> .F^`
    *   `F -> .a`
    *   `F -> .b`

*   **I₈ = GOTO(I₃, ^)** = `CLOSURE({F -> F^.})` = `{ F -> F^. }`

*   **I₉ = GOTO(I₆, T)** = `CLOSURE({E -> E + T., T -> T. * F})` = `{ E -> E + T., T -> T. * F }`
*   **I₁₀ = GOTO(I₆, F)** = I₃
*   **I₁₁ = GOTO(I₆, a)** = I₄
*   **I₁₂ = GOTO(I₆, b)** = I₅

*   **I₁₃ = GOTO(I₇, F)** = `CLOSURE({T -> T * F., F -> F.^})` = `{ T -> T * F., F -> F.^ }`
*   **I₁₄ = GOTO(I₇, a)** = I₄
*   **I₁₅ = GOTO(I₇, b)** = I₅

*   **I₁₆ = GOTO(I₉, *)** = I₇

*   **I₁₇ = GOTO(I₁₃, ^)** = I₈

**SLR Parsing Table for Grammar 1**
(States I₁₀, I₁₁, I₁₂, I₁₄, I₁₅, I₁₆, I₁₇ are merged into existing states)

| State |       | ACTION                        |       |   | GOTO    |   |
|:-----:|:-----:|:-----------------------------:|:-----:|:-:|:-------:|:-:|
|       | **+** |            **\***             | **^** | **a** | **b** | **$** | **E** | **T** | **F** |
| **0** |       |                               | s4    | s5    |       |   1     |  2    |  3    |
| **1** |  s6   |                               |       |       | acc   |         |       |       |
| **2** |  R2   |              s7               | R2    |       | R2    |         |       |       |
| **3** |  R4   |              R4               | s8    |       | R4    |         |       |       |
| **4** |  R6   |              R6               | R6    |       | R6    |         |       |       |
| **5** |  R7   |              R7               | R7    |       | R7    |         |       |       |
| **6** |       |                               | s4    | s5    |       |         |  9    |  3    |
| **7** |       |                               | s4    | s5    |       |         |       |  13   |
| **8** |  R5   |              R5               | R5    |       | R5    |         |       |       |
| **9** |  R1   |              s7               | R1    |       | R1    |         |       |       |
| **13**|  R3   |              R3               | s8    |       | R3    |         |       |       |

*(R_n_ means Reduce by rule _n_. s_n_ means Shift to state _n_.)*

---
### Grammar 2

**Augmented Grammar:**
```
0. S' -> S
1. S -> L = R
2. S -> R
3. L -> *R
4. L -> id
5. R -> L
```

**FOLLOW Sets:**
*   FOLLOW(S') = {$}
*   FOLLOW(S) = {$}
*   FOLLOW(L) = {=, $}
*   FOLLOW(R) = {=, $}

**LR(0) Canonical Collection:**

*   **I₀ = CLOSURE({S' -> .S})**
    *   `S' -> .S`
    *   `S -> .L = R`
    *   `S -> .R`
    *   `L -> .*R`
    *   `L -> .id`
    *   `R -> .L`

*   **I₁ = GOTO(I₀, S)** = `CLOSURE({S' -> S.})` = `{ S' -> S. }`
*   **I₂ = GOTO(I₀, L)** = `CLOSURE({S -> L.= R, R -> L.})` = `{ S -> L.= R, R -> L. }`
*   **I₃ = GOTO(I₀, R)** = `CLOSURE({S -> R.})` = `{ S -> R. }`
*   **I₄ = GOTO(I₀, *)** = `CLOSURE({L -> *.R})`
    *   `L -> *.R`
    *   `R -> .L`
    *   `L -> .*R`
    *   `L -> .id`
*   **I₅ = GOTO(I₀, id)** = `CLOSURE({L -> id.})` = `{ L -> id. }`

*   **I₆ = GOTO(I₂, =)** = `CLOSURE({S -> L =.R})`
    *   `S -> L =.R`
    *   `R -> .L`
    *   `L -> .*R`
    *   `L -> .id`

*   **I₇ = GOTO(I₄, R)** = `CLOSURE({L -> *R.})` = `{ L -> *R. }`
*   **I₈ = GOTO(I₄, L)** = I₂
*   **I₉ = GOTO(I₄, *)** = I₄
*   **I₁₀ = GOTO(I₄, id)** = I₅

*   **I₁₁ = GOTO(I₆, R)** = `CLOSURE({S -> L = R.})` = `{ S -> L = R. }`
*   **I₁₂ = GOTO(I₆, L)** = `CLOSURE({R -> L.})` = `{ R -> L. }`
*   **I₁₃ = GOTO(I₆, *)** = I₄
*   **I₁₄ = GOTO(I₆, id)** = I₅

**Conflict Analysis:**
In state **I₂**, we have two items: `S -> L.= R` and `R -> L.`.
*   The lookahead token is `=`. The first item suggests `Shift 6`.
*   The second item, `R -> L.`, is a reduce item. We look at `FOLLOW(R) = {=, $}`.
    *   This means for lookahead `=`, we should `Reduce by R -> L`.
*   Therefore, in state `I₂` on input `=`, we have a **Shift-Reduce conflict**.

**Conclusion for Grammar 2:**
Because a shift-reduce conflict exists in state I₂, **the grammar is not SLR(1)**. We can still construct the table, but it will have a multi-valued entry.

**SLR Parsing Table for Grammar 2**

| State | ACTION                        |       | GOTO    |   |
|:-----:|:-----------------------------:|:-----:|:-------:|:-:|
|       | **=** | **\*** | **id** | **$** | **S** | **L** | **R** |
| **0** |       | s4     | s5     |       |   1   |  2    |  3    |
| **1** |       |        |        | acc   |       |       |       |
| **2** | **s6/R5** |        |        | **R5**  |       |       |       |
| **3** |  R2   |        |        | R2    |       |       |       |
| **4** |       | s4     | s5     |       |       |  8=2  |  7    |
| **5** |  R4   |        |        | R4    |       |       |       |
| **6** |       | s4     | s5     |       |       | 12    | 11    |
| **7** |  R3   |        |        | R3    |       |       |       |
| **11**|  R1   |        |        | R1    |       |       |       |
| **12**|  R5   |        |        | R5    |       |       |       |

*(The entry for State 2, on input '=', is `s6/R5`, indicating the shift-reduce conflict.)* 