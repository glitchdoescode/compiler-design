# Three-Address Code (TAC)

This document explains Three-Address Code and demonstrates its generation for various code segments.

---

## a) What is Three-Address Code?

### Definition

**Three-Address Code (TAC)** is an intermediate representation where each instruction has at most three operands: two source operands and one destination operand. It is a linearized representation of a syntax tree.

### General Form

```
result = operand1 operator operand2
```

Where:
- `result` is the destination
- `operand1` and `operand2` are the source operands
- `operator` is the operation to be performed

### Characteristics

1. **At most three addresses** per instruction
2. **Linear sequence** of instructions
3. **Explicit temporary variables** for intermediate results
4. **Machine-independent** representation
5. **Easy to optimize** and translate to target code

### Types of Three-Address Instructions

#### 1. Assignment Instructions
```
x = y op z          // Binary operation
x = op y            // Unary operation
x = y               // Copy operation
```

#### 2. Control Flow Instructions
```
goto L              // Unconditional jump
if x relop y goto L // Conditional jump
if x goto L         // Conditional jump (boolean)
ifFalse x goto L    // Conditional jump (negated)
```

#### 3. Procedure Call Instructions
```
param x             // Parameter passing
call p, n           // Call procedure p with n parameters
return y            // Return from procedure
```

#### 4. Array and Pointer Instructions
```
x = y[i]            // Array access
x[i] = y            // Array assignment
x = &y              // Address of
x = *y              // Dereference
*x = y              // Indirect assignment
```

#### 5. Label Instructions
```
L:                  // Label definition
```

### Advantages

1. **Simplicity:** Easy to understand and manipulate
2. **Optimization:** Suitable for various optimization techniques
3. **Target Independence:** Can be translated to different architectures
4. **Analysis:** Facilitates data flow and control flow analysis

### Disadvantages

1. **Space:** May require more instructions than original code
2. **Temporaries:** Generates many temporary variables
3. **Overhead:** Additional instructions for complex expressions

---

## b) Three-Address Code Generation Examples

### Example 1: Complex Control Structure

**Source Code:**
```
while (a < c AND b < d) do
  if (a = 1) then c = c + 1
  else while (a <= 4) do a = a + 3
```

**Three-Address Code Generation:**

```
L1:   t1 = a < c              // Evaluate first condition
      ifFalse t1 goto L2      // If false, exit outer while
      t2 = b < d              // Evaluate second condition  
      ifFalse t2 goto L2      // If false, exit outer while
      t3 = a == 1             // Check if a equals 1
      ifFalse t3 goto L3      // If false, go to else part
      
      // Then part: c = c + 1
      t4 = c + 1              // Calculate c + 1
      c = t4                  // Assign to c
      goto L1                 // Go back to while condition
      
L3:   // Else part: while (a <= 4) do a = a + 3
L4:   t5 = a <= 4             // Inner while condition
      ifFalse t5 goto L1      // If false, go back to outer while
      t6 = a + 3              // Calculate a + 3
      a = t6                  // Assign to a
      goto L4                 // Go back to inner while condition
      
L2:   // End of outer while loop
```

**Detailed Explanation:**

1. **L1:** Start of outer while loop
2. **t1 = a < c:** Evaluate first part of AND condition
3. **ifFalse t1 goto L2:** Short-circuit evaluation - if first condition false, exit
4. **t2 = b < d:** Evaluate second part of AND condition
5. **ifFalse t2 goto L2:** If second condition false, exit
6. **t3 = a == 1:** Check condition for if statement
7. **ifFalse t3 goto L3:** If condition false, go to else part
8. **Then part:** Execute c = c + 1 and return to while
9. **L3-L4:** Else part with inner while loop
10. **L2:** Exit point of outer while

### Example 2: Arithmetic Expression

**Source Code:**
```
P := (K + Y) + (K - C)
```

**Three-Address Code Generation:**

```
t1 = K + Y              // Calculate first subexpression
t2 = K - C              // Calculate second subexpression  
t3 = t1 + t2            // Add the two subexpressions
P = t3                  // Assign result to P
```

