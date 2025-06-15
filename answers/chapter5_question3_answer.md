# Type Systems: Equivalence and Polymorphism

This document covers two key concepts in programming language type systems: the methods for checking type equivalence and the different forms of polymorphism.

---

## a) Equivalence of Type Expressions

A **type expression** is a representation of the type of a language construct. A simple type like `int` is a type expression, as is a more complex one like `array(1..10, pointer(int))`. A compiler's type checker must often determine if two type expressions are equivalent. For example, in an assignment `x = y`, the type of `x` must be equivalent to the type of `y`.

There are two main methods for determining type equivalence:

### 1. Structural Equivalence

Under **structural equivalence**, two type expressions are considered equivalent if they have the **exact same structure**. The names of the types are irrelevant. The compiler recursively compares the structures of the two types to see if they are identical.

**Example:**
Consider the following C-like declarations:
```c
typedef struct {
    int x;
    float y;
} Point;

typedef struct {
    int x;
    float y;
} Coordinate;

Point p;
Coordinate c;
```
Under structural equivalence, the types of `p` and `c` are considered **equivalent**. Even though they were declared with different type names (`Point` and `Coordinate`), their underlying structure (a struct with an `int` followed by a `float`) is identical. An assignment `p = c;` would be legal.

*   **Pros:** Flexible and allows for interoperability between types that are structurally the same but have different names.
*   **Cons:** Can sometimes lead to unintentional type equivalences where the programmer intended for two types to be distinct, even if they share a structure. It can also be complex to implement the recursive comparison.

### 2. Name Equivalence

Under **name equivalence**, two type expressions are equivalent only if they share the **exact same type name**. Each `typedef` or type definition is treated as introducing a new, distinct type.

**Example:**
Using the same code as above:
```c
typedef struct {
    int x;
    float y;
} Point;

typedef struct {
    int x;
    float y;
} Coordinate;

Point p;
Coordinate c;
```
Under strict name equivalence, the types of `p` and `c` are **not equivalent**. `Point` and `Coordinate` are treated as two separate, incompatible types. An assignment `p = c;` would be a type error.

*   **Pros:** Simpler to implement and provides stricter type checking, preventing accidental equivalences. The programmer's intent to create a distinct type is always respected.
*   **Cons:** Less flexible. Requires explicit casting or conversion functions to use values of one type where another, structurally identical type is expected.

Most modern languages use a combination of these or lean towards name equivalence for user-defined types (like `structs` and `classes`) for its safety.

---

## b) Polymorphic Functions

**Polymorphism** (from Greek, meaning "many forms") is the ability of a function, object, or variable to take on multiple forms. In the context of functions, it means a single function name can be used with different types of arguments.

There are three main categories of polymorphism:

### 1. Parametric Polymorphism (Generics)

A function exhibits **parametric polymorphism** if it can work on a wide range of types without its code being dependent on the specifics of those types. The type is passed as a parameter, either explicitly or implicitly. This is the foundation of "generics" in many languages.

**Example (C++ Template):**
```cpp
template<typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

int i = max(5, 10);       // T is deduced as int
float f = max(3.14, 2.71); // T is deduced as float
string s = max("apple", "banana"); // T is deduced as string
```
The function `max` is written once, but it works for any type `T` that supports the `>` operator. The logic is generic and independent of the specific type.

### 2. Ad-hoc Polymorphism (Overloading)

A function is **ad-hoc polymorphic** if there are multiple, distinct implementations of the function that share the same name. The compiler selects the correct implementation to call based on the number and/or types of the arguments provided. This is commonly known as **function overloading**.

**Example (C++):**
```cpp
// One implementation for integers
int add(int a, int b) {
    return a + b;
}

// A separate implementation for doubles
double add(double a, double b) {
    return a + b;
}

// A third implementation for strings
string add(string a, string b) {
    return a + b;
}

add(3, 4);       // Calls the integer version
add(3.5, 4.5);   // Calls the double version
add("hello ", "world"); // Calls the string version
```
Unlike parametric polymorphism, there is no single generic implementation. There are three separate functions that just happen to share the same name.

### 3. Subtype Polymorphism (Inclusion Polymorphism)

**Subtype polymorphism** is the ability of a function to operate on values of a certain type as well as values of any of its subtypes. This is the cornerstone of object-oriented programming, where a function designed to work with a base class object can also work with any derived class object.

**Example (Java):**
```java
class Animal {
    public void makeSound() {
        System.out.println("Some generic animal sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow");
    }
}

// This function is polymorphic
public void doAnimalSound(Animal animal) {
    animal.makeSound(); // The actual method called depends on the object's runtime type
}

doAnimalSound(new Dog()); // Prints "Woof"
doAnimalSound(new Cat()); // Prints "Meow"
```
The function `doAnimalSound` is written once to work with any `Animal`, but its behavior changes depending on the subtype of the object passed to it. 