# Conflicts in Shift-Reduce Parsing

This document explains the types of conflicts that can arise during shift-reduce parsing, a common bottom-up parsing technique used by LR parsers.

---

## 1. Introduction to Shift-Reduce Parsing

A **shift-reduce parser** is a bottom-up parser that attempts to construct a parse tree for an input string by starting at the leaves and working up towards the root (the start symbol). It uses a stack to hold grammar symbols and an input buffer. At each step, the parser makes one of two fundamental decisions:

1.  **Shift:** Move the next input token from the input buffer onto the top of the stack.
2.  **Reduce:** If the top symbols on the stack match the right-hand side (RHS) of a grammar production (this sequence of symbols is called a "handle"), replace them with the non-terminal from the left-hand side (LHS) of that production.

A **conflict** occurs when the parser, given the current state and the next input token, cannot uniquely decide whether to shift or to reduce, or it cannot decide which of several possible reductions to perform. These conflicts arise from ambiguities in the grammar.

There are two main types of conflicts in shift-reduce parsing:

1.  **Shift-Reduce Conflict**
2.  **Reduce-Reduce Conflict**

---

## 2. Shift-Reduce Conflict

**Definition:**
A **shift-reduce conflict** occurs when the parser has a choice between shifting the next input token onto the stack or reducing a handle that is already on top of the stack.

This conflict typically arises from grammars where a production could end with the same terminal that another production could begin with.

**Example: The "Dangling Else" Problem**

A classic example is the grammar for `if-then-else` statements, which is ambiguous:
```
(1) stmt -> if expr then stmt
(2) stmt -> if expr then stmt else stmt
```

Consider the parser's state when processing the string `if E1 then if E2 then S1 else S2`.
At some point, the stack and input buffer might look like this:

*   **Stack:** `... if E1 then if E2 then S1`
*   **Input:** `else ...`

Now the parser has a choice:

1.  **Shift:** The next input token is `else`. The parser could shift `else` onto the stack, with the intention of eventually using production (2) (`if expr then stmt else stmt`) to reduce the outer `if`.

2.  **Reduce:** The handle `if expr then stmt` (corresponding to `if E2 then S1`) is on top of the stack. The parser could reduce this handle using production (1) (`stmt -> if expr then stmt`). This would mean the `else` belongs to the outer `if E1`.

Since the parser has a valid choice to either shift or reduce, this is a **shift-reduce conflict**.

**Resolution:**
Parser generators like YACC/Bison resolve this conflict by default in favor of **shifting**. This default behavior correctly associates the `else` with the closest preceding `if`, which is the standard convention in most programming languages.

---

## 3. Reduce-Reduce Conflict

**Definition:**
A **reduce-reduce conflict** occurs when the parser has a choice between reducing the handle on top of the stack using two or more different production rules.

This conflict arises when two distinct productions have the same right-hand side, or when the end of one production's RHS is also the complete RHS of another production.

**Example:**
Consider a grammar for a simplified programming language that includes procedure calls and variable assignments from an expression.
```
(1) stmt -> id ( expr_list )   // Procedure call
(2) expr -> id ( expr_list )   // Function call within an expression
(3) stmt -> expr               // An expression can be a statement
```
This grammar is problematic, but let's consider a simpler, more direct example of a reduce-reduce conflict.

Suppose a grammar has the following productions:
```
(1) A -> x
(2) B -> x
```
And the parser is in a state where the stack top is `...y` and the next input is `z`, and it has determined that `x` is a handle.
*   **Stack:** `...y x`
*   **Input:** `z...`

Now, the parser sees the handle `x` on top of the stack and must reduce. But it has two choices:

1.  **Reduce by `A -> x`**: Replace `x` on the stack with `A`.
2.  **Reduce by `B -> x`**: Replace `x` on the stack with `B`.

The parser cannot decide which reduction to perform. This is a **reduce-reduce conflict**. It indicates a serious ambiguity in the grammar.

**Resolution:**
Parser generators like YACC/Bison resolve this conflict by default by choosing the production that appears **first** in the grammar specification file. However, unlike a shift-reduce conflict, a reduce-reduce conflict almost always indicates a flaw in the grammar's design that should be fixed by the developer rather than relying on the default resolution. 