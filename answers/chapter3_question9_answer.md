# Shift-Reduce Parsing Conflicts

This document explains the various conflicts that can occur during shift-reduce parsing.

---

## Introduction to Shift-Reduce Parsing

**Shift-reduce parsing** is a bottom-up parsing technique that uses a stack and performs two main operations:
- **Shift:** Move the next input symbol onto the stack
- **Reduce:** Replace symbols on top of the stack with a non-terminal (using a production rule in reverse)

The parser continues until either the input is accepted (stack contains only the start symbol and input is empty) or an error occurs.

---

## Types of Conflicts in Shift-Reduce Parsing

### 1. Shift-Reduce Conflict

**Definition:** A shift-reduce conflict occurs when the parser cannot decide whether to shift the next input symbol onto the stack or reduce the symbols currently on top of the stack.

**When it occurs:** When there are symbols on the stack that form the right-hand side of some production (a handle), but the parser could also shift the next input symbol.

**Example:**

Grammar:
```
E → E + E | E * E | id
```

Input: `id + id * id`

**Parsing trace:**
```
Stack: [id]           Input: + id * id
```

At this point, the parser faces a choice:
- **Shift:** Push '+' onto the stack
- **Reduce:** Reduce `id` to `E` using `E → id`

Both actions are valid, creating a shift-reduce conflict.

**Detailed Analysis:**
```
Option 1 (Shift first):
Stack: [id, +]        Input: id * id
Stack: [id, +, id]    Input: * id
Stack: [id, +, id, *] Input: id
Stack: [id, +, id, *, id] Input: []
Stack: [id, +, id, *, E]  Input: []    [Reduce: E → id]
Stack: [id, +, E]         Input: []    [Reduce: E → id * E]
Stack: [E]                Input: []    [Reduce: E → id + E]

Option 2 (Reduce first):
Stack: [E]            Input: + id * id  [Reduce: E → id]
Stack: [E, +]         Input: id * id
Stack: [E, +, id]     Input: * id
Stack: [E, +, E]      Input: * id       [Reduce: E → id]
Stack: [E]            Input: * id       [Reduce: E → E + E]
Stack: [E, *]         Input: id
Stack: [E, *, id]     Input: []
Stack: [E, *, E]      Input: []         [Reduce: E → id]
Stack: [E]            Input: []         [Reduce: E → E * E]
```

Both lead to different parse trees, indicating **ambiguity** in the grammar.

### 2. Reduce-Reduce Conflict

**Definition:** A reduce-reduce conflict occurs when there are two or more different production rules that could be applied to reduce the symbols currently on top of the stack.

**When it occurs:** When the symbols on the stack match the right-hand side of multiple productions.

**Example:**

Grammar:
```
S → A a | B a
A → b
B → b
```

Input: `b a`

**Parsing trace:**
```
Stack: [b]            Input: a
```

At this point, the parser faces a choice:
- **Reduce using A → b:** Stack becomes [A], then shift 'a' and reduce using S → A a
- **Reduce using B → b:** Stack becomes [B], then shift 'a' and reduce using S → B a

Both reductions are valid, creating a reduce-reduce conflict.

**Analysis:**
```
Option 1:
Stack: [b]     Input: a    
Stack: [A]     Input: a     [Reduce: A → b]
Stack: [A, a]  Input: []
Stack: [S]     Input: []    [Reduce: S → A a]

Option 2:
Stack: [b]     Input: a
Stack: [B]     Input: a     [Reduce: B → b]
Stack: [B, a]  Input: []
Stack: [S]     Input: []    [Reduce: S → B a]
```

This conflict indicates that the grammar is **ambiguous** - the same input can be parsed in multiple ways.

### 3. Accept-Reduce Conflict

**Definition:** An accept-reduce conflict occurs when the parser cannot decide whether to accept the input (parsing is complete) or perform a reduction.

**When it occurs:** When the stack contains the start symbol, the input is empty, but there are still possible reductions.

**Example:**

Grammar:
```
S → S S | a
```

Input: `a`

**Parsing trace:**
```
Stack: [a]            Input: []
```

At this point, the parser faces a choice:
- **Accept:** Since we have consumed all input, but the stack doesn't contain just the start symbol
- **Reduce:** Reduce `a` to `S` using `S → a`

Actually, this specific example would reduce first:
```
Stack: [a]     Input: []
Stack: [S]     Input: []    [Reduce: S → a]
Accept!
```

