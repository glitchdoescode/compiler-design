# Directed Acyclic Graph (DAG)

This document explains Directed Acyclic Graphs (DAGs), their usefulness for basic block transformations, construction algorithms, and provides detailed examples.

---

## a) What is a DAG? How is it useful for transformations on basic blocks?

### Definition of DAG

A **Directed Acyclic Graph (DAG)** is a graphical representation of a basic block that shows the flow of values and operations. It represents the structure of expressions and assignments in a basic block, where:

- **Nodes** represent operations, variables, or constants
- **Edges** represent data dependencies
- **No cycles** exist in the graph (acyclic property)

### Components of a DAG

1. **Leaf Nodes:** Represent variables and constants (operands)
2. **Interior Nodes:** Represent operations (+, -, *, /, etc.)
3. **Root Nodes:** Represent final results or variables being assigned
4. **Edges:** Show data flow from operands to operations

### How DAGs are Useful for Basic Block Transformations

#### 1. **Common Subexpression Elimination**

**Problem:** Multiple computations of the same expression
```
t1 = a + b
t2 = a + b    // Redundant computation
c = t1 * t2
```

**DAG Solution:** Shows that `a + b` is computed only once
- Both `t1` and `t2` point to the same node
- Eliminates redundant computation

#### 2. **Dead Code Elimination**

**Problem:** Variables computed but never used
```
t1 = a + b    // Dead if t1 is never used
t2 = c * d
result = t2
```

**DAG Solution:** Nodes with no outgoing edges (except final assignments) can be eliminated

#### 3. **Constant Folding**

**Problem:** Operations on constants computed at runtime
```
t1 = 2 + 3    // Can be computed at compile time
t2 = t1 * x
```

**DAG Solution:** Constant nodes can be combined into single constant nodes

#### 4. **Copy Propagation**

**Problem:** Unnecessary copy operations
```
t1 = x
t2 = t1 + y   // Can use x directly
```

**DAG Solution:** Shows direct dependencies, eliminating intermediate copies

#### 5. **Strength Reduction**

**Problem:** Expensive operations that can be replaced with cheaper ones
```
t1 = x * 2    // Can be replaced with x + x
```

**DAG Solution:** Identifies patterns suitable for strength reduction

### Advantages of Using DAGs

1. **Visual Representation:** Clear view of data dependencies
2. **Optimization Opportunities:** Easy identification of redundancies
3. **Efficient Analysis:** Single representation for multiple optimizations
4. **Preservation of Semantics:** Maintains program correctness
5. **Compact Representation:** Eliminates redundant subexpressions

---

## b) Algorithm to Construct a DAG from a Basic Block

### DAG Construction Algorithm

```
Algorithm: Construct_DAG(Basic_Block)
Input: A basic block with three-address code statements
Output: DAG representing the basic block

1. Initialize:
   - Create empty DAG
   - Create symbol table for variables
   - Create node table for expressions

2. For each statement in the basic block:
   Case 1: x = y op z
      a) Find or create nodes for y and z
      b) Check if node for (op, y, z) already exists
      c) If exists, attach x to existing node
      d) If not exists, create new node for (op, y, z)
      e) Update symbol table: x points to this node
      f) Remove x from any other nodes it was attached to

   Case 2: x = y (copy statement)
      a) Find node for y
      b) Attach x to the same node as y
      c) Update symbol table: x points to y's node
      d) Remove x from any other nodes

   Case 3: x = constant
      a) Find or create constant node
      b) Attach x to constant node
      c) Update symbol table

3. Mark nodes that correspond to final values needed outside the block

4. Return the constructed DAG
```

### Detailed Algorithm Implementation

```c
typedef struct DAGNode {
    char* operation;        // Operation (+, -, *, etc.) or "LEAF"
    struct DAGNode* left;   // Left operand
    struct DAGNode* right;  // Right operand
    char** variables;       // Variables attached to this node
    int var_count;         // Number of variables attached
    int node_id;           // Unique identifier
    int is_constant;       // Flag for constant nodes
    int constant_value;    // Value if constant
} DAGNode;

DAGNode* find_or_create_operation_node(char* op, DAGNode* left, DAGNode* right) {
    // Search existing nodes for matching operation
    for (int i = 0; i < node_count; i++) {
        if (nodes[i].operation == op && 
            nodes[i].left == left && 
            nodes[i].right == right) {
            return &nodes[i];
        }
    }
    
    // Create new node if not found
    DAGNode* new_node = create_new_node();
    new_node->operation = op;
    new_node->left = left;
    new_node->right = right;
    return new_node;
}
```

