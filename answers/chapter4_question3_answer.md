# Three-Address Code (TAC)

This document explains the concept of Three-Address Code (TAC) and provides TAC translations for two given code segments.

---

## a) Three-Address Code Explained

**Three-Address Code (TAC)** is a type of intermediate representation (IR) used by compilers. It is a high-level, machine-independent representation of the source code that is close to assembly language. Its defining characteristic is that each instruction can have **at most three addresses** (or operands). An "address" can be a variable name, a constant, or a compiler-generated temporary variable.

A typical TAC instruction has the form:
`result = operand1 operator operand2`

For example, the expression `x = (a + b) * (c - d)` would be broken down into a sequence of TAC instructions:
```
t1 = a + b
t2 = c - d
t3 = t1 * t2
x = t3
```
Here, `t1`, `t2`, and `t3` are temporary variables created by the compiler.

### Common Types of TAC Instructions

1.  **Assignment:** `x = y op z` (binary) or `x = op y` (unary).
2.  **Copy:** `x = y`.
3.  **Unconditional Jump:** `goto L`.
4.  **Conditional Jump:** `if x relop y goto L`. Jumps to label `L` if the condition is true.
5.  **Procedure Calls:** `param x`, `call p, n`, and `y = call p, n` (for functions returning a value).
6.  **Indexed Assignment:** `x = y[i]` and `x[i] = y`.
7.  **Address and Pointer:** `x = &y`, `x = *y`, and `*x = y`.

TAC is a beneficial IR because it is simple, easy to generate from the source code, and can be straightforwardly translated into machine code for a target architecture. It is also suitable for many code optimization techniques.

---

## b) Generating Three-Address Code

### Code Segment 1

**Source Code:**
```c
while (a < c AND b < d) do
  if (a = 1) then
    c = c + 1
  else
    while (a <= 4) do
      a = a + 3
```

**Three-Address Code Translation:**
We use labels to control the flow of execution for the loops and conditional statements.

```
L1: // Start of the outer while loop
    if (a >= c) goto L_exit_outer
    if (b >= d) goto L_exit_outer
    // Body of the outer while loop starts here
    if (a != 1) goto L_else
    // 'if' block
    t1 = c + 1
    c = t1
    goto L_continue_outer
L_else:
    // 'else' block containing the inner while loop
L2: // Start of the inner while loop
    if (a > 4) goto L_exit_inner
    // Body of the inner while loop
    t2 = a + 3
    a = t2
    goto L2 // Continue inner loop
L_exit_inner:
L_continue_outer:
    goto L1 // Continue outer loop
L_exit_outer:
```

### Code Segment 2

**Source Code:**
`P := (K + Y) + (K - C)`

**Three-Address Code Translation:**
This is a simple arithmetic expression. We use temporary variables to hold intermediate results.

```
t1 = K + Y
t2 = K - C
t3 = t1 + t2
P = t3
```
**Note:** A smart compiler might recognize the common subexpression `K` but cannot optimize further here because it's used with different operands (`Y` and `C`). If the expression were `(K + Y) + (K + Y)`, the common subexpression `t1 = K + Y` could be computed once and reused. 