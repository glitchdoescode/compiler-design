# Type Systems

This document explains type systems in compilers, focusing on the equivalence of type expressions and polymorphic functions with detailed examples.

---

## a) Equivalence of Type Expressions

### Introduction to Type Expressions

**Type expressions** are formal representations of data types in programming languages. They describe the structure and properties of data that variables can hold and operations can manipulate.

### Basic Type Expressions

#### 1. Primitive Types
```
int, float, char, bool, void
```

#### 2. Constructed Types
```
array(T)     - Array of type T
pointer(T)   - Pointer to type T
function(T₁ × T₂ × ... × Tₙ → T) - Function type
record(f₁:T₁, f₂:T₂, ..., fₙ:Tₙ) - Record/struct type
```

#### 3. Type Expressions Examples
```c
// C declarations and their type expressions
int x;                    // int
int* p;                   // pointer(int)
int arr[10];              // array(int)
int func(int, float);     // function(int × float → int)
struct Point {            // record(x:int, y:int)
    int x, y;
};
```

### Type Equivalence Concepts

**Type equivalence** determines when two type expressions represent the same type. This is crucial for:
- Type checking in assignments
- Function parameter matching
- Operator overloading resolution
- Memory layout compatibility

### Types of Type Equivalence

#### 1. Structural Equivalence

**Definition:** Two types are structurally equivalent if they have the same structure, regardless of the names used to define them.

**Examples:**

```c
// Structural equivalence examples
typedef struct {
    int x, y;
} Point1;

typedef struct {
    int x, y;
} Point2;

// Under structural equivalence:
// Point1 ≡ Point2 (same structure: record(x:int, y:int))
```

**Algorithm for Structural Equivalence:**

```c
bool structurally_equivalent(TypeExpression* t1, TypeExpression* t2) {
    if (t1 == t2) return true;
    if (t1->kind != t2->kind) return false;
    
    switch (t1->kind) {
        case TYPE_PRIMITIVE:
            return t1->primitive == t2->primitive;
            
        case TYPE_POINTER:
            return structurally_equivalent(t1->pointed_type, t2->pointed_type);
            
        case TYPE_ARRAY:
            return (t1->array_size == t2->array_size) &&
                   structurally_equivalent(t1->element_type, t2->element_type);
                   
        case TYPE_FUNCTION:
            if (t1->param_count != t2->param_count) return false;
            if (!structurally_equivalent(t1->return_type, t2->return_type)) 
                return false;
            for (int i = 0; i < t1->param_count; i++) {
                if (!structurally_equivalent(t1->param_types[i], 
                                           t2->param_types[i])) 
                    return false;
            }
            return true;
    }
    return false;
}
```

**Advantages:**
- Flexible: Allows interchangeability of structurally identical types
- Natural: Matches programmer intuition
- Supports anonymous types

**Disadvantages:**
- Expensive: Requires deep structural comparison
- May allow unintended type mixing
- Complex for recursive types

#### 2. Name Equivalence

**Definition:** Two types are name equivalent if they have the same name or are defined by the same type declaration.

**Examples:**

```c
// Name equivalence examples
typedef struct {
    int x, y;
} Point1;

typedef struct {
    int x, y;
} Point2;

// Under name equivalence:
// Point1 ≢ Point2 (different names, different types)
```

**Advantages:**
- Fast: O(1) comparison
- Safe: Prevents accidental type mixing
- Simple implementation

**Disadvantages:**
- Restrictive: May reject valid operations
- Requires explicit type conversions

#### 3. Declaration Equivalence

**Definition:** Two types are declaration equivalent if they refer to the same type declaration or derived through aliases.

**Examples:**

```c
// Declaration equivalence examples
typedef int Integer;
typedef Integer MyInt;

// Under declaration equivalence:
// int ≡ Integer ≡ MyInt (all refer to same base type)
```

### Complex Type Equivalence Examples

#### 1. Array Types

