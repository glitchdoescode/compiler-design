# Backpatching

This document explains the concept of backpatching with detailed examples and implementation techniques.

---

## What is Backpatching?

### Definition

**Backpatching** is a technique used in compiler design to generate intermediate code for boolean expressions and control flow statements in a single pass. Instead of generating complete jump instructions immediately, it generates incomplete instructions with "holes" that are filled in later when the target addresses become known.

### Key Concepts

1. **Forward References:** Jump targets that are not yet defined
2. **Patch Lists:** Lists of incomplete instructions waiting for target addresses
3. **Single Pass:** Generate code without multiple passes over the syntax tree
4. **Deferred Address Resolution:** Fill in addresses when they become available

### Why Backpatching?

**Problem without backpatching:**
- In single-pass compilation, we often encounter forward jumps
- Target labels are not yet defined when we need to generate jump instructions
- Traditional approach requires multiple passes or complex lookahead

**Solution with backpatching:**
- Generate incomplete instructions with placeholder addresses
- Maintain lists of instructions that need the same target
- Fill in the actual addresses when targets are defined

---

## Basic Backpatching Mechanism

### Data Structures

1. **Instruction List:** Sequence of three-address code instructions
2. **Patch Lists:** Lists of instruction indices that need the same target
3. **Next Instruction Pointer:** Points to the next instruction to be generated

### Key Operations

#### 1. `makelist(i)`
Creates a new list containing only instruction `i`.

#### 2. `merge(p1, p2)`
Merges two patch lists `p1` and `p2` into a single list.

#### 3. `backpatch(p, i)`
Fills in the target field of all instructions in patch list `p` with instruction index `i`.

#### 4. `nextinstr()`
Returns the index of the next instruction to be generated.

---

## Backpatching for Boolean Expressions

### Grammar with Attributes

```
B → B₁ || M B₂     { backpatch(B₁.falselist, M.instr);
                     B.truelist = merge(B₁.truelist, B₂.truelist);
                     B.falselist = B₂.falselist; }

B → B₁ && M B₂     { backpatch(B₁.truelist, M.instr);
                     B.truelist = B₂.truelist;
                     B.falselist = merge(B₁.falselist, B₂.falselist); }

B → !B₁            { B.truelist = B₁.falselist;
                     B.falselist = B₁.truelist; }

B → (B₁)           { B.truelist = B₁.truelist;
                     B.falselist = B₁.falselist; }

B → E₁ relop E₂    { B.truelist = makelist(nextinstr());
                     B.falselist = makelist(nextinstr() + 1);
                     emit('if', E₁.place, 'relop', E₂.place, 'goto', '_');
                     emit('goto', '_'); }

B → true           { B.truelist = makelist(nextinstr());
                     emit('goto', '_'); }

B → false          { B.falselist = makelist(nextinstr());
                     emit('goto', '_'); }

M → ε              { M.instr = nextinstr(); }
```

### Attributes

- **B.truelist:** List of instructions to be backpatched with the label when B is true
- **B.falselist:** List of instructions to be backpatched with the label when B is false
- **M.instr:** Instruction number for marker M (used to capture current position)

---

## Example 1: Simple Boolean Expression

### Expression: `a < b || c > d`

**Step-by-step generation:**

1. **Parse `a < b`:**
   ```
   100: if a < b goto _     // truelist = [100]
   101: goto _              // falselist = [101]
   ```

2. **Encounter `||` operator:**
   - Insert marker M: `M.instr = 102`

3. **Parse `c > d`:**
   ```
   102: if c > d goto _     // truelist = [102]
   103: goto _              // falselist = [103]
   ```

4. **Combine with `||`:**
   - `backpatch([101], 102)` - patch false case of first expression to second expression
   - `truelist = merge([100], [102]) = [100, 102]`
   - `falselist = [103]`

**Final code:**
```
100: if a < b goto _     // Will be patched with true target
101: goto 102            // Patched to evaluate second expression
102: if c > d goto _     // Will be patched with true target  
103: goto _              // Will be patched with false target
```

---

## Example 2: Complex Boolean Expression

### Expression: `(a < b) && (c > d || e == f)`

**Parse tree and generation:**

1. **Parse `a < b`:**
   ```
   100: if a < b goto _     // truelist = [100]
   101: goto _              // falselist = [101]
   ```

2. **Encounter `&&`, insert marker M₁:**
   - `M₁.instr = 102`

3. **Parse `c > d`:**
   ```
   102: if c > d goto _     // truelist = [102]
   103: goto _              // falselist = [103]
   ```

4. **Encounter `||`, insert marker M₂:**
   - `M₂.instr = 104`

5. **Parse `e == f`:**
   ```
   104: if e == f goto _    // truelist = [104]
   105: goto _              // falselist = [105]
   ```

6. **Combine `c > d || e == f`:**
   - `backpatch([103], 104)` - patch false case to second expression
   - `truelist = merge([102], [104]) = [102, 104]`
   - `falselist = [105]`

7. **Combine with `&&`:**
   - `backpatch([100], 102)` - patch true case of first expression
   - `truelist = [102, 104]`
   - `falselist = merge([101], [105]) = [101, 105]`

**Intermediate code:**
```
100: if a < b goto 102   // Patched to continue with second part
101: goto _              // Will be patched with false target
102: if c > d goto _     // Will be patched with true target
103: goto 104            // Patched to evaluate e == f
104: if e == f goto _    // Will be patched with true target
105: goto _              // Will be patched with false target
```

---

## Backpatching for Control Statements

### If-Then-Else Statement

