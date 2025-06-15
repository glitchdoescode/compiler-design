# Answer to Question 6

---

### (a) Write an algorithm to construct a DAG from a basic block.

A **Directed Acyclic Graph (DAG)** is a data structure used to represent a basic block. It gives a clear picture of how the values computed within the block are used. Nodes in the DAG represent values, and operators are attached to them. This representation is useful for identifying common subexpressions and simplifying the code within the basic block.

#### Algorithm: DAG Construction from a Basic Block

**Input:** A basic block of three-address code instructions.
**Output:** A DAG representing the basic block.

**Data Structures:**
*   A list of nodes for the DAG. Each node can represent a variable, a constant, or an operation.
*   A lookup table (e.g., a hash map) called `node_map` that maps variable names to the DAG node that currently holds their most recent value.

**Algorithm:**
1.  Initialize `node_map` to be empty.
2.  For each three-address instruction `i` in the basic block (processed in order):

    a. **Case 1: Instruction is `x = y op z`**
        *   Find the nodes for `y` and `z`. If `y` or `z` are in `node_map`, use their corresponding nodes (`n_y`, `n_z`). If they are not in `node_map` (i.e., they are initial values or constants), create new leaf nodes for them and add them to the DAG. Let's call them `n_y` and `n_z`.
        *   Search the DAG for an existing node representing the operation `op` with children `n_y` and `n_z`.
        *   If such a node (`n_op`) exists (this is a common subexpression), use it.
        *   If not, create a new interior node `n_op` with the label `op` and attach `n_y` and `n_z` as its children. Add `n_op` to the DAG.
        *   Update `node_map` to map the variable `x` to this node `n_op`. That is, `node_map[x] = n_op`.

    b. **Case 2: Instruction is `x = y` (copy operation)**
        *   Find the node for `y` in `node_map` (or create a leaf if it doesn't exist). Let this be `n_y`.
        *   Update `node_map` to map `x` to this same node. That is, `node_map[x] = n_y`. No new node is created.

3.  After processing all instructions, the nodes in the DAG that are still associated with live variables in `node_map` are marked as "output nodes."

#### Example:
Consider the basic block:
```
t1 = a + b
t2 = t1 * c
t3 = a + b
t4 = t2 + t3
```
1.  **`t1 = a + b`:** Create leaves for `a` and `b`, a `+` node with `a` and `b` as children. Map `t1` to this `+` node.
2.  **`t2 = t1 * c`:** Get the node for `t1`. Create a leaf for `c`. Create a `*` node with `t1`'s node and `c` as children. Map `t2` to this `*` node.
3.  **`t3 = a + b`:** Get nodes for `a` and `b`. Search for a `+` node with these children. We find the one created in step 1. Map `t3` to this *same* node. (Common subexpression identified).
4.  **`t4 = t2 + t3`:** Get the node for `t2` and `t3`. Create a `+` node with them as children. Map `t4` to this new `+` node.

This algorithm efficiently builds the DAG, reusing existing nodes whenever a common subexpression is found.

---

### (b) What is S-attributed SDT & L-attributed SDT? Consider the following grammar and write the SDT rules for the given grammar.

#### Definitions

**S-Attributed SDT (Syntax-Directed Translation)**
An SDT is **S-attributed** if it uses **only synthesized attributes**. Synthesized attributes are computed from the attribute values of the children of a node in the parse tree. Since the evaluation order is a simple post-order traversal of the parse tree, S-attributed definitions can be evaluated by bottom-up parsers (like LR parsers) during the parsing process itself.

**L-Attributed SDT**
An SDT is **L-attributed** if, for every production `A -> X₁X₂...Xₙ`, each inherited attribute of a symbol `Xⱼ` on the right-hand side depends only on:
1.  The inherited attributes of the parent `A`.
2.  The attributes (synthesized or inherited) of the symbols to its **left** (`X₁`, `X₂`, ..., `Xⱼ₋₁`).
Synthesized attributes can be used without restriction. L-attributed definitions are more general than S-attributed ones and can be evaluated during a depth-first, left-to-right traversal of the parse tree. This makes them suitable for use with top-down (LL) parsers. Every S-attributed SDT is also L-attributed.

#### SDT Rules for the Given Grammar

**Grammar:**
```
S -> S + A
S -> A
A -> A + B
A -> B
B -> (S)
B -> id
```
This grammar defines simple arithmetic expressions with addition, where parentheses can be used for grouping and `id` represents an operand. The goal of the SDT will be to calculate the value of the expression. We will use a synthesized attribute `val` for each non-terminal to store the computed value. Since we are only using a synthesized attribute, this will be an **S-attributed SDT**.

**SDT Rules:**

| Production | Semantic Rule                                  | Description                                                                                                   |
|:-----------|:-----------------------------------------------|:--------------------------------------------------------------------------------------------------------------|
| `S -> S₁ + A` | `{ S.val = S₁.val + A.val }`                   | The value of the expression is the sum of the values of the two sub-expressions.                              |
| `S -> A`      | `{ S.val = A.val }`                            | Pass the value up from `A` to `S`.                                                                            |
| `A -> A₁ + B` | `{ A.val = A₁.val + B.val }`                   | Similar to the `S` production, this handles addition at the `A` level.                                      |
| `A -> B`      | `{ A.val = B.val }`                            | Pass the value up from `B` to `A`.                                                                            |
| `B -> (S)`    | `{ B.val = S.val }`                            | The value of a parenthesized expression is the value of the expression inside the parentheses.                |
| `B -> id`     | `{ B.val = id.lexval }`                        | The base case. The value is the lexical value of the identifier (assuming `id.lexval` is provided by the lexer). |

This set of rules defines a complete S-attributed SDT for evaluating the expressions generated by the grammar. Each rule computes the `val` of the non-terminal on the LHS using only the `val` attributes of the symbols on the RHS, which is the definition of a synthesized attribute. 