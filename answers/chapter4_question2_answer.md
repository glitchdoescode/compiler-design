# Syntax-Directed Translation Schemes

This document provides solutions to two problems involving Syntax-Directed Translation (SDT) schemes: evaluating an expression with a given SDT and creating an SDT to convert infix expressions to postfix.

---

## 2a: Expression Evaluation with a Given SDT

**Grammar and Semantic Rules:**
```
E -> E1 # T      { E.value = E1.value * T.value }
E -> T           { E.value = T.value }
T -> T1 & F      { T.value = T1.value + F.value }
T -> F           { T.value = F.value }
F -> num         { F.value = num.value }
```

**Input Expression:** `2 & 3 & 5 # 6 & 4`

### Analysis of the Grammar

The grammar specifies the following operator precedence and associativity:
*   `#` is the operator for `E` productions. It is left-associative (from `E -> E1 # T`).
*   `&` is the operator for `T` productions. It is left-associative (from `T -> T1 & F`).
*   Since `E` derives `T`, but `T` does not derive `E`, the `#` operator has **lower precedence** than the `&` operator.

Therefore, the expression should be parsed as: `(2 & 3 & 5) # (6 & 4)`.

### Evaluation

Let's evaluate the two parenthesized sub-expressions first.

1.  **Evaluate `(2 & 3 & 5)`:**
    *   This corresponds to the `T` production `T -> T1 & F`. Due to left-associativity, it's `(2 & 3) & 5`.
    *   `2 & 3` evaluates to `2 + 3 = 5`.
    *   The expression becomes `5 & 5`, which evaluates to `5 + 5 = 10`.
    *   So, `(2 & 3 & 5).value = 10`.

2.  **Evaluate `(6 & 4)`:**
    *   This also uses `T -> T1 & F`.
    *   `6 & 4` evaluates to `6 + 4 = 10`.
    *   So, `(6 & 4).value = 10`.

3.  **Evaluate the final expression:**
    *   The expression is now `10 # 10`.
    *   This corresponds to the `E` production `E -> E1 # T`.
    *   `10 # 10` evaluates to `10 * 10 = 100`.

The final value of `E.value` for the expression `2 & 3 & 5 # 6 & 4` is **100**.

### Annotated Parse Tree

The parse tree below is annotated with the `.value` attribute, showing the bottom-up flow of the calculation.

```
                      E (value=100)
                      /   |   \
                     /    |    \
            E1 (value=10) #    T (value=10)
                 |               |
                 |               |
             T (value=10)       T1 (value=10) & F (value=4)
                 |                    |             |
                 |                    |             |
        T1 (value=5) & F (value=5) F (value=6)    num(4)
             |             |          |
             |             |          |
     T (value=2) & F (value=3) num(5)   num(6)
          |            |
          |            |
      F(value=2)     num(3)
          |
          |
        num(2)
```

---

## 2b: SDT for Infix to Postfix Translation

We need to create an SDT that prints the postfix equivalent of an infix arithmetic expression. The standard grammar for arithmetic expressions is used. The idea is to print the operands as they are seen and to print the operators only after both of their operands have been processed.

**Grammar:**
```
E -> E + T | E - T | T
T -> T * F | T / F | F
F -> (E) | id
```

**Syntax-Directed Translation Scheme:**

The semantic rules are embedded within the productions as actions `{...}`. The `print` action appends to an output buffer.

```
Production        | Semantic Rule (Action)
------------------|----------------------------------------------------------
E -> E1 + T       | { E1.code; T.code; print('+'); }
E -> E1 - T       | { E1.code; T.code; print('-'); }
E -> T            | { T.code; }
                  |
T -> T1 * F       | { T1.code; F.code; print('*'); }
T -> T1 / F       | { T1.code; F.code; print('/'); }
T -> F            | { F.code; }
                  |
F -> (E)          | { E.code; }
F -> id           | { print(id.name); }
```

**Explanation:**
*   This is an **L-attributed definition**. The actions are placed at the end of the productions, meaning they are executed after all children have been processed. This works for S-attributed definitions as well.
*   When a production like `E -> E1 + T` is reduced, the parser has already processed the code for `E1` and `T`. The action then simply prints the operator `+` after them.
*   For a terminal `id`, its name is printed immediately.
*   For parentheses `(E)`, they only serve to group the expression; nothing is printed for them, we just process the sub-expression `E` inside.

**Example Walkthrough:**
Let's trace the translation of `a + b * c`.

1.  The parse starts with `E`. It eventually reaches `a`, which is an `id`. `a` is printed.
2.  The parser sees `+`, but the rule `E -> E + T` requires `T` to be processed first.
3.  The parser processes the `T` part, which is `b * c`.
4.  It finds `b`, an `id`. `b` is printed.
5.  It sees `*`, but the rule `T -> T * F` requires `F` (`c`) to be processed.
6.  It finds `c`, an `id`. `c` is printed.
7.  Now the `T -> T * F` production for `b * c` can be reduced. Its action is to print `*`.
8.  Now the `E -> E + T` production for `a + (b*c)` can be reduced. Its action is to print `+`.

**Final Output:** `a b c * +`
This matches the correct postfix notation. 