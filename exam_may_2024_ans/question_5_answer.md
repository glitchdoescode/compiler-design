# Answer to Question 5

---

### a) What is a DAG? Draw the DAG for the statement `a = (a * b + c) - (a * b + c)`.

A **Directed Acyclic Graph (DAG)** is a data structure used in compilers to represent the structure of a basic block. It provides a more compact representation than a syntax tree by identifying and merging common subexpressions.

*   **Nodes:** Leaves represent identifiers, names, or constants. Interior nodes represent operators.
*   **Edges:** Edges point from an operator node to its operands.
*   **Acyclic:** There are no cycles, meaning a node cannot be its own ancestor.
*   **Benefit:** DAGs are a primary tool for local optimization, especially for **common subexpression elimination**. If an expression is calculated more than once, it will be represented by a single node in the DAG, making the redundancy obvious.

#### DAG for `a = (a * b + c) - (a * b + c)`

Let's construct the DAG step-by-step for the three-address code equivalent of the statement.

**Three-Address Code:**
```
1. t1 = a * b
2. t2 = t1 + c
3. t3 = a * b   // Redundant
4. t4 = t3 + c   // Redundant
5. t5 = t2 - t4
6. a = t5
```

**DAG Construction:**

1.  **`t1 = a * b`**: Create leaf nodes for `a` and `b`. Create an interior node for `*` with edges to the `a` and `b` nodes.
2.  **`t2 = t1 + c`**: Create a leaf node for `c`. Create an interior node for `+` with edges to the `*` node (from step 1) and the `c` node.
3.  **`t3 = a * b`**: The expression `a * b` has already been computed and is represented by the `*` node created in step 1. We reuse this node. No new nodes are created.
4.  **`t4 = t3 + c`**: The expression `(a * b) + c` is represented by the `+` node from step 2. We reuse this node. No new nodes are created.
5.  **`t5 = t2 - t4`**: We get the nodes for `t2` and `t4`. They are the *same* `+` node. Create a new `-` node with edges pointing to the `+` node for both its left and right children.
6.  **`a = t5`**: The variable `a` is now associated with the result of the `-` operation. The old node for `a` (the leaf) is "killed," and the new value of `a` is the root of the expression.

**Final DAG:**

The final DAG visually demonstrates that the two subexpressions `(a * b + c)` are identical, represented by a single `+` node. The subtraction `t2 - t4` is effectively `t2 - t2`, which an optimizer could further simplify to `0`.

```
      -
     / \
    /   \
   +<----
  / \
 /   \
*     c
/ \
a   b
```
The final assignment updates the label for `a` to point to the root `-` node.

---

### b) Explain synthesized attribute and inherited attribute with suitable examples.

**Synthesized** and **inherited attributes** are two types of attributes associated with grammar symbols in a Syntax-Directed Definition (SDD). They are used to pass semantic information up, down, and across a parse tree.

#### Synthesized Attributes

A synthesized attribute at a parse tree node is computed from the attribute values of its **children**. Information flows **up** the tree.

*   **Definition:** For a production `A -> XYZ`, a synthesized attribute `A.s` is computed using the attributes of `X`, `Y`, and `Z`.
*   **Use Case:** They are ideal for situations where you are "building" something up from the components, like calculating the value of an expression or constructing the intermediate code for a statement.
*   **Evaluation:** Can be evaluated by a single bottom-up (post-order) traversal of the parse tree.

**Example: Evaluating an Arithmetic Expression**

Consider the grammar:
`expr -> expr₁ + term | term`
`term -> num`

The `value` of an `expr` is synthesized from the values of its children.

**Production:** `expr -> expr₁ + term`
**Semantic Rule:** `expr.value = expr₁.value + term.value`

For the string `5 + 3`, the parse tree would be:
```
      expr (value = 8)
      / | \
     /  |  \
expr₁(value=5) + term(value=3)
      |          |
    term(value=5) num(lexval=3)
      |
    num(lexval=5)
```
The values from the `num` leaves (5 and 3) are passed up to the `term` nodes, then to the `expr` nodes. The final value `8` is synthesized at the root.

#### Inherited Attributes

An inherited attribute at a parse tree node is computed from the attribute values of its **parent** and/or its **siblings**. Information flows **down and/or sideways** across the tree.

*   **Definition:** For a production `A -> XYZ`, an inherited attribute `X.i` can be computed from the attributes of `A`, and `Y.i` can be computed from attributes of `A` and `X`.
*   **Use Case:** They are used to pass context-dependent information down the tree. A classic example is tracking the data type of variables in a declaration list.
*   **Evaluation:** Requires a more complex traversal, often a depth-first, left-to-right walk.

**Example: Processing Variable Declarations**

Consider the grammar for a declaration like `int i, j, k;`:
`decl -> type var_list`
`var_list -> id, var_list₁ | id`

We want all identifiers in the `var_list` to inherit the data `type`.

**Production:** `decl -> type var_list`
**Semantic Rule:** `var_list.type = type.lexval`  (The `var_list` inherits its type from `type`)

**Production:** `var_list -> id, var_list₁`
**Semantic Rules:**
1. `add_type(id.entry, var_list.type)` (Use the inherited type to update the symbol table for `id`)
2. `var_list₁.type = var_list.type` (Pass the inherited type to the rest of the list)

For `int i, j`, the tree would show the type `int` being passed down:
```
        decl
        /  \
       /    \
     type    var_list (type = 'int')
 (lexval='int') /   |   \
             /    |    \
            id, var_list₁ (type = 'int')
           (i)      |
                    id (j)
```
The `type` attribute is computed at the parent (`decl`) and passed down to `var_list`, which in turn passes it down to `var_list₁` and uses it to update the symbol table entries for `i` and `j`. 