---

## c) Construct DAGs for Given Statements

### Example 1: `a = (a * b + c) - (a * b + c)`

**Three-Address Code:**
```
t1 = a * b
t2 = t1 + c
t3 = a * b      // Same as t1
t4 = t3 + c     // Same as t2
t5 = t2 - t4    // Same as t2 - t2 = 0
a = t5
```

**Step-by-step DAG Construction:**

1. **Process `t1 = a * b`:**
   - Create leaf nodes: `a`, `b`
   - Create operation node: `*(a,b)`
   - Attach `t1` to `*(a,b)`

2. **Process `t2 = t1 + c`:**
   - Create leaf node: `c`
   - Create operation node: `+(*(a,b), c)`
   - Attach `t2` to `+(*(a,b), c)`

3. **Process `t3 = a * b`:**
   - Operation `*(a,b)` already exists
   - Attach `t3` to existing `*(a,b)` node
   - Node `*(a,b)` now has variables: `{t1, t3}`

4. **Process `t4 = t3 + c`:**
   - Operation `+(*(a,b), c)` already exists
   - Attach `t4` to existing `+(*(a,b), c)` node
   - Node `+(*(a,b), c)` now has variables: `{t2, t4}`

5. **Process `t5 = t2 - t4`:**
   - Both `t2` and `t4` point to same node `+(*(a,b), c)`
   - Create operation node: `-(+(*(a,b), c), +(*(a,b), c))`
   - Attach `t5` to this node

6. **Process `a = t5`:**
   - Remove `a` from leaf node
   - Attach `a` to the same node as `t5`

**Final DAG:**
```
        a, t5
          |
          -
         / \
        /   \
   t2,t4     t2,t4  (same node)
      |
      +
     / \
    /   \
  t1,t3   c
    |
    *
   / \
  a    b
 (old)
```

**Optimization Result:** The compiler can recognize that this computes `0` since we're subtracting identical expressions.

### Example 2: `a = a * (b - c) + (b - c) * d`

**Three-Address Code:**
```
t1 = b - c
t2 = a * t1
t3 = b - c      // Same as t1
t4 = t3 * d     // Can use t1 instead of t3
t5 = t2 + t4
a = t5
```

**Step-by-step DAG Construction:**

1. **Process `t1 = b - c`:**
   - Create leaf nodes: `b`, `c`
   - Create operation node: `-(b,c)`
   - Attach `t1` to `-(b,c)`

2. **Process `t2 = a * t1`:**
   - Create leaf node: `a`
   - Create operation node: `*(a, -(b,c))`
   - Attach `t2` to `*(a, -(b,c))`

3. **Process `t3 = b - c`:**
   - Operation `-(b,c)` already exists
   - Attach `t3` to existing `-(b,c)` node
   - Node `-(b,c)` now has variables: `{t1, t3}`

4. **Process `t4 = t3 * d`:**
   - Create leaf node: `d`
   - Create operation node: `*(-(b,c), d)`
   - Attach `t4` to `*(-(b,c), d)`

5. **Process `t5 = t2 + t4`:**
   - Create operation node: `+(*(a, -(b,c)), *(-(b,c), d))`
   - Attach `t5` to this node

6. **Process `a = t5`:**
   - Remove `a` from leaf node
   - Attach `a` to the same node as `t5`

**Final DAG:**
```
        a, t5
          |
          +
         / \
        /   \
      t2     t4
      |      |
      *      *
     / \    / \
    a  t1,t3  t1,t3  d
   (old) |      |
         -      - (same node)
        / \    / \
       b   c  b   c
```

**Optimization Result:** Common subexpression `(b - c)` is computed only once and reused.

### Example 3: `D := B * C; E := A + B; B := B * C; A := E - D;`

**Three-Address Code:**
```
t1 = B * C
D = t1
t2 = A + B
E = t2
t3 = B * C      // Same as t1
B = t3          // B gets new value
t4 = E - D
A = t4
```

