# Top-Down vs. Bottom-Up Parsing

This document differentiates between top-down and bottom-up parsing strategies, provides examples, and explains the concepts of backtracking and non-backtracking parsers.

---

## 1. Differentiating Top-Down and Bottom-Up Parsing

Parsing is the process of analyzing a string of tokens to determine its grammatical structure with respect to a given formal grammar. The two fundamental strategies for parsing are top-down and bottom-up.

| Feature                      | Top-Down Parsing                                                                                                | Bottom-Up Parsing                                                                                             |
| :--------------------------- | :-------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------ |
| **Starting Point**           | Starts with the **start symbol** of the grammar.                                                                | Starts with the **input string** of tokens.                                                                   |
| **Goal**                     | Tries to derive the input string by applying production rules forward, expanding non-terminals.                   | Tries to reduce the input string back to the start symbol by applying production rules in reverse.          |
| **Construction of Parse Tree** | Builds the parse tree from the **root down to the leaves**.                                                     | Builds the parse tree from the **leaves up to the root**.                                                     |
| **Derivation Type**          | Corresponds to finding a **leftmost derivation (LMD)** for the input string.                                    | Corresponds to finding a **rightmost derivation (RMD) in reverse**.                                           |
| **Grammar Constraints**      | Cannot handle left-recursive grammars. Requires left-factoring for predictive parsers.                          | Can handle left-recursive grammars. Cannot handle some ambiguous grammars or those with reduce-reduce conflicts. |
| **Common Parsers**           | Recursive Descent Parser, LL(1) Parser.                                                                         | Shift-Reduce Parser, LR Parsers (SLR, CLR, LALR), Operator-Precedence Parser.                                  |
| **Primary Challenge**        | Deciding which production rule to use for a non-terminal to match the input.                                    | Deciding when to **shift** (consume input) and when to **reduce** (apply a grammar rule).                     |

---

## 2. Example of Parsing Strategies

Let's use a simple grammar and a string to illustrate the two approaches.

**Grammar:**
```
S -> aAB
A -> b
B -> c
```
**Input String:** `abc`

### a) Example: Top-Down Parsing

The goal is to start with `S` and derive `abc`.

1.  **Start with `S`**: The only production for `S` is `S -> aAB`. So, we expand `S` to `aAB`.
    *   *Parse Tree starts:* A root `S` with children `a`, `A`, `B`.
    *   *Match:* The 'a' in the production matches the first character of the input string `abc`. We now need to match `AB` with the rest of the string, `bc`.
2.  **Expand `A`**: The current non-terminal is `A`. The only production is `A -> b`. We apply it.
    *   *Parse Tree:* The `A` node gets a child `b`.
    *   *Match:* The 'b' from the rule matches the next character of the input. We now need to match `B` with the remaining string, `c`.
3.  **Expand `B`**: The current non-terminal is `B`. The only production is `B -> c`. We apply it.
    *   *Parse Tree:* The `B` node gets a child `c`.
    *   *Match:* The 'c' from the rule matches the final character of the input.
4.  **Finish**: The derived string matches the input string exactly. The parse is successful.

This process traces out a **leftmost derivation**: `S => aAB => abB => abc`.

### b) Example: Bottom-Up Parsing

The goal is to start with `abc` and reduce it back to `S`. This is typically done with a stack in a shift-reduce parser.

1.  **Start**: Stack is empty, input is `abc$`.
2.  **Shift 'a'**: Read 'a' from input and push it onto the stack. Stack: `[a]`, Input: `bc$`.
3.  **Shift 'b'**: Read 'b'. Stack: `[a, b]`, Input: `c$`.
4.  **Reduce**: The top of the stack, `b`, matches the right-hand side of the production `A -> b`. We **reduce** by replacing `b` with `A`. Stack: `[a, A]`, Input: `c$`.
5.  **Shift 'c'**: Read 'c'. Stack: `[a, A, c]`, Input: `$`.
6.  **Reduce**: The top of the stack, `c`, matches the RHS of `B -> c`. Reduce by replacing `c` with `B`. Stack: `[a, A, B]`, Input: `$`.
7.  **Reduce**: The top of the stack, `aAB`, matches the RHS of `S -> aAB`. Reduce by replacing `aAB` with `S`. Stack: `[S]`, Input: `$`.
8.  **Accept**: The stack contains only the start symbol `S` and the input is empty. The parse is successful.

This process traces a **rightmost derivation in reverse**: `abc <= abB <= aAB <= S`.

---

## 3. Backtracking vs. Non-Backtracking Parsers

This distinction primarily applies to top-down parsers. It relates to how a parser handles situations where there are multiple production choices for a single non-terminal.

### a) Backtracking Parsers

A **backtracking parser** is a type of parser that explores production choices speculatively. If a chosen production leads to a dead end (i.e., it fails to derive the input string), the parser "backtracks" to its previous state and tries the next available production choice.

*   **Mechanism:** It works like a depth-first search through the possible derivations.
*   **Example:** A simple Recursive Descent Parser can be implemented with backtracking.
*   **Advantages:**
    *   Conceptually simple and easy to implement by hand.
    *   Can handle a wider range of grammars than non-backtracking parsers, including some ambiguous or non-left-factored ones.
*   **Disadvantages:**
    *   **Inefficiency:** The process of repeated parsing and backtracking can be very slow, potentially leading to exponential time complexity.
    *   **Poor Error Reporting:** It's difficult to provide precise error messages, as the parser only knows a parse has failed after trying all possibilities.
    *   **Cannot handle left-recursion.**

### b) Non-Backtracking Parsers (Predictive Parsers)

A **non-backtracking parser**, also known as a **predictive parser**, commits to a single production choice for a non-terminal based on the next one (or k) input token(s). It does not go back and try alternatives.

*   **Mechanism:** It uses a lookahead token to "predict" which production rule to apply. If it makes a wrong choice, it's a syntax error; there is no backtracking.
*   **Example:** An LL(1) parser is a non-backtracking, predictive parser. "LL(1)" stands for **L**eft-to-right scan, **L**eftmost derivation, and **1** token of lookahead.
*   **Advantages:**
    *   **Efficiency:** These parsers are very fast, typically running in linear time, O(n), where n is the length of the input.
    *   **Good Error Reporting:** Since they don't backtrack, they can report a syntax error as soon as they encounter a situation where the lookahead token cannot be produced by any valid rule.
*   **Disadvantages:**
    *   **Restrictive Grammar Requirements:** The grammar must be suitable for predictive parsing. It must not be left-recursive and must be left-factored. For an LL(1) parser, the grammar must be LL(1), meaning the choice of production for any non-terminal must be uniquely determinable by the next single input token. 