# Symbol Table Management System

A **symbol table** is a fundamental data structure used by a compiler to store and retrieve information about the identifiers (variables, functions, classes, etc.) encountered in a source program. It is a critical component that acts as a database for the compiler, accessed and updated by most of its phases. The **symbol table management system** refers to the implementation of this data structure, the operations it supports, and how it handles language features like scope.

---

## 1. Role of the Symbol Table

The symbol table's primary role is to assist in the analysis and translation of the source code by keeping track of declared names.

*   **During Semantic Analysis:** The symbol table is used to verify that identifiers are used correctly. This includes:
    *   **Declaration Checks:** Ensuring every identifier is declared before it is used.
    *   **Scope Checks:** Verifying that an identifier is accessible in the part of the code where it is used.
    *   **Type Checks:** Storing the type of each identifier to check for type errors in expressions and assignments (e.g., preventing the addition of a string to an integer).
*   **During Code Generation:** The symbol table provides the information needed to generate target code, such as the memory address of a variable or the entry point of a function.

## 2. Information Stored in a Symbol Table

For each identifier, the symbol table stores a collection of **attributes** or properties. The exact attributes depend on the identifier and the language, but common entries include:

*   **Name:** The identifier's name as a string (e.g., "myVariable").
*   **Type:** The data type of the identifier (e.g., `int`, `float`, `char*`, or a pointer to another table entry for a `struct`).
*   **Scope:** The region of the program where the name is valid (e.g., the name of the function it belongs to, or "global").
*   **Memory Location:** The runtime address (e.g., an offset in an activation record or a static memory address) assigned to the variable.
*   **For Functions:**
    *   Number and types of parameters.
    *   Return type.
    *   Address of the first instruction of the function body.
*   **Other Attributes:** May include storage class (`static`, `extern`), access modifiers (`public`, `private`), or whether a variable is a constant.

---

## 3. Symbol Table Operations

A symbol table management system must provide efficient methods to perform several key operations:

1.  **`insert(name, attributes)`:** This operation adds a new identifier to the table along with its associated attributes. It is typically called when the parser encounters a declaration. The operation must first check if the name already exists in the *current scope* to prevent re-declarations.
2.  **`lookup(name)`:** This is the most frequent operation. It searches the table for an identifier to retrieve its attributes. The lookup must respect the language's scoping rules. For example, in a block-structured language, it first searches the current, most-nested scope. If the name is not found, it searches the parent scope, and so on, up to the global scope. If not found anywhere, it results in an "undeclared identifier" error.
3.  **`set_attribute(name, attribute, value)`:** Modifies an existing attribute for a given name. For example, the code generator might use this to update the memory location attribute after it has been determined.
4.  **`enter_scope()` and `exit_scope()`:** These operations are essential for handling block-structured languages. `enter_scope` signals that a new scope has begun (e.g., on entering a function body or a `{...}` block), and `exit_scope` signals that the current scope has ended, making all identifiers declared within it inaccessible.

---

## 4. Data Structures for Symbol Tables

The choice of data structure is crucial for the compiler's performance, as table operations are very frequent.

### a) Linear List (Unordered)
An array or linked list of records.
*   **Implementation:** A simple array or list where each element is a record containing the identifier's name and attributes.
*   **Pros:** Very simple to implement.
*   **Cons:** Extremely inefficient. `lookup` requires a linear search, taking O(n) time. This is too slow for any practical compiler.

### b) Sorted List
The list is kept sorted alphabetically by identifier name.
*   **Pros:** `lookup` can be performed efficiently using a binary search, taking O(log n) time.
*   **Cons:** `insert` is slow, taking O(n) time because elements may need to be shifted to maintain the sorted order.

### c) Hash Table
This is the most common and effective data structure for symbol tables.
*   **Implementation:** An array of buckets. A hash function maps the identifier name to an index in the array. Collisions (when two names hash to the same index) are handled by creating a linked list of records at that index.
*   **Pros:** Extremely fast on average. `insert` and `lookup` operations take, on average, O(1) constant time.
*   **Cons:** Can have worst-case O(n) performance if the hash function is poor and causes many collisions. Requires more complex implementation.

### d) Handling Scopes

For block-structured languages, the symbol table must manage scopes correctly. This is typically done in one of two ways, often in combination with a hash table:

1.  **A Separate Hash Table for Each Scope:** A stack of hash tables is maintained. `enter_scope` pushes a new, empty hash table onto the stack. `exit_scope` pops the top table. A `lookup` operation searches the tables from the top of the stack down to the bottom. This is conceptually clean but can be inefficient if scopes are numerous.
2.  **A Single Global Hash Table:** Each entry in the hash table's linked lists contains not only the name and attributes but also a scope level or scope ID. When inserting, the current scope level is added. When looking up a name, the list at the corresponding hash index is scanned for an entry matching the name and the highest scope level less than or equal to the current level. When exiting a scope, all entries from that scope must be found and removed, which can be inefficient. 