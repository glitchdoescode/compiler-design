# Recursive Descent Parser with Backtracking

This document explains and provides a pseudocode implementation for a Recursive Descent Parser with backtracking for the given grammar.

**Grammar:**
```
S -> aSbS | bSaS | ε
```

---

## 1. Concept of a Recursive Descent Parser with Backtracking

A **Recursive Descent Parser (RDP)** is a top-down parsing technique that uses a set of mutually recursive procedures to process the input. Each non-terminal in the grammar corresponds to a procedure. The parser attempts to "prove" that the input string can be derived from the start symbol.

When a grammar is not suitable for predictive parsing (i.e., it's not LL(1)), a simple RDP can get stuck. This happens when there are multiple production choices for a non-terminal, and the parser cannot decide which one to take based on the lookahead token.

This is where **backtracking** comes in. A backtracking RDP works as follows:

1.  To parse a non-terminal `A`, the parser systematically tries each production for `A` in order.
2.  It chooses the first production, say `A -> α`, and tries to match `α` with the current input.
3.  If `α` is successfully matched, the procedure for `A` returns `success`.
4.  If `α` fails to match the input at any point, the parser "backtracks". It resets the input pointer to where it was before it attempted this production and then tries the *next* available production for `A`.
5.  If all productions for `A` have been tried and have failed, the procedure for `A` returns `failure`.

This approach is essentially a depth-first search of the parse tree. It is more powerful than non-backtracking RDPs but can be very inefficient and is not typically used in production compilers.

The given grammar `S -> aSbS | bSaS | ε` requires backtracking because for the non-terminal `S`, productions `aSbS` and `bSaS` start with different terminals, but the `ε` production means that if the current input doesn't start with `a` or `b`, `ε` could be a choice. More complex situations could arise where a simple lookahead isn't enough.

---

## 2. Pseudocode for the Parser

Here is a conceptual implementation of a backtracking recursive descent parser for the grammar. We'll use a global `input` string and a global `cursor` to keep track of our position in the string.

```pseudocode
// --- Global State ---
string input; // The input string to be parsed, e.g., "ab"
int cursor;   // The current position (index) in the input string

// --- Main Parsing Function ---
// Returns true if the entire input string is successfully parsed, false otherwise.
function main_parser(string to_parse):
    input = to_parse;
    cursor = 0;
    
    // Call the procedure for the start symbol 'S'
    if (S() AND cursor == input.length):
        // Parsing is successful only if S() returns true AND the entire input was consumed.
        print "String successfully parsed."
        return true;
    else:
        print "Parsing failed."
        return false;
    end if
end function

// --- Procedure for non-terminal S ---
// Tries to match one of the productions for S at the current cursor position.
// Returns true on success, false on failure.
function S():
    // Save the current cursor position in case we need to backtrack.
    int backtrack_position = cursor;

    // --- Try rule 1: S -> aSbS ---
    if (match('a')):
        if (S()):
            if (match('b')):
                if (S()):
                    // Successfully parsed 'aSbS'
                    return true;
                end if
            end if
        end if
    end if
    
    // If we reach here, the first rule failed. We must backtrack.
    cursor = backtrack_position;

    // --- Try rule 2: S -> bSaS ---
    if (match('b')):
        if (S()):
            if (match('a')):
                if (S()):
                    // Successfully parsed 'bSaS'
                    return true;
                end if
            end if
        end if
    end if

    // If we reach here, the second rule also failed. We must backtrack.
    cursor = backtrack_position;

    // --- Try rule 3: S -> ε ---
    // The empty string always matches successfully without consuming input.
    return true;

end function

// --- Helper function to match a terminal ---
// Consumes one character from the input if it matches the expected terminal.
// Returns true if it matches, false otherwise.
function match(char expected_terminal):
    if (cursor < input.length AND input[cursor] == expected_terminal):
        cursor = cursor + 1; // Consume the character
        return true;
    else:
        return false; // No match
    end if
end function
```

### How it Works (Example Walkthrough)

Let's trace the parsing of the input string `"ab"`:

1.  `main_parser("ab")` is called. `cursor` is `0`.
2.  `main_parser` calls `S()`. `backtrack_position` is `0`.

3.  **S() tries rule 1 (`aSbS`):**
    *   `match('a')`: `input[0]` is 'a'. Success. `cursor` becomes `1`.
    *   Calls `S()` recursively. Let's call this `S_inner1`.
        *   `S_inner1` `backtrack_position` is `1`.
        *   `S_inner1` tries rule 1 (`aSbS`):
            *   `match('a')`: `input[1]` is 'b', not 'a'. Fail.
        *   `S_inner1` backtracks: `cursor` resets to `1`.
        *   `S_inner1` tries rule 2 (`bSaS`):
            *   `match('b')`: `input[1]` is 'b'. Success. `cursor` becomes `2`.
            *   Calls `S()` recursively (`S_inner2`).
                *   `S_inner2` `backtrack_position` is `2`.
                *   `S_inner2` tries rule 1: `match('a')` fails (`cursor` is at end of string).
                *   `S_inner2` backtracks: `cursor` resets to `2`.
                *   `S_inner2` tries rule 2: `match('b')` fails.
                *   `S_inner2` backtracks: `cursor` resets to `2`.
                *   `S_inner2` tries rule 3 (`ε`): Success. Returns `true`. `cursor` remains `2`.
            *   `S_inner1` continues from the `bS` part. Needs `aS`.
            *   `match('a')`: `input[2]` is out of bounds. Fail.
        *   `S_inner1` backtracks: `cursor` resets to `1`.
        *   `S_inner1` tries rule 3 (`ε`): Success. Returns `true`. `cursor` remains `1`.
    *   Back in the first `S()` call, the first recursive `S()` returned `true`. Now it needs `match('b')`.
    *   `match('b')`: `input[1]` is 'b'. Success. `cursor` becomes `2`.
    *   Calls `S()` recursively (`S_inner3`).
        *   This `S_inner3` will try rules 1 and 2, which will fail as the cursor is at the end.
        *   It will succeed on rule 3 (`ε`) and return `true`. `cursor` remains `2`.
    *   The first `S()` call successfully matched `aSbS` where both recursive `S` calls matched `ε`.
    *   The first `S()` returns `true`.

4.  Back in `main_parser`, `S()` returned `true`. It checks `cursor == input.length`.
    *   `cursor` is `2`, `input.length` is `2`. The condition is `true`.
5.  Parsing succeeds.

This example shows the backtracking process: when a path fails, the `cursor` is reset, and the next alternative is tried. 