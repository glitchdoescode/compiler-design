# B.Tech., VI Semester Examination, May 2024: Compiler Design

**Maximum Marks:** 70
**Time:** Three Hours

**Note:**
i) Attempt any five questions.
ii) All questions carry equal marks.
iii) In case of any doubt or dispute the English version question should be treated as final.

---

## Question 1
Discuss the analysis and synthesis model of compilation. Explain each phase by using the input: `a = (b + c) * (b + c) * 2`.

---

## Question 2
a) Explain the various Compiler Construction Tools.
b) What are the issues to be considered in the design of a lexical analyzer? Why is the buffering of input used in lexical analysis?

---

## Question 3
a) Differentiate Top Down Parser and Bottom Up Parser. Give Example for each.
b) Prepare and Eliminate the left recursion for the grammar
```
S -> Aa | b
A -> Ac | Sd | ε
```

---

## Question 4
a) Consider the grammar
```
E -> E+T
E -> T
T -> T*F
T -> F
F -> (E)
F -> id
```
Construct a LALR parsing table for the grammar given above. Verify whether the input string `id + id * id` is accepted by the grammar or not.

b) What are the various conflicts that occur during shift reduce parsing?

---

## Question 5
a) What is a DAG? Draw the DAG for the statement `a = (a * b + c) - (a * b + c)`.
b) Explain synthesized attribute and inherited attribute with suitable examples.

---

## Question 6
a) Discuss the various storage allocation strategies in detail.
b) Write the three address code sequence for the assignment statement. `P := (X + Y) * (X - C)`.

---

## Question 7
a) Represent the following in flow graph: `i=1; sum=0; while(i<=10){sum=sum+i; i=i+1;}`.
b) What are the different sources of optimization of basic blocks?

---

## Question 8
Write brief notes: (any three)
a) Peephole Optimization
b) Global data flow equations
c) Dynamic Storage Allocation
d) Error Detection and Recovery 