```c
int arr1[10];           // array(int, 10)
int arr2[10];           // array(int, 10)
int arr3[20];           // array(int, 20)

// Structural equivalence: arr1 ≡ arr2, arr1 ≢ arr3
```

#### 2. Function Types

```c
int func1(int x, float y);     // function(int × float → int)
int func2(int a, float b);     // function(int × float → int)
int func3(float x, int y);     // function(float × int → int)

// Structural equivalence: func1 ≡ func2, func1 ≢ func3
// Parameter names don't matter, only types and order
```

---

## b) Polymorphic Functions

### Introduction to Polymorphism

**Polymorphism** allows a single function or operator to work with multiple types. It enables code reuse and generic programming while maintaining type safety.

### Types of Polymorphism

#### 1. Parametric Polymorphism (Generics)

**Definition:** Functions that work uniformly over a range of types, with type parameters that can be instantiated with specific types.

**Examples:**

##### Generic Function in C++ Templates
```cpp
// Generic swap function
template<typename T>
void swap(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}

// Usage
int x = 10, y = 20;
swap<int>(x, y);        // T = int

string s1 = "hello", s2 = "world";
swap<string>(s1, s2);   // T = string
```

##### Generic Function in Haskell
```haskell
-- Generic identity function
identity :: a -> a
identity x = x

-- Generic list length function
length :: [a] -> Int
length [] = 0
length (x:xs) = 1 + length xs

-- Usage
identity 42          -- a = Int
identity "hello"     -- a = String
```

##### Implementation in Compiler

```c
// Type parameter representation
typedef struct TypeParameter {
    char* name;              // Parameter name (e.g., "T")
    int parameter_id;        // Unique identifier
} TypeParameter;

// Generic function representation
typedef struct GenericFunction {
    char* name;
    TypeParameter* type_params;
    int param_count;
    FunctionType* generic_type;
} GenericFunction;
```

#### 2. Ad-hoc Polymorphism (Overloading)

**Definition:** Different implementations of a function for different types, selected based on the types of arguments.

**Examples:**

##### Function Overloading in C++
```cpp
// Overloaded print function
void print(int x) {
    cout << "Integer: " << x << endl;
}

void print(float x) {
    cout << "Float: " << x << endl;
}

void print(string x) {
    cout << "String: " << x << endl;
}

// Usage - compiler selects appropriate version
print(42);        // Calls print(int)
print(3.14f);     // Calls print(float)
print("hello");   // Calls print(string)
```

##### Operator Overloading
```cpp
class Complex {
    double real, imag;
public:
    // Overloaded + operator
    Complex operator+(const Complex& other) {
        return Complex(real + other.real, imag + other.imag);
    }
};

// Usage
Complex a(1, 2);
Complex b(3, 4);
Complex c = a + b;    // Uses overloaded +
```

#### 3. Subtype Polymorphism (Inheritance)

**Definition:** Functions that work with a base type can also work with any of its subtypes.

**Examples:**

##### Virtual Functions in C++
```cpp
class Shape {
public:
    virtual double area() = 0;      // Pure virtual function
    virtual void draw() = 0;
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    
    double area() override {
        return 3.14159 * radius * radius;
    }
    
    void draw() override {
        cout << "Drawing circle with radius " << radius << endl;
    }
};

class Rectangle : public Shape {
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    
    double area() override {
        return width * height;
    }
    
    void draw() override {
        cout << "Drawing rectangle " << width << "x" << height << endl;
    }
};

// Polymorphic function
void process_shape(Shape* shape) {
    cout << "Area: " << shape->area() << endl;  // Virtual dispatch
    shape->draw();                              // Virtual dispatch
}

// Usage
Circle circle(5.0);
Rectangle rect(4.0, 6.0);

process_shape(&circle);    // Works with Circle
process_shape(&rect);      // Works with Rectangle
```

### Advanced Polymorphic Concepts

#### 1. Bounded Polymorphism (Type Constraints)

**Examples:**

