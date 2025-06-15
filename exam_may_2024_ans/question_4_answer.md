# Answer to Question 4

---

### a) Construct a LALR parsing table for the grammar given above. Verify whether the input string `id + id * id` is accepted by the grammar or not.

**Grammar:**
```
0. E' -> E
1. E -> E + T
2. E -> T
3. T -> T * F
4. T -> F
5. F -> (E)
6. F -> id
```

#### Step 1: Construct the LR(1) Canonical Collection of Item Sets

First, we create the canonical collection of LR(1) items. An LR(1) item is a production with a dot and a lookahead symbol, e.g., `[A -> α.β, a]`.

*   **I₀ = CLOSURE({[E' -> .E, $]})**
    *   `[E' -> .E, $]`
    *   `[E -> .E + T, $]`
    *   `[E -> .T, $]`
    *   `[T -> .T * F, +/$]` (Lookahead is FIRST(+$) or FIRST($))
    *   `[T -> .F, +/$]`
    *   `[F -> .(E), +/*/$]`
    *   `[F -> .id, +/*/$]`

*   **I₁ = GOTO(I₀, E)** = `CLOSURE({[E' -> E., $], [E -> E. + T, $]})`
*   **I₂ = GOTO(I₀, T)** = `CLOSURE({[E -> T., $], [T -> T. * F, +/$]})`
*   **I₃ = GOTO(I₀, F)** = `CLOSURE({[T -> F., +/$]})`
*   **I₄ = GOTO(I₀, id)** = `CLOSURE({[F -> id., +/*/$]})`
*   **I₅ = GOTO(I₀, ()** = `CLOSURE({[F -> (.E), +/*/$], ...})` -> This leads to a set of new states which we can label I₅, and then GOTO from there. The structure will mirror I₀ but with different lookaheads.
    *   `[F -> (.E), +/*/$]`
    *   `[E -> .E + T, )]`
    *   `[E -> .T, )]`
    *   `[T -> .T * F, +/*/)`
    *   `[T -> .F, +/*/)`
    *   `[F -> .(E), +/*/)`
    *   `[F -> .id, +/*/)`

*   **I₆ = GOTO(I₁, +)** = `CLOSURE({[E -> E + .T, $]})`
    *   `[E -> E + .T, $]`
    *   `[T -> .T * F, $]`
    *   `[T -> .F, $]`
    *   `[F -> .(E), */$]`
    *   `[F -> .id, */$]`

*   **I₇ = GOTO(I₂, *)** = `CLOSURE({[T -> T * .F, +/$]})`
    *   `[T -> T * .F, +/$]`
    *   `[F -> .(E), +/$]`
    *   `[F -> .id, +/$]`

...and so on. Constructing all LR(1) states is lengthy. Key states will be:
*   I₀, I₁, I₂, I₃, I₄, I₅ (from `(`)
*   I₆ = GOTO(I₁, +)
*   I₇ = GOTO(I₂, *)
*   I₈ = GOTO(I₅, E) -> `{[F->(E.),+/*/$], [E->E.+T,)]}`
*   I₉ = GOTO(I₆, T) -> `{[E->E+T.,$], [T->T.*F,$]}`
*   ...and several others.

#### Step 2: Merge LR(1) Sets to Form LALR Sets
LALR parsers merge LR(1) states that have the same core set of LR(0) items, combining their lookaheads.
*   `I₃` has core `{T -> F.}`. Let's see if another state has this core. Yes, `GOTO(I₆, F)` would be `CLOSURE({[T -> F., $]})`. So, we merge these into a new state, let's call it `I_36`, with lookaheads `{+ / * / $}`.
*   `I₄` has core `{F -> id.}`. `GOTO(I₅, id)` and `GOTO(I₆, id)` will also have this core. We merge them into a state `I_456` with lookaheads `{+ / * / $ / )}`.
*   `I₇`'s GOTO on `F` will result in a state with core `{T -> T*F.}`. Let's call it `I_10`.
This merging process simplifies the state machine.

#### Step 3: Construct the LALR Parsing Table

| State |  id |  +  |  *  |  (  |  )  |  $  | E | T | F |
|:-----:|:---:|:---:|:---:|:---:|:---:|:---:|:-:|:-:|:-:|
| **0** | s4  |     |     | s5  |     |     | 1 | 2 | 3 |
| **1** |     | s6  |     |     |     | acc |   |   |   |
| **2** |     | R2  | s7  |     | R2  | R2  |   |   |   |
| **3** |     | R4  | R4  |     | R4  | R4  |   |   |   |
| **4** |     | R6  | R6  |     | R6  | R6  |   |   |   |
| **5** | s4  |     |     | s5  |     |     | 8 | 2 | 3 |
| **6** | s4  |     |     | s5  |     |     |   | 9 | 3 |
| **7** | s4  |     |     | s5  |     |     |   |   | 10 |
| **8** |     | s6  |     |     | s11 |     |   |   |   |
| **9** |     | R1  | s7  |     | R1  | R1  |   |   |   |
| **10**|     | R3  | R3  |     | R3  | R3  |   |   |   |
| **11**|     | R5  | R5  |     | R5  | R5  |   |   |   |

*   `sX` = Shift to state X.
*   `RX` = Reduce by rule X.
*   `acc` = Accept. Blanks are errors.

#### Step 4: Verify `id + id * id`

| Stack       | Input           | Action                 |
|:------------|:----------------|:-----------------------|
| [0]         | `id + id * id $`| Shift 4                |
| [0, 4]      | `+ id * id $`   | Reduce F -> id (R6)    |
| [0, 3]      | `+ id * id $`   | Reduce T -> F (R4)     |
| [0, 2]      | `+ id * id $`   | Reduce E -> T (R2)     |
| [0, 1]      | `+ id * id $`   | Shift 6                |
| [0, 1, 6]   | `id * id $`     | Shift 4                |
| [0, 1, 6, 4]| `* id $`        | Reduce F -> id (R6)    |
| [0, 1, 6, 3]| `* id $`        | Reduce T -> F (R4)     |
| [0, 1, 6, 9]| `* id $`        | Shift 7                |
| [0, 1, 6, 9, 7] | `id $`      | Shift 4                |
| [0, 1, 6, 9, 7, 4] | `$`     | Reduce F -> id (R6)    |
| [0, 1, 6, 9, 7, 10] | `$`    | Reduce T -> T * F (R3) |
| [0, 1, 6, 9] | `$`             | Reduce E -> E + T (R1) |
| [0, 1]      | `$`             | **Accept**             |

Yes, the string `id + id * id` is **accepted** by the grammar.

---

### b) What are the various conflicts that occur during shift reduce parsing?

Shift-reduce parsers (like SLR, LALR, CLR) work by shifting input tokens onto a stack until the top of the stack matches the right-hand side of a grammar production (a handle). They then reduce, replacing the handle with the non-terminal from the left-hand side. Conflicts arise when the parser cannot decide whether to shift or to reduce, or which reduction to perform.

There are two main types of conflicts:

#### 1. Shift-Reduce (S/R) Conflict

A shift-reduce conflict occurs when the parser, looking at the current state on top of the stack and the next input token, cannot decide whether to:
a. **Shift** the input token onto the stack.
b. **Reduce** using a completed grammar rule that is on top of the stack.

*   **Cause:** This conflict is typically caused by ambiguity in the grammar.
*   **Classic Example: The "Dangling Else" Problem**
    *   **Grammar:** `stmt -> if expr then stmt | if expr then stmt else stmt`
    *   **Situation:** The parser has `if expr then stmt` on the stack and the next input token is `else`.
    *   **The Conflict:**
        *   **Shift:** The parser can shift `else`, assuming it belongs to the current `if...then...stmt` structure. This corresponds to matching the `if...then...else` production.
        *   **Reduce:** The parser can reduce the `if...then...stmt` on the stack using the rule `stmt -> if expr then stmt`. This would imply the `else` belongs to an outer `if` statement.
    *   **Resolution:** Most parser generators resolve this conflict in favor of **shifting**, which correctly associates the `else` with the innermost `if`.

#### 2. Reduce-Reduce (R/R) Conflict

A reduce-reduce conflict occurs when the parser, looking at the current state, finds that two or more different grammar rules could be applied to reduce the same set of symbols on top of the stack.

*   **Cause:** This is a more severe problem, almost always indicating a serious ambiguity in the grammar. It means the language syntax is not uniquely defined for that sequence of tokens.
*   **Example:**
    *   **Grammar:**
        ```
        stmt -> id ( params )   // A procedure call
        expr -> id ( params )   // A function call
        ```
    *   **Situation:** The parser has `id ( params )` on top of the stack.
    *   **The Conflict:** Should it reduce by `stmt -> id ( params )` or by `expr -> id ( params )`? The parser has no way to know from the syntax alone.
    *   **Resolution:** This conflict usually cannot be resolved automatically. The grammar must be rewritten to remove the ambiguity, perhaps by merging `stmt` and `expr` or by using different keywords for procedure and function calls. YACC/Bison will typically resolve it by reducing with the rule that appears **first** in the grammar specification, but this is often not what the language designer intended. 