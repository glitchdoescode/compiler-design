# Attributes in Syntax-Directed Translation

This document explains synthesized and inherited attributes, S-attributed and L-attributed SDTs, and provides examples of annotated parse trees and dependency graphs.

---

## a) Synthesized vs Inherited Attributes

### Synthesized Attributes

**Definition:** A synthesized attribute is an attribute whose value is computed from the values of attributes of its children in the parse tree. The value flows upward from leaves to root.

**Characteristics:**
- Computed bottom-up
- Value depends on child nodes
- Natural for bottom-up parsing
- Easy to implement

**Example:**

Grammar for arithmetic expressions:
```
E → E₁ + T    { E.value = E₁.value + T.value }
E → T         { E.value = T.value }
T → T₁ * F    { T.value = T₁.value * F.value }
T → F         { T.value = F.value }
F → (E)       { F.value = E.value }
F → num       { F.value = num.lexval }
```

**Parse tree for `3 + 4 * 5`:**

```
        E (23)
       / | \
   E(3)  +  T(20)
    |      / | \
   T(3)  T(4) * F(5)
    |     |     |
   F(3)  F(4)  num(5)
    |     |
  num(3) num(4)
```

All attributes flow upward (synthesized):
- `F.value = 5` (from num)
- `T.value = 4 * 5 = 20`
- `E.value = 3 + 20 = 23`

### Inherited Attributes

**Definition:** An inherited attribute is an attribute whose value is computed from the values of attributes of its parent and/or siblings in the parse tree. The value flows downward from root to leaves.

**Characteristics:**
- Computed top-down
- Value depends on parent/sibling nodes
- Natural for top-down parsing
- More complex to implement

**Example:**

Grammar for variable declarations with type inheritance:
```
D → T L       { L.type = T.type }
T → int       { T.type = integer }
T → float     { T.type = real }
L → L₁, id    { L₁.type = L.type; addtype(id.entry, L.type) }
L → id        { addtype(id.entry, L.type) }
```

**Parse tree for `int x, y`:**

```
        D
       / \
   T(int) L(int)
      |   / | \
    int  L(int) , id(y)
         |
       id(x)
```

The `type` attribute flows downward (inherited):
- `T.type = integer`
- `L.type = integer` (inherited from T)
- Both `x` and `y` get type `integer`

---

## b) S-attributed and L-attributed SDT

### S-attributed SDT (Synthesized)

**Definition:** An S-attributed SDT is one that uses only synthesized attributes. All attributes are computed bottom-up.

**Characteristics:**
- Only synthesized attributes
- Can be evaluated during bottom-up parsing
- Simple to implement
- Natural for LR parsers

**Example:**

```
E → E₁ + T    { E.val = E₁.val + T.val }
E → T         { E.val = T.val }
T → T₁ * F    { T.val = T₁.val * F.val }
T → F         { T.val = F.val }
F → (E)       { F.val = E.val }
F → num       { F.val = num.value }
```

**Evaluation:** All attributes computed after reducing each production.

### L-attributed SDT (Left-to-right)

**Definition:** An L-attributed SDT is one where:
1. All synthesized attributes are allowed
2. Inherited attributes are allowed, but with restrictions:
   - For production A → X₁X₂...Xₙ
   - Inherited attribute Xᵢ.a can depend only on:
     - Attributes of X₁, X₂, ..., Xᵢ₋₁ (symbols to the left)
     - Inherited attributes of A

**Characteristics:**
- Mix of synthesized and inherited attributes
- Can be evaluated during top-down parsing
- More powerful than S-attributed
- Natural for LL parsers

**Example:**

```
D → T L       { L.type = T.type }
T → int       { T.type = integer }
T → float     { T.type = real }
L → L₁, id    { L₁.type = L.type; addtype(id.entry, L.type); L.count = L₁.count + 1 }
L → id        { addtype(id.entry, L.type); L.count = 1 }
```

**Valid because:**
- `L.type` is inherited from parent
- `L₁.type` depends on `L.type` (from parent)
- `L.count` is synthesized from `L₁.count`

---

## c) Annotated Parse Tree and Dependency Graph

### Annotated Parse Tree

**Definition:** An annotated parse tree is a parse tree where each node is annotated with the values of its attributes.

**Example:**

