# Recursive Descent Parser with Backtracking

This document constructs a Recursive Descent Parser with backtracking for the given grammar.

---

## Given Grammar

```
S → aSbS | bSaS | ε
```

---

## Understanding the Grammar

This grammar generates strings where:
- Each 'a' is eventually followed by a 'b' somewhere in the string
- Each 'b' is eventually followed by an 'a' somewhere in the string  
- The empty string is also valid
- Examples of valid strings: ε, ab, ba, abab, baba, aabb, bbaa, etc.

---

## Recursive Descent Parser Structure

A recursive descent parser has one function for each non-terminal in the grammar. Since our grammar has only one non-terminal (S), we need one parsing function.

### Basic Algorithm

For each production A → α₁ | α₂ | ... | αₙ, the parsing function for A:
1. Try each alternative αᵢ in order
2. If αᵢ succeeds, return success
3. If αᵢ fails, backtrack and try αᵢ₊₁
4. If all alternatives fail, return failure

---

## Implementation

### Global Variables

```c
char *input;        // Input string
int pos;           // Current position in input
int length;        // Length of input string
```

### Utility Functions

```c
// Save current position for backtracking
int save_position() {
    return pos;
}

// Restore position for backtracking
void restore_position(int saved_pos) {
    pos = saved_pos;
}

// Check if we've consumed all input
int at_end() {
    return pos >= length;
}

// Match a specific character
int match_char(char c) {
    if (pos < length && input[pos] == c) {
        pos++;
        return 1;  // Success
    }
    return 0;  // Failure
}
```

### Main Parsing Function

```c
// Parse non-terminal S
// S → aSbS | bSaS | ε
int parse_S() {
    int saved_pos;
    
    // Try first alternative: S → aSbS
    saved_pos = save_position();
    if (match_char('a') && parse_S() && match_char('b') && parse_S()) {
        return 1;  // Success
    }
    restore_position(saved_pos);  // Backtrack
    
    // Try second alternative: S → bSaS
    saved_pos = save_position();
    if (match_char('b') && parse_S() && match_char('a') && parse_S()) {
        return 1;  // Success
    }
    restore_position(saved_pos);  // Backtrack
    
    // Try third alternative: S → ε
    // ε always succeeds without consuming input
    return 1;  // Success
}
```

### Complete Parser

```c
#include <stdio.h>
#include <string.h>

char *input;
int pos;
int length;

int save_position() {
    return pos;
}

void restore_position(int saved_pos) {
    pos = saved_pos;
}

int at_end() {
    return pos >= length;
}

int match_char(char c) {
    if (pos < length && input[pos] == c) {
        pos++;
        return 1;
    }
    return 0;
}

int parse_S() {
    int saved_pos;
    
    // Alternative 1: S → aSbS
    saved_pos = save_position();
    if (match_char('a') && parse_S() && match_char('b') && parse_S()) {
        return 1;
    }
    restore_position(saved_pos);
    
    // Alternative 2: S → bSaS  
    saved_pos = save_position();
    if (match_char('b') && parse_S() && match_char('a') && parse_S()) {
        return 1;
    }
    restore_position(saved_pos);
    
    // Alternative 3: S → ε
    return 1;  // ε always succeeds
}

int parse(char *str) {
    input = str;
    pos = 0;
    length = strlen(str);
    
    if (parse_S() && at_end()) {
        return 1;  // Success: parsed entire input
    }
    return 0;  // Failure
}

int main() {
    char test_strings[][20] = {
        "",        // ε
        "ab",      // aSbS with S→ε, S→ε
        "ba",      // bSaS with S→ε, S→ε
        "abab",    // aSbS with S→ab, S→ε
        "baba",    // bSaS with S→ab, S→ε
        "aabb",    // aSbS with S→ε, S→ab
        "bbaa",    // bSaS with S→ε, S→ab
        "abc",     // Invalid
        "aab"      // Invalid
    };
    
    int num_tests = sizeof(test_strings) / sizeof(test_strings[0]);
    
    for (int i = 0; i < num_tests; i++) {
        printf("String \"%s\": %s\n", 
               test_strings[i], 
               parse(test_strings[i]) ? "ACCEPTED" : "REJECTED");
    }
    
    return 0;
}
```

