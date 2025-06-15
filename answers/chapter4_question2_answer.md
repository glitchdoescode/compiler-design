# Syntax-Directed Translation (SDT) Schemes

This document covers evaluation of expressions using SDT and construction of an SDT for infix to postfix translation.

---

## Question 2a: Expression Evaluation

### Given Grammar and Translation Rules

```
E → E1 # T      { E.value = E1.value * T.value }
E → T           { E.value = T.value }
T → T1 & F      { T.value = T1.value + F.value }
T → F           { T.value = F.value }
F → num         { F.value = num.value }
```

### Input Expression: `2 & 3 & 5 # 6 & 4`

### Step 1: Parse the Expression

**Operator Precedence:**
- `#` has lower precedence than `&`
- Both operators are left-associative

**Parse Tree:**
```
                E
              / | \
             E  #  T
             |   / | \
             T  T  &  F
           / | \ |     |
          T  &  F F    num(4)
        / | \   | |
       T  &  F  num(6)
       |     |
       F     F
       |     |
     num(2) num(3)
             |
           num(5)
```

Wait, let me re-parse this correctly. The expression is `2 & 3 & 5 # 6 & 4`.

Given the precedence (`&` higher than `#`), this should be parsed as:
`((2 & 3) & 5) # (6 & 4)`

**Correct Parse Tree:**
```
                    E
                  / | \
                 E  #  T
                 |   / | \
                 T  T  &  F
               / | \ |     |
              T  &  F F    num(4)
            / | \   | |
           T  &  F  num(6)
           |     |
           F     F
           |     |
         num(2) num(5)
                |
              num(3)
```

Actually, let me be more careful about the associativity. For `2 & 3 & 5`, with left associativity:

**Correct Parse Tree:**
```
                    E
                  / | \
                 E  #  T
                 |   / | \
                 T  T  &  F
               / | \ |     |
              T  &  F F    num(4)
            / | \   | |
           T  &  F  num(6)
           |     |
           F     F
           |     |
         num(2) num(5)
                |
              num(3)
```

Let me construct this step by step more carefully:

### Step 2: Detailed Parse Tree Construction

**Expression:** `2 & 3 & 5 # 6 & 4`

**Precedence:** `&` (higher) > `#` (lower)
**Associativity:** Both left-associative

**Parsing:**
1. `2 & 3 & 5` forms the left operand of `#`
2. `6 & 4` forms the right operand of `#`

**Left side: `2 & 3 & 5`**
- `(2 & 3) & 5` due to left associativity

**Right side: `6 & 4`**
- Simple binary operation

### Step 3: Complete Annotated Parse Tree

```
                        E
                      value=70
                    /    |    \
                   E     #     T
                value=10      value=7
                   |         / | \
                   T        T  &  F
                value=10  value=6  value=4
                 / | \       |      |
                T  &  F      F    num
             value=5 value=5 value=6 value=4
             / | \     |      |
            T  &  F    F    num
         value=2 value=3 value=5 value=6
            |     |     |
            F     F     F
         value=2 value=3 value=5
            |     |     |
          num   num   num
         value=2 value=3 value=5
```

### Step 4: Attribute Evaluation

**Bottom-up evaluation:**

1. **Leaf nodes:**
   - `num(2).value = 2`
   - `num(3).value = 3`
   - `num(5).value = 5`
   - `num(6).value = 6`
   - `num(4).value = 4`

2. **F nodes:**
   - `F₁.value = 2` (from num)
   - `F₂.value = 3` (from num)
   - `F₃.value = 5` (from num)
   - `F₄.value = 6` (from num)
   - `F₅.value = 4` (from num)

3. **T nodes (bottom level):**
   - `T₁.value = 2` (F → T)
   - `T₂.value = 3` (F → T)
   - `T₃.value = 5` (F → T)
   - `T₄.value = 6` (F → T)

4. **T nodes (& operations):**
   - `T₅.value = T₁.value + F₂.value = 2 + 3 = 5`
   - `T₆.value = T₅.value + F₃.value = 5 + 5 = 10`
   - `T₇.value = T₄.value + F₅.value = 6 + 4 = 10`

5. **E nodes:**
   - `E₁.value = T₆.value = 10`
   - `E.value = E₁.value * T₇.value = 10 * 10 = 100`

**Wait, let me recalculate the right side:**
- `T₇.value = T₄.value + F₅.value = 6 + 4 = 10`

**Final calculation:**
- `E.value = E₁.value * T₇.value = 10 * 10 = 100`

### Final Answer: E.value = 100

---

## Question 2b: Infix to Postfix Translation SDT

### Grammar for Infix to Postfix Translation

```
E → E1 + T      { E.code = E1.code || T.code || '+' }
E → E1 - T      { E.code = E1.code || T.code || '-' }
E → T           { E.code = T.code }
T → T1 * F      { T.code = T1.code || F.code || '*' }
T → T1 / F      { T.code = T1.code || F.code || '/' }
T → F           { T.code = F.code }
F → (E)         { F.code = E.code }
F → id          { F.code = id.lexeme }
F → num         { F.code = num.lexeme }
```

**Note:** `||` denotes string concatenation.

### Example Translation

**Input:** `a + b * c`