**Alternative (more optimized):**
```
t1 = K + Y              // Calculate K + Y
t2 = K - C              // Calculate K - C
P = t1 + t2             // Directly assign the sum to P
```

### Example 3: Array Operations

**Source Code:**
```
A[i] = B[j] + C[k] * 2
```

**Three-Address Code:**
```
t1 = C[k]               // Load C[k]
t2 = t1 * 2             // Multiply by 2
t3 = B[j]               // Load B[j]
t4 = t3 + t2            // Add B[j] + (C[k] * 2)
A[i] = t4               // Store result in A[i]
```

### Example 4: Function Call

**Source Code:**
```
result = func(a + b, c * d)
```

**Three-Address Code:**
```
t1 = a + b              // Calculate first argument
t2 = c * d              // Calculate second argument
param t1                // Pass first parameter
param t2                // Pass second parameter
call func, 2            // Call function with 2 parameters
result = return_value   // Get return value
```

### Example 5: Complex Boolean Expression

**Source Code:**
```
if ((x > 0) && (y < 10) || (z == 5)) then s1 else s2
```

**Three-Address Code:**
```
t1 = x > 0              // Evaluate x > 0
ifFalse t1 goto L1      // If false, check second part of OR
t2 = y < 10             // Evaluate y < 10
ifFalse t2 goto L1      // If false, check second part of OR
goto L2                 // Both conditions true, execute then part

L1:   t3 = z == 5       // Evaluate z == 5
      ifFalse t3 goto L3 // If false, execute else part

L2:   // Then part
      // Code for s1
      goto L4           // Skip else part

L3:   // Else part  
      // Code for s2

L4:   // Continue after if-else
```

---

## Advanced TAC Generation Techniques

### 1. Short-Circuit Evaluation

For boolean expressions with AND/OR:

**Source:** `if (a && b) then s1`

**TAC with short-circuit:**
```
ifFalse a goto L1       // If a is false, skip b evaluation
ifFalse b goto L1       // If b is false, skip then part
// Code for s1
L1: // Continue
```

### 2. Optimized Temporary Usage

**Before optimization:**
```
t1 = a + b
t2 = t1 * c
t3 = t2 - d
result = t3
```

**After optimization:**
```
t1 = a + b
t1 = t1 * c             // Reuse t1
t1 = t1 - d             // Reuse t1
result = t1
```

### 3. Addressing Modes

**Array with computed index:**
```
A[i + j * 2] = x
```

**TAC:**
```
t1 = j * 2              // Calculate offset
t2 = i + t1             // Calculate final index
A[t2] = x               // Store using computed index
```

---

## Implementation Data Structures

### Quadruple Representation

Each TAC instruction can be represented as a quadruple:
```
(operator, operand1, operand2, result)
```

**Example:**
```
t1 = a + b    →    (+, a, b, t1)
goto L1       →    (goto, L1, -, -)
if x goto L2  →    (if, x, L2, -)
```

### Triple Representation

More compact representation without explicit result field:
```
(operator, operand1, operand2)
```

Results are referenced by instruction number.

### Indirect Triple

Combines benefits of quadruples and triples with an indirection table.

---

## TAC in Compiler Pipeline

```
Source Code
     ↓
   Parser
     ↓
Syntax Tree
     ↓
TAC Generator  ← (This phase)
     ↓
Three-Address Code
     ↓
Optimizer
     ↓
Optimized TAC
     ↓
Code Generator
     ↓
Target Code
```

---

## Summary

**Three-Address Code characteristics:**
- **Linear representation** of program structure
- **At most three operands** per instruction
- **Explicit temporaries** for intermediate results
- **Machine-independent** intermediate form
- **Suitable for optimization** and analysis

**Key benefits:**
1. Simplifies code generation and optimization
2. Provides clear separation between front-end and back-end
3. Enables various optimization techniques
4. Facilitates target-independent compilation

**Generated TAC for examples:**
1. **Complex control:** Proper handling of nested loops and conditions
2. **Arithmetic:** Efficient temporary usage for subexpressions
3. **Boolean expressions:** Short-circuit evaluation for efficiency

The three-address code serves as an excellent intermediate representation that balances simplicity with expressiveness, making it ideal for compiler optimization and code generation phases. 