---

## Trace Examples

### Example 1: Input "ab"

```
parse_S():
  Try S → aSbS:
    match_char('a'): Success, pos = 1
    parse_S(): Try S → aSbS: match_char('a') fails (input[1] = 'b')
               Try S → bSaS: match_char('b') fails (input[1] = 'b')
               Try S → ε: Success
    match_char('b'): Success, pos = 2  
    parse_S(): Try S → ε: Success
    Return Success
```

### Example 2: Input "abc" (Invalid)

```
parse_S():
  Try S → aSbS:
    match_char('a'): Success, pos = 1
    parse_S(): Try S → ε: Success
    match_char('b'): Success, pos = 2
    parse_S(): Try S → ε: Success
    Return Success
  
Main parser: parse_S() succeeded but pos = 2, not at end (length = 3)
Return Failure
```

### Example 3: Input "abab"

```
parse_S():
  Try S → aSbS:
    match_char('a'): Success, pos = 1
    parse_S(): 
      Try S → aSbS: match_char('a') fails (input[1] = 'b')
      Try S → bSaS: match_char('b') succeeds, pos = 2
                    parse_S() → ε succeeds
                    match_char('a') succeeds, pos = 3
                    parse_S() → ε succeeds
                    Return Success
    match_char('b'): Success, pos = 4
    parse_S(): Try S → ε: Success
    Return Success
```

---

## Optimizations and Considerations

### 1. Left Recursion
This grammar doesn't have left recursion, so the parser won't enter infinite loops.

### 2. Performance
- **Time Complexity:** Exponential in worst case due to backtracking
- **Space Complexity:** O(n) for recursion stack

### 3. Memoization
For better performance, we could add memoization:

```c
// Memoization table
int memo[MAX_INPUT_LENGTH][NUM_NONTERMINALS];

int parse_S_memo() {
    if (memo[pos][S_INDEX] != -1) {
        return memo[pos][S_INDEX];
    }
    
    int result = parse_S();
    memo[pos][S_INDEX] = result;
    return result;
}
```

### 4. Error Reporting
Enhanced version with error reporting:

```c
int parse_S_with_errors(int depth) {
    printf("%*sTrying S at position %d\n", depth*2, "", pos);
    
    int saved_pos = save_position();
    
    // Try S → aSbS
    printf("%*sTrying S → aSbS\n", depth*2, "");
    if (match_char('a') && parse_S_with_errors(depth+1) && 
        match_char('b') && parse_S_with_errors(depth+1)) {
        printf("%*sS → aSbS succeeded\n", depth*2, "");
        return 1;
    }
    restore_position(saved_pos);
    
    // Try S → bSaS
    printf("%*sTrying S → bSaS\n", depth*2, "");
    if (match_char('b') && parse_S_with_errors(depth+1) && 
        match_char('a') && parse_S_with_errors(depth+1)) {
        printf("%*sS → bSaS succeeded\n", depth*2, "");
        return 1;
    }
    restore_position(saved_pos);
    
    // Try S → ε
    printf("%*sTrying S → ε\n", depth*2, "");
    printf("%*sS → ε succeeded\n", depth*2, "");
    return 1;
}
```

---

## Summary

The recursive descent parser with backtracking:

1. **Systematically tries all alternatives** for each non-terminal
2. **Backtracks when an alternative fails** and tries the next one
3. **Uses recursion** to handle nested structures naturally
4. **Guarantees finding a parse** if one exists
5. **Has exponential time complexity** in the worst case due to backtracking

This approach is simple to implement and understand, making it suitable for educational purposes and small grammars, but not practical for large-scale parsing due to performance concerns. 