A better example of accept-reduce conflict:

Grammar:
```
S' → S
S → S a | ε
```

Input: `` (empty)

```
Stack: []      Input: []
Stack: [S]     Input: []    [Reduce: S → ε]
```

Now the parser could:
- **Accept:** Reduce using S' → S and accept
- **Reduce:** But there's no more input to process

### 4. Shift-Shift Conflict

**Definition:** A shift-shift conflict is rare and typically occurs in parsers that use multiple lookahead symbols or in certain LR parser variants.

**When it occurs:** When the parser has multiple ways to shift the same symbol.

This is less common in standard LR parsers but can occur in:
- Parsers with multiple lookahead
- GLR (Generalized LR) parsers
- Parsers handling ambiguous grammars

---

## Causes of Conflicts

### 1. Grammar Ambiguity
Most conflicts arise from ambiguous grammars where the same input string can be parsed in multiple ways.

**Example:** The classic "dangling else" problem
```
S → if E then S else S | if E then S | other
```

### 2. Inadequate Lookahead
The parser doesn't have enough lookahead information to make the correct decision.

### 3. Left Recursion vs Right Recursion
Different recursive structures can lead to conflicts.

**Example:**
```
E → E + T | T    (left recursive)
E → T + E | T    (right recursive)
```

### 4. Operator Precedence Issues
When operator precedence is not properly encoded in the grammar.

---

## Resolution Strategies

### 1. Grammar Rewriting
**Approach:** Modify the grammar to eliminate ambiguity.

**Example:** For the ambiguous expression grammar:
```
Original (ambiguous):
E → E + E | E * E | id

Rewritten (unambiguous):
E → E + T | T
T → T * F | F  
F → id
```

### 2. Precedence and Associativity Rules
**Approach:** Use precedence declarations to resolve conflicts automatically.

**Example in YACC/Bison:**
```yacc
%left '+'
%left '*'
%right UMINUS

expr: expr '+' expr
    | expr '*' expr
    | '-' expr %prec UMINUS
    | '(' expr ')'
    | ID
    ;
```

### 3. Parser Generator Directives
**Approach:** Use specific directives to tell the parser generator how to resolve conflicts.

**YACC/Bison examples:**
- `%expect n` - expect n shift-reduce conflicts
- `%glr-parser` - use GLR parsing for ambiguous grammars

### 4. Manual Conflict Resolution
**Approach:** Manually specify which action to take in conflict situations.

**Default resolution in many tools:**
- **Shift-Reduce:** Prefer shift (shift wins)
- **Reduce-Reduce:** Prefer the production that appears first in the grammar

---

## Detection and Analysis

### 1. Parser Generator Reports
Most parser generators (YACC, Bison, ANTLR) produce detailed reports showing:
- States where conflicts occur
- The specific productions involved
- The lookahead symbols causing conflicts

### 2. State Machine Analysis
Examine the LR automaton to identify problematic states:
```
State 7:
    E → E + E •
    E → E • + E
    
    On lookahead '+': shift/reduce conflict
```

### 3. Grammar Analysis Tools
Use tools to analyze grammar properties:
- Ambiguity checkers
- FIRST/FOLLOW set calculators
- LR table generators

---

## Impact on Parsing

### 1. Determinism Loss
Conflicts make parsing non-deterministic, requiring:
- Backtracking (expensive)
- Multiple parse trees (ambiguity)
- Error-prone parsing

### 2. Performance Issues
- Increased parsing time
- Memory overhead for multiple parse states
- Complexity in error handling

### 3. Semantic Ambiguity
Different parse trees may lead to different program meanings, causing:
- Incorrect program interpretation
- Unpredictable behavior
- Maintenance difficulties

---

## Summary

| Conflict Type | Cause | Resolution |
|---------------|-------|------------|
| **Shift-Reduce** | Ambiguous grammar, inadequate lookahead | Grammar rewriting, precedence rules |
| **Reduce-Reduce** | Multiple applicable productions | Grammar rewriting, eliminate ambiguity |
| **Accept-Reduce** | Unclear completion condition | Grammar restructuring |
| **Shift-Shift** | Multiple shift possibilities | Rare, usually tool-specific |

**Best Practices:**
1. Design unambiguous grammars when possible
2. Use precedence and associativity declarations appropriately
3. Analyze parser generator reports carefully
4. Test with comprehensive input sets
5. Consider using more powerful parsing techniques (GLR, Earley) for inherently ambiguous languages 