**Parse Tree with Translation:**
```
                E
            code="abc*+"
           /     |     \
          E      +      T
      code="a"       code="bc*"
         |           /   |   \
         T          T    *    F
      code="a"   code="b"   code="c"
         |          |         |
         F          F        id
      code="a"   code="b"   lexeme="c"
         |          |
        id         id
    lexeme="a"  lexeme="b"
```

**Step-by-step evaluation:**
1. `F.code = "a"` (from id)
2. `T.code = "a"` (T → F)
3. `E₁.code = "a"` (E → T)
4. `F.code = "b"` (from id)
5. `T₁.code = "b"` (T → F)
6. `F.code = "c"` (from id)
7. `T.code = T₁.code || F.code || '*' = "b" || "c" || "*" = "bc*"`
8. `E.code = E₁.code || T.code || '+' = "a" || "bc*" || "+" = "abc*+"`

### Enhanced SDT with Semantic Actions

```
E → E1 + T      { print('+'); E.code = E1.code || T.code || '+' }
E → E1 - T      { print('-'); E.code = E1.code || T.code || '-' }
E → T           { E.code = T.code }
T → T1 * F      { print('*'); T.code = T1.code || F.code || '*' }
T → T1 / F      { print('/'); T.code = T1.code || F.code || '/' }
T → F           { T.code = F.code }
F → (E)         { F.code = E.code }
F → id          { print(id.lexeme); F.code = id.lexeme }
F → num         { print(num.lexeme); F.code = num.lexeme }
```

### Alternative Implementation with Actions

For immediate output during parsing:

```
E → E1 + T      { printf("+"); }
E → E1 - T      { printf("-"); }
E → T           { /* no action */ }
T → T1 * F      { printf("*"); }
T → T1 / F      { printf("/"); }
T → F           { /* no action */ }
F → (E)         { /* no action */ }
F → id          { printf("%s ", id.lexeme); }
F → num         { printf("%s ", num.lexeme); }
```

### Complete Example: `(a + b) * c - d`

**Parse Tree:**
```
                    E
                  / | \
                 E  -  T
               / | \   |
              T  *  F  F
              |     |  |
              F     id id
              |   (c) (d)
             (E)
            / | \
           E  +  T
           |     |
           T     F
           |     |
           F     id
           |    (b)
          id
         (a)
```

**Translation Steps:**
1. Process `a`: output "a"
2. Process `b`: output "b"
3. Process `+`: output "+"
4. Process `c`: output "c"
5. Process `*`: output "*"
6. Process `d`: output "d"
7. Process `-`: output "-"

**Final Postfix:** `a b + c * d -`

### Implementation in C

```c
typedef struct {
    char* code;
} Attribute;

// Semantic actions
void concat3(Attribute* result, Attribute* a1, Attribute* a2, char* op) {
    int len = strlen(a1->code) + strlen(a2->code) + strlen(op) + 1;
    result->code = malloc(len);
    strcpy(result->code, a1->code);
    strcat(result->code, a2->code);
    strcat(result->code, op);
}

void copy_attr(Attribute* dest, Attribute* src) {
    dest->code = malloc(strlen(src->code) + 1);
    strcpy(dest->code, src->code);
}

void set_attr(Attribute* attr, char* value) {
    attr->code = malloc(strlen(value) + 1);
    strcpy(attr->code, value);
}

// Grammar rules implementation
void rule_E_plus_T(Attribute* E, Attribute* E1, Attribute* T) {
    concat3(E, E1, T, "+");
}

void rule_E_T(Attribute* E, Attribute* T) {
    copy_attr(E, T);
}

void rule_T_mult_F(Attribute* T, Attribute* T1, Attribute* F) {
    concat3(T, T1, F, "*");
}

void rule_F_id(Attribute* F, char* id_lexeme) {
    set_attr(F, id_lexeme);
}
```

### L-attributed Version

For LL parsers, we can use an L-attributed version:

```
E → T E'        { E'.inh = T.code; E.code = E'.syn }
E' → + T E'₁    { E'₁.inh = E'.inh || T.code || '+'; E'.syn = E'₁.syn }
E' → - T E'₁    { E'₁.inh = E'.inh || T.code || '-'; E'.syn = E'₁.syn }
E' → ε          { E'.syn = E'.inh }
T → F T'        { T'.inh = F.code; T.code = T'.syn }
T' → * F T'₁    { T'₁.inh = T'.inh || F.code || '*'; T'.syn = T'₁.syn }
T' → / F T'₁    { T'₁.inh = T'.inh || F.code || '/'; T'.syn = T'₁.syn }
T' → ε          { T'.syn = T'.inh }
F → (E)         { F.code = E.code }
F → id          { F.code = id.lexeme }
```

---

## Summary

### Question 2a Result
For the expression `2 & 3 & 5 # 6 & 4`:
- **Parse structure:** `((2 & 3) & 5) # (6 & 4)`
- **Left side evaluation:** `(2 + 3) + 5 = 10`
- **Right side evaluation:** `6 + 4 = 10`
- **Final result:** `10 * 10 = 100`

### Question 2b Key Points
- **SDT for postfix translation** uses synthesized attributes
- **String concatenation** builds the postfix expression
- **Operator precedence** in grammar ensures correct translation
- **Can be implemented** in both S-attributed and L-attributed forms
- **Immediate output** version prints postfix during parsing 