##### Java Bounded Generics
```java
// Bounded type parameter
public class NumberProcessor<T extends Number> {
    public double process(T value) {
        return value.doubleValue() * 2;  // Can call Number methods
    }
}

// Usage
NumberProcessor<Integer> intProcessor = new NumberProcessor<>();
NumberProcessor<Double> doubleProcessor = new NumberProcessor<>();
```

##### Haskell Type Classes
```haskell
-- Type class constraint
sum_list :: (Num a) => [a] -> a
sum_list [] = 0
sum_list (x:xs) = x + sum_list xs

-- Multiple constraints
compare_and_show :: (Ord a, Show a) => a -> a -> String
compare_and_show x y 
    | x < y = show x ++ " is less than " ++ show y
    | x > y = show x ++ " is greater than " ++ show y
    | otherwise = show x ++ " equals " ++ show y
```

#### 2. Higher-Order Polymorphism

**Examples:**

##### Higher-Order Functions in Haskell
```haskell
-- map function: applies function to each element
map :: (a -> b) -> [a] -> [b]
map f [] = []
map f (x:xs) = f x : map f xs

-- filter function: selects elements matching predicate
filter :: (a -> Bool) -> [a] -> [a]
filter p [] = []
filter p (x:xs) 
    | p x = x : filter p xs
    | otherwise = filter p xs

-- Usage
numbers = [1, 2, 3, 4, 5]
doubled = map (*2) numbers        -- [2, 4, 6, 8, 10]
evens = filter even numbers       -- [2, 4]
```

#### 3. Polymorphic Data Structures

**Examples:**

##### Generic List in C++
```cpp
template<typename T>
class List {
private:
    struct Node {
        T data;
        Node* next;
        Node(const T& value) : data(value), next(nullptr) {}
    };
    
    Node* head;
    
public:
    List() : head(nullptr) {}
    
    void push_front(const T& value) {
        Node* new_node = new Node(value);
        new_node->next = head;
        head = new_node;
    }
    
    T& front() {
        return head->data;
    }
};

// Usage
List<int> int_list;
List<string> string_list;
```

##### Algebraic Data Types in Haskell
```haskell
-- Generic Maybe type
data Maybe a = Nothing | Just a

-- Generic Tree type
data Tree a = Empty | Node a (Tree a) (Tree a)

-- Functions working with generic types
map_maybe :: (a -> b) -> Maybe a -> Maybe b
map_maybe f Nothing = Nothing
map_maybe f (Just x) = Just (f x)
```

---

## Summary

### Type Equivalence

| Type | Basis | Advantages | Disadvantages | Use Cases |
|------|-------|------------|---------------|-----------|
| **Structural** | Type structure | Flexible, intuitive | Expensive comparison | ML, Haskell |
| **Name** | Type names | Fast, safe | Restrictive | Pascal, Ada |
| **Declaration** | Type declarations | Balanced approach | Moderate complexity | C, C++ |

### Polymorphism Types

| Type | Mechanism | Benefits | Examples |
|------|-----------|----------|----------|
| **Parametric** | Type parameters | Code reuse, type safety | Templates, generics |
| **Ad-hoc** | Overloading | Type-specific optimization | Function/operator overloading |
| **Subtype** | Inheritance | Runtime flexibility | Virtual functions |

### Key Concepts

1. **Type Equivalence** determines when types can be used interchangeably
2. **Structural Equivalence** focuses on type structure
3. **Name Equivalence** focuses on type names and declarations
4. **Parametric Polymorphism** enables generic programming
5. **Ad-hoc Polymorphism** provides type-specific implementations
6. **Subtype Polymorphism** supports inheritance hierarchies

### Implementation Considerations

1. **Type Checking:** Efficient algorithms for type equivalence
2. **Type Inference:** Automatic type deduction for polymorphic functions
3. **Code Generation:** Handling generic instantiation and virtual dispatch
4. **Error Messages:** Clear reporting of type mismatches
5. **Performance:** Balancing flexibility with runtime efficiency

Type systems and polymorphism are fundamental to modern programming languages, enabling both type safety and code reusability while providing the flexibility needed for complex software development. 