**Grammar:**
```
S → if B then M₁ S₁ N else M₂ S₂
  { backpatch(B.truelist, M₁.instr);
    backpatch(B.falselist, M₂.instr);
    S.nextlist = merge(S₁.nextlist, N.nextlist, S₂.nextlist); }

S → if B then M S₁
  { backpatch(B.truelist, M.instr);
    S.nextlist = merge(B.falselist, S₁.nextlist); }

N → ε
  { N.nextlist = makelist(nextinstr());
    emit('goto', '_'); }

M → ε
  { M.instr = nextinstr(); }
```

### Example: If-Then-Else

**Source code:**
```
if (a < b) then x = 1 else y = 2
```

**Generation process:**

1. **Parse condition `a < b`:**
   ```
   100: if a < b goto _     // B.truelist = [100]
   101: goto _              // B.falselist = [101]
   ```

2. **Insert marker M₁ for then part:**
   - `M₁.instr = 102`

3. **Generate then statement:**
   ```
   102: x = 1               // S₁.nextlist = []
   ```

4. **Insert N for jump over else:**
   ```
   103: goto _              // N.nextlist = [103]
   ```

5. **Insert marker M₂ for else part:**
   - `M₂.instr = 104`

6. **Generate else statement:**
   ```
   104: y = 2               // S₂.nextlist = []
   ```

7. **Backpatch:**
   - `backpatch([100], 102)` - true case goes to then part
   - `backpatch([101], 104)` - false case goes to else part
   - `S.nextlist = merge([], [103], []) = [103]`

**Final code:**
```
100: if a < b goto 102   // Patched
101: goto 104            // Patched
102: x = 1               // Then part
103: goto _              // Will be patched to next statement
104: y = 2               // Else part
```

---

## Backpatching for Loops

### While Loop

**Grammar:**
```
S → while M₁ B do M₂ S₁
  { backpatch(S₁.nextlist, M₁.instr);
    backpatch(B.truelist, M₂.instr);
    S.nextlist = B.falselist;
    emit('goto', M₁.instr); }
```

### Example: While Loop

**Source code:**
```
while (a < b) do a = a + 1
```

**Generation process:**

1. **Insert marker M₁ for loop start:**
   - `M₁.instr = 100`

2. **Parse condition `a < b`:**
   ```
   100: if a < b goto _     // B.truelist = [100]
   101: goto _              // B.falselist = [101]
   ```

3. **Insert marker M₂ for loop body:**
   - `M₂.instr = 102`

4. **Generate loop body:**
   ```
   102: t1 = a + 1          // Loop body
   103: a = t1              // S₁.nextlist = []
   ```

5. **Generate jump back to condition:**
   ```
   104: goto 100            // Jump back to loop condition
   ```

6. **Backpatch:**
   - `backpatch([], 100)` - no breaks in body
   - `backpatch([100], 102)` - true case enters loop body
   - `S.nextlist = [101]` - false case exits loop

**Final code:**
```
100: if a < b goto 102   // Patched
101: goto _              // Will be patched to next statement
102: t1 = a + 1          // Loop body
103: a = t1
104: goto 100            // Jump back to condition
```

---

## Implementation Details

### Data Structure for Patch Lists

```c
typedef struct patchlist {
    int instruction;
    struct patchlist* next;
} PatchList;

// Create a new list with one instruction
PatchList* makelist(int instr) {
    PatchList* p = malloc(sizeof(PatchList));
    p->instruction = instr;
    p->next = NULL;
    return p;
}

// Merge two patch lists
PatchList* merge(PatchList* p1, PatchList* p2) {
    if (p1 == NULL) return p2;
    if (p2 == NULL) return p1;
    
    PatchList* temp = p1;
    while (temp->next != NULL) {
        temp = temp->next;
    }
    temp->next = p2;
    return p1;
}

// Backpatch all instructions in list with target
void backpatch(PatchList* p, int target) {
    while (p != NULL) {
        // Fill in the target field of instruction p->instruction
        instructions[p->instruction].target = target;
        p = p->next;
    }
}
```

### Instruction Representation

```c
typedef struct instruction {
    char* op;           // Operation
    char* arg1;         // First argument
    char* arg2;         // Second argument  
    int target;         // Target (for jumps)
} Instruction;

Instruction instructions[1000];
int nextinstr_counter = 0;

int nextinstr() {
    return nextinstr_counter;
}

void emit(char* op, char* arg1, char* arg2, int target) {
    instructions[nextinstr_counter].op = op;
    instructions[nextinstr_counter].arg1 = arg1;
    instructions[nextinstr_counter].arg2 = arg2;
    instructions[nextinstr_counter].target = target;
    nextinstr_counter++;
}
```

---

## Advantages and Applications

### Advantages

1. **Single Pass:** Generates code in one pass without lookahead
2. **Efficient:** Avoids multiple traversals of the syntax tree
3. **Flexible:** Handles complex control structures naturally
4. **Memory Efficient:** Uses minimal additional storage

### Applications

1. **Boolean Expression Evaluation:** Short-circuit evaluation
2. **Control Flow:** If-then-else, while loops, for loops
3. **Break and Continue:** In loop constructs
4. **Exception Handling:** Try-catch blocks
5. **Switch Statements:** Multiple branch targets

### Limitations

1. **Complexity:** More complex than multi-pass approaches
2. **Memory Management:** Need to manage patch lists carefully
3. **Debugging:** Generated code can be harder to trace

---

## Summary

**Backpatching** is a powerful technique that enables:

- **Single-pass code generation** for control structures
- **Efficient handling** of forward references
- **Natural implementation** of short-circuit evaluation
- **Flexible code generation** for complex boolean expressions

**Key components:**
- **Patch lists** to track incomplete instructions
- **Marker productions** to capture instruction positions
- **Backpatch operations** to fill in target addresses
- **Merge operations** to combine patch lists

The technique is essential for modern compiler implementations, particularly for generating efficient code for boolean expressions and control flow statements in a single pass. 