Grammar:
```
E → E₁ + T    { E.val = E₁.val + T.val }
E → T         { E.val = T.val }
T → F * T₁    { T.val = F.val * T₁.val }
T → F         { T.val = F.val }
F → (E)       { F.val = E.val }
F → num       { F.val = num.value }
```

**Annotated parse tree for `2 * 3 + 4`:**

```
           E
         val=10
        /  |  \
    E      +    T
  val=6       val=4
    |           |
    T           F
  val=6       val=4
  / | \         |
 F  *  T      num
val=2 val=3   val=4
 |     |
num   F
val=2 val=3
       |
      num
      val=3
```

**Attribute Values:**
- Leaf nodes: `num.val = lexical value`
- Internal nodes: computed using semantic rules
- Root: `E.val = 10` (final result)

### Dependency Graph

**Definition:** A dependency graph shows the dependencies between attribute instances in a parse tree. An edge from attribute `b` to attribute `c` means that `c` depends on `b`.

**Example:**

For the same expression `2 * 3 + 4`:

**Dependency Graph:**
```
num₁.val → F₁.val → T₁.val → E₁.val → E.val
num₂.val → F₂.val → T₁.val ↗
num₃.val → F₃.val → T₂.val → E.val
```

**Detailed Dependencies:**
1. `F₁.val` depends on `num₁.val` (2)
2. `F₂.val` depends on `num₂.val` (3)  
3. `T₁.val` depends on `F₁.val` and `F₂.val`
4. `E₁.val` depends on `T₁.val`
5. `F₃.val` depends on `num₃.val` (4)
6. `T₂.val` depends on `F₃.val`
7. `E.val` depends on `E₁.val` and `T₂.val`

**Topological Order for Evaluation:**
1. `num₁.val`, `num₂.val`, `num₃.val`
2. `F₁.val`, `F₂.val`, `F₃.val`
3. `T₁.val`, `T₂.val`
4. `E₁.val`
5. `E.val`

### Complex Example with Inherited Attributes

**Grammar:**
```
D → T L       { L.type = T.type }
T → int       { T.type = integer }
L → L₁, id    { L₁.type = L.type; addtype(id.entry, L.type) }
L → id        { addtype(id.entry, L.type) }
```

**Annotated Parse Tree for `int x, y`:**

```
        D
       / \
   T      L
type=int type=int
   |     / | \
  int   L   , id(y)
        type=int
         |
       id(x)
```

**Dependency Graph:**
```
T.type → L.type → L₁.type
       ↘        ↘
         addtype(y) addtype(x)
```

**Evaluation Order:**
1. `T.type = integer`
2. `L.type = T.type` (inherited)
3. `L₁.type = L.type` (inherited)
4. `addtype(x.entry, L₁.type)`
5. `addtype(y.entry, L.type)`

---

## Applications and Implementation

### Translation During Parsing

**S-attributed (Bottom-up):**
```c
// During LR parsing
case REDUCE_E_PLUS_T:
    E.val = E1.val + T.val;
    break;
```

**L-attributed (Top-down):**
```c
// During LL parsing
void parseL(int inherited_type) {
    if (lookahead == ID) {
        addtype(id.entry, inherited_type);
        match(ID);
    } else if (lookahead == ID && peek() == COMMA) {
        parseL(inherited_type);  // Pass inherited attribute
        match(COMMA);
        addtype(id.entry, inherited_type);
        match(ID);
    }
}
```

### Attribute Storage

**Stack-based (S-attributed):**
- Attributes stored on parsing stack
- Computed during reductions

**Tree-based (L-attributed):**
- Attributes stored in parse tree nodes
- Multiple passes may be needed

---

## Summary

| Aspect | Synthesized | Inherited |
|--------|-------------|-----------|
| **Direction** | Bottom-up | Top-down |
| **Dependencies** | Children → Parent | Parent/Siblings → Node |
| **Parsing** | Natural for LR | Natural for LL |
| **Implementation** | Simpler | More complex |
| **Power** | Limited | More expressive |

| SDT Type | Attributes | Evaluation | Parsing |
|----------|------------|------------|---------|
| **S-attributed** | Only synthesized | Single bottom-up pass | LR parsers |
| **L-attributed** | Synthesized + restricted inherited | Single left-to-right pass | LL parsers |

**Key Points:**
- Synthesized attributes flow upward, inherited attributes flow downward
- S-attributed SDTs are simpler but less expressive
- L-attributed SDTs provide good balance of power and implementability
- Dependency graphs help visualize attribute relationships and evaluation order 