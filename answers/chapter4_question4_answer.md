# Backpatching Explained

This document explains the concept of backpatching, a code generation technique used to handle forward jumps in a single pass.

---

## 1. The Problem: Forward Jumps

When a compiler generates code for control flow statements like `if-else`, `while`, or boolean expressions, it often needs to produce **forward jumps**. For example, in an `if (condition) then { ... }` statement, the jump that skips the `then` block when the condition is false points to a location that is not yet known.

Consider the boolean expression `A or B`. In Three-Address Code (TAC), this is often translated using short-circuit evaluation:
```
    if A goto L_true
    if B goto L_true
    goto L_false
L_true:
    ... // code for when the expression is true
L_false:
    ... // code for when the expression is false
```
When the parser generates the `goto L_true` instruction, it doesn't yet know the address of `L_true`.

A two-pass approach could solve this: the first pass generates code with placeholders for labels, and the second pass fills in the correct addresses. However, this is inefficient.

## 2. The Solution: Backpatching

**Backpatching** is a one-pass technique that addresses this problem by generating code with "holes" (missing label addresses) and keeping track of these holes in lists. When the target address of a jump is finally determined, the compiler "backpatches" the code by going back and filling in the address in all the instructions on the corresponding list.

### How it Works

The process uses three data structures associated with non-terminals in the grammar:

1.  **`truelist`**: A list of the addresses of all incomplete jump instructions that should go to the `true` label (the start of the code to be executed if the condition is true).
2.  **`falselist`**: A list of the addresses of all incomplete jump instructions that should go to the `false` label.
3.  **`nextlist`**: A list of addresses of `goto` instructions that should jump to the instruction immediately following the current statement. This is used for sequencing statements (`S1; S2`).

Semantic actions in the SDT are used to manage these lists and perform the backpatching.

*   `makelist(i)`: Creates a new list containing only `i`, the address of an instruction.
*   `merge(p1, p2)`: Concatenates two lists, `p1` and `p2`, and returns the new merged list.
*   `backpatch(p, i)`: Inserts `i` as the target label for all the instructions in the list `p`.

---

## 3. Backpatching Example

Let's see how backpatching works for the statement `if (A or B) then S1 else S2`. We'll use a marker non-terminal `M` to capture the address of the next instruction.

**Grammar and Actions:**
```
P  -> S
S  -> if (B) M1 S1 N else M2 S2
B  -> B1 or M B2 | B1 and M B2 | not B1 | (B1) | id relop id | true | false
M  -> ε  { M.quad = nextquad } // M captures the address of the next instruction
N  -> ε  { N.nextlist = makelist(nextquad); emit('goto'); } // For skipping the 'else' part
```
*`nextquad` is a global variable holding the index of the next instruction to be generated.*

**Parsing `if (A or B) then S1 else S2`:**

1.  **Process `A`:**
    *   Generates TAC for `A`. Let's say `A` is `x < y`.
    *   `emit('if x < y goto _')` at address 100. The `_` is the hole.
    *   `emit('goto _')` at address 101.
    *   `A.truelist = makelist(100)`
    *   `A.falselist = makelist(101)`

2.  **Process `M` (between `or` and `B`):**
    *   The `M -> ε` production is reduced.
    *   `M.quad` is set to the address of the next instruction, which is 102.

3.  **Process `B`:**
    *   Generates TAC for `B`. Let's say `B` is `p > q`.
    *   `emit('if p > q goto _')` at address 102.
    *   `emit('goto _')` at address 103.
    *   `B.truelist = makelist(102)`
    *   `B.falselist = makelist(103)`

4.  **Reduce `A or M B` to `B_overall`:**
    *   The semantic action for `or` is:
        *   `backpatch(A.falselist, M.quad)`: This fills the hole at address 101. The instruction becomes `101: goto 102`. This implements the short-circuit logic: if A is false, we proceed to check B.
        *   `B_overall.truelist = merge(A.truelist, B.truelist)` -> `{100, 102}`
        *   `B_overall.falselist = B.falselist` -> `{103}`

5.  **Process `M1` (after `)`):**
    *   `M1.quad` is set to the next instruction address, 104.

6.  **Reduce `if (B_overall) M1 ...`:**
    *   `backpatch(B_overall.truelist, M1.quad)`: The `true` jumps are filled.
        *   `100: if x < y goto 104`
        *   `102: if p > q goto 104`
    *   The body of `S1` is now generated, starting at address 104.

7.  **Process `N` (after `S1`):**
    *   Assume `S1` generated code up to address 110.
    *   `N.nextlist = makelist(111)`
    *   `emit('goto _')` at address 111. This jump will skip the `else` block.

8.  **Process `M2` (after `else`):**
    *   `M2.quad` is set to the next address, 112.

9.  **Reduce the full `if-else` statement:**
    *   `backpatch(B_overall.falselist, M2.quad)`: The `false` jump is filled.
        *   `103: goto 112`
    *   The body of `S2` is now generated, starting at address 112.
    *   The final `S.nextlist` for the whole statement is `merge(S1.nextlist, N.nextlist, S2.nextlist)`. The jump at 111 would be patched later when we know where the statement after the `if-else` begins.

This single pass correctly generates the control flow code by maintaining and patching lists of incomplete jumps. 