**Step-by-step DAG Construction:**

1. **Process `t1 = B * C`:**
   - Create leaf nodes: `B`, `C`
   - Create operation node: `*(B,C)`
   - Attach `t1` to `*(B,C)`

2. **Process `D = t1`:**
   - Attach `D` to the same node as `t1`
   - Node `*(B,C)` now has variables: `{t1, D}`

3. **Process `t2 = A + B`:**
   - Create leaf node: `A`
   - Create operation node: `+(A,B)`
   - Attach `t2` to `+(A,B)`

4. **Process `E = t2`:**
   - Attach `E` to the same node as `t2`
   - Node `+(A,B)` now has variables: `{t2, E}`

5. **Process `t3 = B * C`:**
   - Operation `*(B,C)` already exists
   - Attach `t3` to existing `*(B,C)` node
   - Node `*(B,C)` now has variables: `{t1, D, t3}`

6. **Process `B = t3`:**
   - Remove `B` from leaf node (B gets redefined)
   - Attach `B` to the same node as `t3`
   - Node `*(B,C)` now has variables: `{t1, D, t3, B}`

7. **Process `t4 = E - D`:**
   - Create operation node: `-(+(A,B_old), *(B_old,C))`
   - Attach `t4` to this node

8. **Process `A = t4`:**
   - Remove `A` from leaf node
   - Attach `A` to the same node as `t4`

**Final DAG:**
```
        A, t4
          |
          -
         / \
        /   \
    t2,E     t1,D,t3,B
      |         |
      +         *
     / \       / \
    A   B     B   C
   (old)(old)(old)
```

**Key Points:**
- Original values of `A` and `B` are preserved for the computation
- `B` gets redefined but the old value is still used in the subtraction
- Common subexpression `B * C` is identified and reused

---

## Advanced DAG Optimizations

### 1. **Algebraic Simplifications**

**Example:** `x - x = 0`
```
DAG can detect when both operands of subtraction are identical
Result: Replace with constant 0
```

**Example:** `x * 1 = x`
```
DAG can detect multiplication by 1
Result: Replace with copy operation
```

### 2. **Constant Propagation**

```
a = 5
b = a + 3    // Can be computed as 5 + 3 = 8
```

**DAG representation:**
- Constant nodes can be folded
- Propagate constants through operations

### 3. **Copy Propagation**

```
a = b
c = a + d    // Can use b directly: c = b + d
```

**DAG representation:**
- Variables pointing to same node can be substituted

---

## Code Generation from DAG

### Algorithm: Generate Optimized Code

```
1. Perform topological sort of DAG nodes
2. For each node in reverse topological order:
   - If node has multiple variables attached:
     * Choose one as primary
     * Generate copies for others if needed
   - Generate operation code
3. Eliminate dead assignments
4. Generate final code sequence
```

### Example Code Generation

**From DAG for `a = a * (b - c) + (b - c) * d`:**

**Optimized Code:**
```
t1 = b - c      // Compute common subexpression once
t2 = a * t1     // Use t1 instead of recomputing b - c
t3 = t1 * d     // Reuse t1 again
a = t2 + t3     // Final result
```

**Original vs Optimized:**
- **Original:** 6 operations (including redundant `b - c`)
- **Optimized:** 4 operations (common subexpression eliminated)

---

## Summary

### Key Benefits of DAGs

1. **Automatic Optimization:** Multiple optimizations applied simultaneously
2. **Common Subexpression Elimination:** Redundant computations removed
3. **Dead Code Detection:** Unused computations identified
4. **Constant Folding:** Compile-time evaluation of constants
5. **Visual Analysis:** Clear representation of data dependencies

### DAG Construction Process

1. **Sequential Processing:** Handle statements one by one
2. **Node Reuse:** Identify and merge identical operations
3. **Variable Tracking:** Maintain symbol table for variable locations
4. **Dependency Preservation:** Maintain correct evaluation order

### Optimization Results from Examples

- **Example 1:** Expression simplifies to constant `0`
- **Example 2:** Common subexpression `(b - c)` computed once
- **Example 3:** Variable redefinition handled correctly with value preservation

DAGs provide a powerful framework for basic block optimization, enabling compilers to generate more efficient code while preserving program semantics. 