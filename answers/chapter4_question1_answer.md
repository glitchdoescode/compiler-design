# Attributes, SDTs, and Dependency Graphs

This document explains key concepts in Syntax-Directed Translation (SDT), including synthesized and inherited attributes, S-attributed and L-attributed SDTs, and the tools used to visualize them: annotated parse trees and dependency graphs.

---

## a) Differentiating Synthesized and Inherited Attributes

In a Syntax-Directed Definition (SDD), attributes are values associated with grammar symbols. They carry information up or down a parse tree. There are two types of attributes.

| Feature         | Synthesized Attributes                                                                    | Inherited Attributes                                                                              |
|:----------------|:------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|
| **Definition**  | An attribute whose value at a node in the parse tree is computed from the values of attributes at the **children** of that node. | An attribute whose value at a node is computed from the values of attributes at the node's **parent** and/or **siblings**. |
| **Information Flow** | **Bottom-up:** Information flows up the parse tree from the leaves towards the root. | **Top-down/Sideways:** Information flows down the parse tree from the root and/or across from siblings. |
| **Typical Use** | Evaluating expressions, constructing syntax trees, type checking. The result at a parent node is synthesized from its parts. | Propagating contextual information, such as the data type of a declaration to all identifiers in that declaration. |
| **Example**     | In `E -> E1 + T`, the value of `E` (`E.val`) is synthesized by adding the values of its children: `E.val = E1.val + T.val`. | In `D -> T L`, the type `T.type` is passed down to `L` (`L.in = T.type`) so `L` knows the type for the identifiers it represents. |
| **Association** | Associated with non-terminals on the **left-hand side (LHS)** of a production.              | Associated with non-terminals on the **right-hand side (RHS)** of a production. Can also be associated with terminals. |

### Example: A Grammar with Both Attribute Types

Consider a grammar for declaring typed variables:
```
D -> T L      { L.in = T.type } // Inherited: type is passed down
T -> int      { T.type = integer } // Synthesized
T -> float    { T.type = float }   // Synthesized
L -> L1, id   { L1.in = L.in; addType(id.entry, L.in) } // Inherited
L -> id       { addType(id.entry, L.in) }
```
*   `T.type` is **synthesized**. Its value is determined by its child (`int` or `float`).
*   `L.in` is **inherited**. Its value is determined by its parent (`D`) and sibling (`T`). It's used to pass the type down the list of identifiers.

---

## b) S-Attributed and L-Attributed SDTs

These are two important classes of Syntax-Directed Definitions, defined by the types of attributes they use.

### S-Attributed SDT

An SDD is **S-attributed** if it **only uses synthesized attributes**.
*   **Evaluation Order:** The attributes can be evaluated in a single **bottom-up traversal** of the parse tree (a post-order traversal).
*   **Implementation:** S-attributed definitions are easy to implement with bottom-up parsing techniques like LR parsing. Semantic rules can be evaluated as soon as a production is reduced.

### L-Attributed SDT

An SDD is **L-attributed** if its attributes (which can be both synthesized and inherited) can be evaluated in a **single depth-first, left-to-right traversal** of the parse tree.
For a production `A -> X1 X2 ... Xn`, an inherited attribute for a symbol `Xj` on the right-hand side can only depend on:
1.  Inherited attributes of the parent `A`.
2.  Attributes (either inherited or synthesized) of the symbols to its left, `X1, X2, ..., Xj-1`.

**Key Points:**
*   S-attributed definitions are a proper subset of L-attributed definitions.
*   L-attributed definitions are more powerful than S-attributed ones because they allow for top-down and sideways information flow.
*   They are well-suited for top-down parsing (like LL parsing) because the evaluation order matches the parsing order.

---

## c) Annotated Parse Trees and Dependency Graphs

These are graphical tools used to visualize attribute evaluation.

### Annotated Parse Tree

An **annotated parse tree** is a parse tree showing the values of the attributes at each node. It is the result of the attribute evaluation process.

**Example:**
Consider the S-attributed grammar `E -> E1 + T { E.val = E1.val + T.val }` and the string `3+4`.

```
      E (val=7)
      / | \
     /  |  \
 E1(val=3)  +   T(val=4)
    |           |
    |           |
 T(val=3)    F(val=4)
    |           |
    |           |
 F(val=3)     num(4)
    |
    |
  num(3)
```
This tree is annotated with the `val` attribute at each node, showing the result of the bottom-up evaluation.

### Dependency Graph

A **dependency graph** shows the flow of information among the attribute instances in a particular parse tree. It helps determine the evaluation order for attributes.

*   **Nodes:** For each attribute instance in the parse tree, we create a node.
*   **Edges:** If an attribute `b`'s value is used to compute an attribute `a`'s value, we draw a directed edge from the node for `b` to the node for `a`.

**Example:**
Using the inherited attribute example `D -> T L { L.in = T.type }`.

Parse Tree:
```
      D
     / \
    /   \
   T     L
   |
  int
```

Dependency Graph for the `D -> T L` production:
```
      T.type  ----->  L.in
```
This graph shows that the value of `L.in` depends on the value of `T.type`. The evaluation order must respect the graph's topology (e.g., using a topological sort). An evaluation order is valid if it computes each attribute only after all the attributes it depends on have been computed. 