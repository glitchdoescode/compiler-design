# Basic Blocks, Flow Graphs, and Data Flow Analysis

This document explains basic blocks and control flow graphs, the algorithm for creating them, and the concept of data flow analysis.

---

## a) Basic Blocks and Control Flow Graphs

### Basic Blocks

A **basic block** is a maximal sequence of consecutive three-address instructions with two key properties:
1.  **Single Entry Point:** The control flow can only enter the basic block at its first instruction. There are no jumps from elsewhere into the middle of the block.
2.  **Single Exit Point:** Once the sequence of instructions begins, it continues to the end without any possibility of halting or branching out, except at the very last instruction.

In essence, a basic block is a chunk of "straight-line code." Identifying basic blocks is the first step in analyzing the control flow of a program.

**Example:**
The following code...
```
L1:
t1 = x + y
t2 = a * b
if t2 > t1 goto L2
a = a + 1
goto L1
L2:
...
```
...contains two basic blocks:
*   **Block 1:** `t1 = x + y`, `t2 = a * b`, `if t2 > t1 goto L2`
*   **Block 2:** `a = a + 1`, `goto L1`

### Control Flow Graphs (CFG)

A **Control Flow Graph (CFG)** is a directed graph used to represent the control flow of a program. In a CFG:
*   **Nodes:** The nodes of the graph are the basic blocks.
*   **Edges:** A directed edge from block `B1` to block `B2` exists if it is possible for the program to execute `B2` immediately after `B1`. This can happen in two ways:
    1.  There is a conditional or unconditional jump from the end of `B1` to the beginning of `B2`.
    2.  `B2` immediately follows `B1` in the original order of the code, and `B1` does not end in an unconditional jump. This is called "falling through."

Two special blocks are often added: an `ENTRY` block and an `EXIT` block, which represent the start and end of the procedure, respectively.

---

## b) Algorithm to Partition into Basic Blocks

This algorithm takes a sequence of three-address instructions and divides it into a set of basic blocks.

**Input:** A sequence of three-address instructions.
**Output:** A list of basic blocks, where each instruction is in exactly one block.

**Algorithm:**
1.  **Determine Leaders:** First, identify all the "leader" instructions in the code. A leader is the first instruction of a basic block. The rules for finding leaders are:
    *   **Rule 1:** The very first instruction in the sequence is a leader.
    *   **Rule 2:** Any instruction that is the target of a conditional or unconditional `goto` is a leader.
    *   **Rule 3:** Any instruction that immediately follows a `goto` or a conditional `goto` is a leader.

2.  **Construct Blocks:** For each leader identified, its basic block consists of:
    *   The leader itself.
    *   All instructions that follow the leader, up to (but not including) the next leader or the end of the program.

**Example:**
Consider the code for a 3x3 matrix multiplication:
```
(1)  prod = 0
(2)  i = 1
(3) L1: j = 1
(4) L2: t1 = 4 * i
(5)  t2 = t1 + j
(6)  t3 = a[t2]
(7)  t4 = 4 * j
(8)  t5 = t4 + k  // Assume k is available
(9)  t6 = b[t5]
(10) t7 = t3 * t6
(11) t8 = prod + t7
(12) prod = t8
(13) j = j + 1
(14) if j <= 3 goto L2
(15) i = i + 1
(16) if i <= 3 goto L1
```
*   **Leaders:**
    *   `(1)` is a leader (Rule 1).
    *   `(3)` is a leader (target of goto at 16, `L1`).
    *   `(4)` is a leader (target of goto at 14, `L2`).
    *   `(15)` is a leader (follows goto at 14, Rule 3).
*   **Blocks:**
    *   **B1:** Instructions (1)-(2)
    *   **B2:** Instruction (3)
    *   **B3:** Instructions (4)-(14)
    *   **B4:** Instructions (15)-(16)

---

## c) Data Flow Analysis of a Structured Flow Graph

**Data flow analysis (DFA)** is a technique for gathering information about the possible set of values or properties ("data flow values") that can exist at various points in a program. It is the foundation for most global optimizations. DFA operates on the CFG and typically involves solving a system of equations.

### Key Concepts:

*   **Data Flow Values:** The information being gathered. For example, in "reaching definitions," the values are sets of variable definitions that might reach a certain point.
*   **`IN[B]` and `OUT[B]`:** For each basic block `B`, we compute `IN[B]`, the data flow value at the entry of the block, and `OUT[B]`, the value at the exit.
*   **Transfer Function (`f_B`):** This function models the effect of the basic block `B`. It takes the data flow value at the entry of the block and produces the value at the exit: `OUT[B] = f_B(IN[B])`. The function is composed of two parts:
    *   **`gen[B]`:** The set of values *generated* within block `B`.
    *   **`kill[B]`:** The set of values that are *invalidated* or "killed" by block `B`.
    *   The transfer function is often of the form: `OUT[B] = gen[B] ∪ (IN[B] - kill[B])`.
*   **Meet Operator (`∧`):** This operator combines the data flow values from predecessor blocks to compute the value at the entry of a block. For a block `B`, `IN[B] = ∧ P∈pred(B) OUT[P]`. The meet operator is typically set union (`∪`) for "forward, any-path" problems (like reaching definitions) or set intersection (`∩`) for "forward, all-paths" problems.

### The Iterative Algorithm

Since the `IN` and `OUT` sets are interdependent, they are solved using an iterative algorithm until a fixed point is reached.

1.  **Initialization:** For every block `B`, initialize `OUT[B] = ∅`. For the `ENTRY` block, `IN[ENTRY]` is also initialized (e.g., to empty).
2.  **Iteration:** Repeatedly execute the following steps for each block `B` until the `OUT` sets for all blocks no longer change:
    *   `IN[B] = ∪ P∈pred(B) OUT[P]` (applying the meet operator)
    *   `OUT[B] = f_B(IN[B])` (applying the transfer function)

This process is guaranteed to terminate because the transfer functions are monotonic and the set of data flow values is finite.

**Example: Reaching Definitions**
*   **Goal:** For each point in the program, find the set of definitions that might reach that point.
*   **`gen[B]`:** The set of definitions `d: y = ...` within block `B` that are "live" at the end of `B`.
*   **`kill[B]`:** For a definition `d: y = ...` in `B`, `kill[B]` is the set of all other definitions of `y` from anywhere else in the program.
*   **Meet Operator:** `∪` (union), because a definition reaches a point if it can get there from *any* predecessor.
The algorithm would compute the sets `IN` and `OUT` for each block, giving the optimizer a complete picture of which variable definitions are active across the entire function. 