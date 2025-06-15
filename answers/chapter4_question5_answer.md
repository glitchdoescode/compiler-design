# Directed Acyclic Graphs (DAGs)

This document explains Directed Acyclic Graphs (DAGs), their use in code optimization, provides an algorithm for their construction, and demonstrates their creation for several examples.

---

## a) What is a DAG and How is it Useful?

A **Directed Acyclic Graph (DAG)** is a data structure used by compilers to represent the structure of a **basic block**. A basic block is a sequence of consecutive instructions where control flow enters at the beginning and leaves at the end without any branching in or out.

In a DAG representation of a basic block:
*   **Leaf nodes** represent initial values of variables or constants.
*   **Interior nodes** represent the operators applied.
*   **Nodes** are given labels, which can be variables or temporary values.

The key feature of a DAG is that it **automatically identifies common subexpressions**. If an expression is computed multiple times, it will be represented by a single node in the DAG with multiple parents (i.e., multiple places where its result is used).

### Usefulness in Transformations on Basic Blocks

DAGs are highly useful for local optimizations within a basic block:

1.  **Common Subexpression Elimination:** This is the primary benefit. If a node already exists for an expression `a + b`, the compiler doesn't need to generate code to re-compute it. It can just use the result from the existing node.
2.  **Dead Code Elimination:** If a variable is assigned a value but that node in the DAG has no parents (meaning the variable is never used later in the block), the assignment is "dead" and the instruction can be removed.
3.  **Algebraic Simplification:** Expressions like `x + 0` or `y * 1` can be simplified directly during DAG construction.
4.  **Code Generation:** A DAG helps in generating efficient three-address code. By traversing the DAG, the compiler can determine an optimal ordering of instructions that minimizes the number of temporary variables needed.

---

## b) Algorithm to Construct a DAG from a Basic Block

This algorithm processes each three-address instruction `i` in a basic block to build the DAG. It uses a list of nodes, `NODES`, where `NODES[j]` points to the most recently created node for the value held by variable `j`.

**Input:** A basic block of three-address instructions.
**Output:** A DAG for the basic block.

**Algorithm:**
For each instruction `i` of the form `x = y op z` in the basic block:
1.  Find `node(y)`: Check if there's a node in the DAG for `y`. If not, create a new leaf node for `y` and let `node(y)` be this new node.
2.  Find `node(z)`: Similarly, find or create a leaf node for `z`, denoted `node(z)`.
3.  Find Operator Node: Search the DAG for an existing node representing the operator `op` with children `node(y)` and `node(z)`.
    *   If such a node `n` exists, use it.
    *   If not, create a new interior node `n` for `op`, with `node(y)` and `node(z)` as its children.
4.  Update Node List: Add variable `x` to the list of attached variables for node `n`. If `x` was previously attached to another node, remove it from that old node's list. Set `NODES[x] = n`.

For an instruction `x = y`:
1.  Find `node(y)`.
2.  Add `x` to the list of attached variables for `node(y)`. Set `NODES[x] = node(y)`.

After processing all instructions, any node whose attached variable list is empty and which corresponds to a temporary variable can be identified. Variables that have no live uses outside the block and are attached to nodes with no parents are dead code.

---

## c) Constructing DAGs for Example Statements

### Example 1: `a = (a * b + c) - (a * b + c)`

This is a great example of common subexpression elimination.
1.  `t1 = a * b`
2.  `t2 = t1 + c`
3.  `t3 = a * b` -> Re-use `t1`
4.  `t4 = t3 + c` -> Re-use `t2`
5.  `t5 = t2 - t4` -> Becomes `t5 = t2 - t2`
6.  `a = t5`

The DAG construction immediately sees that `a * b + c` is computed twice.

**DAG:**
```
          -
         / \
        /   \
      (+) --' (node is its own second child)
     /   \
    /     \
   (*)     c
  /   \
 a     b
```
The final node for `-` has the `(+)` node as both of its children. This shows the expression is `(something) - (the same thing)`. The optimized code would compute `t1=a*b`, `t2=t1+c`, `t5=t2-t2`, `a=t5`. A further optimization would recognize `t2-t2` is 0 and set `a=0`.

### Example 2: `a = a * (b - c) + (b - c) * d`

This example shows both common subexpression and re-use of a variable.
1.  `t1 = b - c`
2.  `t2 = a * t1`
3.  `t3 = b - c` -> Re-use `t1`
4.  `t4 = t3 * d` -> `t4 = t1 * d`
5.  `t5 = t2 + t4`
6.  `a = t5`

**DAG:**
The node for `b-c` is created once and used twice.

```
          (+) -- a
         /   \
        /     \
      (*)     (*)
     /   \   /   \
    a    (-)    d
        /   \
       b     c
```

### Example 3: `D := B * C; E := A + B; B := B * C; A := E - D;`

This sequence shows how a DAG handles re-assignment of variables.
1.  `D = B * C`
2.  `E = A + B`
3.  `B = B * C` -> This is a common subexpression with the first line, but the result is assigned to `B`, changing its value for subsequent instructions.
4.  `A = E - D`

**DAG Construction Steps:**
*   **`D = B * C`**: Create `*` node with children `B₀` and `C₀`. Label this node `D`.
*   **`E = A + B`**: Create `+` node with children `A₀` and `B₀`. Label this node `E`.
*   **`B = B * C`**: A node for `B₀ * C₀` already exists (it's node `D`). We attach `B` as a new label to this node. The old label `B₀` is now superseded for future steps, though the node itself remains.
*   **`A = E - D`**: Create `-` node with children `E` and `D`. Label this node `A`.

**Final DAG:**
*Initial values are denoted with a subscript 0.*

```
      (-) -- A
     /   \
    /     \
  (+) -- E  (*) -- D, B
 /   \     /   \
A₀    B₀  B₀    C₀
```
This DAG correctly shows that the final value of `A` depends on the initial values of `A`, `B`, and `C`. It also shows that `D` and the new `B` hold the same value (`B₀ * C₀`). 