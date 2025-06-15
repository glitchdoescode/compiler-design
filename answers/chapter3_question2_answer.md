# Key Concepts in Grammars

This document explains three important concepts related to grammars: Ambiguity, Left Recursion, and Left Factoring, each with suitable examples.

---

## a) Ambiguity

**Definition:**
A grammar is said to be **ambiguous** if there exists at least one string for which the grammar can produce more than one distinct parse tree. This means the string has more than one leftmost derivation (LMD) or more than one rightmost derivation (RMD).

Ambiguity is an undesirable property for programming language grammars because it implies that a single statement could be interpreted in multiple ways, leading to unpredictable program behavior. When an ambiguous grammar is used with a parser, the parser may not know which production rule to apply at a certain point.

**Example: The "Dangling Else" Problem**

A classic example of ambiguity is the "dangling else" problem. Consider the following grammar for conditional statements:

```
stmt -> if (expr) stmt
      | if (expr) stmt else stmt
      | other_stmt
```

Now, let's analyze the string: `if (E1) if (E2) S1 else S2`

This string has two possible parse trees:

**Parse Tree 1: `else` belongs to the inner `if`**
This interpretation corresponds to `if (E1) { if (E2) S1 else S2 }`.

```
      stmt
        |
 if (E1) stmt
        / | \
       /  |  \
if (E2) stmt else stmt
         |        |
         S1       S2
```

**Parse Tree 2: `else` belongs to the outer `if`**
This interpretation corresponds to `if (E1) { if (E2) S1 } else S2`.

```
          stmt
      /    |     \
     /     |      \
if (E1)  stmt  else stmt
         |           |
  if (E2) stmt         S2
           |
           S1
```
Since there are two possible parse trees for the same string, the grammar is ambiguous. Most programming languages resolve this by defining a rule, typically that an `else` clause belongs to the nearest preceding unmatched `if`.

---

## b) Left Recursion

**Definition:**
A grammar is **left-recursive** if it contains a non-terminal `A` such that there is a derivation `A => Aα` for some string `α` of terminals and non-terminals. In simpler terms, a production is left-recursive if the leftmost symbol on its right-hand side is the same as the non-terminal on its left-hand side.

*   **Direct Left Recursion:** This occurs when a production is of the form `A -> Aα`.
*   **Indirect Left Recursion:** This occurs when the recursion happens through a chain of productions, such as `A -> Bx`, `B -> Cy`, `C -> Az`.

Left recursion is a problem for top-down parsers (like Recursive Descent or LL(1) parsers) because they can enter an infinite loop of recursive calls without consuming any input. Therefore, grammars must be transformed to eliminate left recursion before being used in a top-down parser.

**Example of Direct Left Recursion:**
Consider the standard grammar for arithmetic expressions:

```
E -> E + T | T
```

This grammar is directly left-recursive because of the production `E -> E + T`. A top-down parser trying to parse `E` might first apply `E -> E + T`, which would immediately lead it to parse `E` again, causing an infinite loop.

**Elimination of Direct Left Recursion:**
A production `A -> Aα | β` (where `β` does not start with `A`) can be transformed into the following equivalent non-left-recursive productions:

```
A  -> βA'
A' -> αA' | ε
```

Applying this to the example:
```
E  -> TA'
A' -> +TA' | ε
```
This new grammar generates the same language but is suitable for top-down parsing.

---

## c) Left Factoring

**Definition:**
**Left factoring** is a grammar transformation technique used to eliminate a different kind of ambiguity that poses problems for top-down parsers. It is required when two or more production rules for the same non-terminal have a common prefix on their right-hand sides.

When a parser encounters input that matches this common prefix, it cannot decide which production to choose until it has looked further ahead. LL(1) parsers, which are allowed to look only one token ahead, cannot handle such grammars. Left factoring involves "factoring out" the common prefix into a new production.

**Example of a Grammar Requiring Left Factoring:**
Consider the grammar for a conditional statement:

```
stmt -> if expr then stmt else stmt
      | if expr then stmt
```
Here, both productions for `stmt` start with the common prefix `if expr then stmt`.

**Transformation by Left Factoring:**
A set of productions `A -> αβ1 | αβ2 | ... | αβn` can be transformed by factoring out the common prefix `α`:

```
A  -> αA'
A' -> β1 | β2 | ... | βn
```

Applying this to the example:
The common prefix is `α = if expr then stmt`.
The remaining parts are `β1 = else stmt` and `β2 = ε` (since the second production has nothing after the prefix).

The transformed, left-factored grammar is:

```
stmt -> if expr then stmt A'
A'   -> else stmt | ε
```

Now, when the parser sees an `if`, it knows it must parse `expr`, `then`, and `stmt`. After that, it calls the procedure for `A'`, which then decides based on the next token (either `else` or something else, indicating the `ε` production) which rule to apply. This removes the uncertainty. 