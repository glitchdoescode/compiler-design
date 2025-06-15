# B.Tech., VI Semester Examination, December 2024: Compiler Design

**Maximum Marks:** 70

**Note:**
i) Attempt any five questions.
ii) All questions carry equal marks.
iii) Assume missing data, if any, suitably.
iv) In case of any doubt or dispute the English version question should be treated as final.

---

## Question 1

### (a) 
What is a translator? Compare compiler and interpreter in terms of translation method, memory requirement and speed. *(8 marks)*

### (b) 
Explain LEX tool in brief. *(3 marks)*

### (c) 
What is left factoring in grammar? Explain with an example. *(3 marks)*

---

## Question 2

### (a) 
Consider the following grammar for list structures:
```
S -> a | ^ | (T)
T -> T,S | S
```
Find left most derivation, right most derivation and parse tree for the string `((a,a),^(a)),a`. *(6 marks)*

### (b)
For the following grammar find First and Follow sets for each of non-terminal.
```
S -> aAB | bA | ε
A -> aAb | ε
B -> bB | ε
```
*(4 marks)*

### (c)
Describe the following in brief:
i) Ambiguity
ii) Left recursion
*(4 marks)*

---

## Question 3

### (a)
What is parser? Explain backtracking and non-backtracking parsers with their types. *(8 marks)*

### (b)
Consider the following grammar.
```
E -> E + T | T
T -> T * F | F
F -> F^ | a | b
```
Construct the SLR Parsing table for this grammar. *(6 marks)*

---

## Question 4

### (a)
Define the following with an example.
i) Synthesized attributes
ii) Annotated parse tree
iii) Dependency graph
*(8 marks)*

### (b)
Generate the three address code for the following code segment:
```
While (a < c and b < d) do
  If a=1 then c=c+1.
  Else
    While (a <= 4) do a = a + 3
```
*(6 marks)*

---

## Question 5

### (a)
Explain various intermediate code generation techniques in brief. *(8 marks)*

### (b)
Consider the grammar with the following translation rules and E as the start symbol.
```
E -> E1 # T      { E.value = E1.value * T.value }
T -> T1 & F      { T.value = T1.value + F.value }
T -> F           { T.value = F.value }
F -> num         { F.value = num.value }
```
Compute E.value for the root of the parse tree for the expression `2 & 3 & 5 # 6 & 4`. *(6 marks)*

---

## Question 6

### (a)
Write an algorithm to construct a DAG from a basic block. *(6 marks)*

### (b)
What is S-attributed SDT & L-attributed SDT? Consider the following grammar and write the SDT rules for the given grammar.
```
S -> S + A
S -> A
A -> A + B
A -> B
B -> (S)
B -> id
```
*(8 marks)*

---

## Question 7

### (a)
Write the differences between Synthesized and Inherited attributes. *(7 marks)*

### (b)
Explain the common sub expression, elimination, copy propagation, and transformation for moving loop invariant computations in detail. *(7 marks)*

---

## Question 8

### (a)
What is Local transformation and Global transformation? *(4 marks)*

### (b)
What are the 3 areas of code optimization? Explain each one in detail. *(10 marks)* 