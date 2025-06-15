# Answer to Question 4

---

### (a) Define the following with an example. i) Synthesized attributes ii) Annotated parse tree iii) Dependency graph

#### i) Synthesized Attributes

A **synthesized attribute** is an attribute whose value at a node in a parse tree is computed from the values of attributes at the **children** of that node. Synthesized attributes are used to pass information up the parse tree, from the leaves towards the root. They are associated with the non-terminal on the left-hand side (LHS) of a production. Grammars that exclusively use synthesized attributes are called S-attributed grammars.

**Example:**
Consider a simple grammar for evaluating an expression.
**Production:** `E -> E₁ + T`
**Semantic Rule:** `E.value = E₁.value + T.value`

Here, `E.value` is a synthesized attribute. Its value is calculated by adding the `value` attributes of its children, `E₁` and `T`. The result of the addition is passed up to the parent node `E`.

#### ii) Annotated Parse Tree

An **annotated parse tree** is a parse tree showing the values of the attributes at each node, computed from the semantic rules. It is a visual representation of the result of attribute evaluation. Each node is annotated with the final values of its attributes.

**Example:**
For the grammar rule `E -> E₁ + T { E.val = E₁.val + T.val }` and the string `3+4`, the annotated parse tree would show the `val` attribute at each node.

```
      E (val=7)
      / | \
     /  |  \
 E₁(val=3)  +   T(val=4)
    |           |
    |           |
 T(val=3)    F(val=4)
    |           |
    |           |
 F(val=3)     num(val=4)
    |
    |
  num(val=3)
```
The tree clearly shows the synthesized attribute `val` being passed up the tree, with the root `E` holding the final result, 7.

#### iii) Dependency Graph

A **dependency graph** is a directed graph that illustrates the flow of information among the attribute instances in a parse tree. It helps visualize the interdependencies between attributes and is used to determine a valid evaluation order.

*   **Nodes:** The graph has a node for each attribute instance associated with a node in the parse tree.
*   **Edges:** An edge is drawn from the node for attribute `b` to the node for attribute `a` if the value of `b` is needed to compute the value of `a`.

**Example:**
Consider a grammar for type declarations with both synthesized and inherited attributes:
**Production:** `D -> T L`
**Semantic Rule:** `L.in = T.type` (where `L.in` is inherited and `T.type` is synthesized)

Parse Tree snippet for this rule:
```
      D
     / \
    /   \
   T     L
```
The dependency graph for the attributes in this production would be:
```
           T.type  (synthesized)
               |
               v
           L.in    (inherited)
```
This edge from `T.type` to `L.in` shows that the value of `L.in` depends on the value of `T.type`. A topological sort of this graph gives a valid order for attribute evaluation (`T.type` must be computed before `L.in`).

---

### (b) Generate the three address code for the following code segment

**Code Segment:**
```
While (a < c and b < d) do
  If a=1 then c=c+1.
  Else
    While (a <= 4) do a = a + 3
```

**Three-Address Code (TAC):**
We use labels to manage the control flow for the `while` loops and the `if-else` statement.

```
L_WHILE_START:
    // Condition for the outer while loop
    t1 = a < c
    t2 = b < d
    if t1 == 0 goto L_WHILE_END // if !(a < c) exit
    if t2 == 0 goto L_WHILE_END // if !(b < d) exit

    // Body of the outer while loop
    if a != 1 goto L_ELSE

    // 'then' block
    t3 = c + 1
    c = t3
    goto L_WHILE_START // Loop back

L_ELSE:
    // 'else' block containing the inner while loop
L_INNER_WHILE_START:
    if a > 4 goto L_INNER_WHILE_END

    // Body of the inner while loop
    t4 = a + 3
    a = t4
    goto L_INNER_WHILE_START // Loop back

L_INNER_WHILE_END:
    goto L_WHILE_START // Continue outer loop

L_WHILE_END:
    // Code after the loop
```
This TAC translation uses temporary variables (`t1`, `t2`, etc.) and labels to represent the logic of the nested control structures in a linear, instruction-by-instruction format suitable for further processing by the compiler. 