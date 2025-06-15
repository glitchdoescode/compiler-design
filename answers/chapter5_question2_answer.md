# Symbol Table Management System

This document explains the symbol table management system in a compiler, covering its structure, operations, implementation techniques, and role in compilation.

---

## Introduction to Symbol Tables

### Definition

A **Symbol Table** is a data structure used by compilers to store information about identifiers (symbols) encountered in the source program. It serves as a central repository for all symbol-related information needed during compilation.

### Purpose

The symbol table maintains essential information about:
- **Variables:** Names, types, scope, storage location
- **Functions:** Names, return types, parameters, addresses
- **Constants:** Values, types
- **User-defined types:** Structures, classes, enums
- **Labels:** Jump targets in control flow

---

## Symbol Table Structure and Organization

### Basic Symbol Table Entry

```c
typedef struct SymbolEntry {
    char* name;              // Identifier name
    SymbolType type;         // Data type
    SymbolKind kind;         // Variable, function, constant, etc.
    Scope scope;             // Scope information
    StorageClass storage;    // Storage class (auto, static, extern)
    union {
        struct {
            int offset;      // Memory offset
            int size;        // Size in bytes
        } variable;
        struct {
            int param_count; // Number of parameters
            SymbolEntry* params; // Parameter list
            int address;     // Function address
        } function;
        struct {
            Value value;     // Constant value
        } constant;
    } info;
    int line_number;         // Declaration line
    struct SymbolEntry* next; // For chaining
} SymbolEntry;
```

### Symbol Classifications

#### 1. Symbol Types
```c
typedef enum {
    TYPE_INT,
    TYPE_FLOAT,
    TYPE_CHAR,
    TYPE_BOOL,
    TYPE_ARRAY,
    TYPE_POINTER,
    TYPE_STRUCT,
    TYPE_FUNCTION,
    TYPE_VOID
} SymbolType;
```

#### 2. Symbol Kinds
```c
typedef enum {
    KIND_VARIABLE,
    KIND_FUNCTION,
    KIND_PARAMETER,
    KIND_CONSTANT,
    KIND_TYPE,
    KIND_LABEL
} SymbolKind;
```

#### 3. Storage Classes
```c
typedef enum {
    STORAGE_AUTO,
    STORAGE_STATIC,
    STORAGE_EXTERN,
    STORAGE_REGISTER,
    STORAGE_GLOBAL
} StorageClass;
```

---

## Implementation Techniques

### 1. Linear List Implementation

#### Structure
```c
typedef struct LinearSymbolTable {
    SymbolEntry* entries;
    int count;
    int capacity;
} LinearSymbolTable;
```

#### Operations
```c
// Insert symbol
int insert_symbol(LinearSymbolTable* table, SymbolEntry* entry) {
    if (table->count >= table->capacity) {
        table->capacity *= 2;
        table->entries = realloc(table->entries, 
                               table->capacity * sizeof(SymbolEntry));
    }
    
    // Check for duplicate in current scope
    for (int i = 0; i < table->count; i++) {
        if (strcmp(table->entries[i].name, entry->name) == 0 &&
            table->entries[i].scope == entry->scope) {
            return -1; // Duplicate symbol
        }
    }
    
    table->entries[table->count++] = *entry;
    return table->count - 1;
}

// Lookup symbol
SymbolEntry* lookup_symbol(LinearSymbolTable* table, char* name) {
    for (int i = table->count - 1; i >= 0; i--) {
        if (strcmp(table->entries[i].name, name) == 0) {
            return &table->entries[i];
        }
    }
    return NULL; // Symbol not found
}
```

**Advantages:** Simple implementation, good for small programs
**Disadvantages:** O(n) lookup time, inefficient for large programs

### 2. Hash Table Implementation

#### Structure
```c
#define HASH_TABLE_SIZE 1009

typedef struct HashSymbolTable {
    SymbolEntry* buckets[HASH_TABLE_SIZE];
    int count;
} HashSymbolTable;
```

#### Hash Function
```c
unsigned int hash_function(char* name) {
    unsigned int hash = 0;
    while (*name) {
        hash = hash * 31 + *name;
        name++;
    }
    return hash % HASH_TABLE_SIZE;
}
```

#### Operations
```c
// Insert symbol
int hash_insert_symbol(HashSymbolTable* table, SymbolEntry* entry) {
    unsigned int index = hash_function(entry->name);
    
    // Check for duplicates in the chain
    SymbolEntry* current = table->buckets[index];
    while (current != NULL) {
        if (strcmp(current->name, entry->name) == 0 &&
            current->scope == entry->scope) {
            return -1; // Duplicate symbol
        }
        current = current->next;
    }
    
    // Insert at beginning of chain
    entry->next = table->buckets[index];
    table->buckets[index] = entry;
    table->count++;
    return 0;
}

// Lookup symbol
SymbolEntry* hash_lookup_symbol(HashSymbolTable* table, char* name) {
    unsigned int index = hash_function(name);
    SymbolEntry* current = table->buckets[index];
    
    while (current != NULL) {
        if (strcmp(current->name, name) == 0) {
            return current;
        }
        current = current->next;
    }
    return NULL;
}
```

**Advantages:** O(1) average lookup time, efficient for large programs
**Disadvantages:** More complex implementation, potential hash collisions

### 3. Binary Search Tree Implementation

#### Structure
```c
typedef struct BSTNode {
    SymbolEntry* entry;
    struct BSTNode* left;
    struct BSTNode* right;
} BSTNode;

typedef struct BSTSymbolTable {
    BSTNode* root;
    int count;
} BSTSymbolTable;
```

**Advantages:** O(log n) lookup time, ordered traversal possible
**Disadvantages:** Can become unbalanced, complex scope handling

---

## Scope Management

### Scope Types

#### 1. Global Scope
```c
int global_var;        // Global scope
void global_func();    // Global scope
```

#### 2. Function Scope
```c
void function() {
    int local_var;     // Function scope
}
```

#### 3. Block Scope
```c
void function() {
    int x = 10;
    {
        int y = 20;    // Block scope
        // y is visible here
    }
    // y is not visible here
}
```

### Scope Stack Implementation

```c
typedef struct ScopeStack {
    int* scope_levels;
    int top;
    int capacity;
    int current_scope;
} ScopeStack;

// Enter new scope
void enter_scope(ScopeStack* stack) {
    if (stack->top >= stack->capacity - 1) {
        stack->capacity *= 2;
        stack->scope_levels = realloc(stack->scope_levels,
                                    stack->capacity * sizeof(int));
    }
    stack->scope_levels[++stack->top] = stack->current_scope++;
}

// Exit current scope
void exit_scope(ScopeStack* stack, HashSymbolTable* table) {
    if (stack->top < 0) return;
    
    int scope_to_remove = stack->scope_levels[stack->top--];
    
    // Remove all symbols from this scope
    for (int i = 0; i < HASH_TABLE_SIZE; i++) {
        SymbolEntry** current = &table->buckets[i];
        while (*current != NULL) {
            if ((*current)->scope == scope_to_remove) {
                SymbolEntry* to_remove = *current;
                *current = (*current)->next;
                free(to_remove);
                table->count--;
            } else {
                current = &(*current)->next;
            }
        }
    }
}
```

### Nested Scope Lookup

```c
SymbolEntry* nested_lookup(HashSymbolTable* table, char* name, 
                          ScopeStack* stack) {
    // Search from current scope to global scope
    for (int i = stack->top; i >= 0; i--) {
        int scope = stack->scope_levels[i];
        unsigned int index = hash_function(name);
        SymbolEntry* current = table->buckets[index];
        
        while (current != NULL) {
            if (strcmp(current->name, name) == 0 && 
                current->scope == scope) {
                return current;
            }
            current = current->next;
        }
    }
    return NULL; // Symbol not found in any scope
}
```

---

## Symbol Table Operations During Compilation

### 1. Declaration Processing

```c
void process_declaration(char* name, SymbolType type, 
                        HashSymbolTable* table, ScopeStack* stack) {
    SymbolEntry* entry = malloc(sizeof(SymbolEntry));
    entry->name = strdup(name);
    entry->type = type;
    entry->kind = KIND_VARIABLE;
    entry->scope = stack->scope_levels[stack->top];
    entry->storage = STORAGE_AUTO;
    entry->line_number = current_line;
    
    if (hash_insert_symbol(table, entry) == -1) {
        printf("Error: Redeclaration of '%s' at line %d\n", 
               name, current_line);
        free(entry);
    }
}
```

### 2. Type Checking

```c
bool type_check_assignment(SymbolEntry* lhs, SymbolEntry* rhs) {
    if (lhs->type == rhs->type) {
        return true;
    }
    
    // Handle implicit conversions
    if ((lhs->type == TYPE_FLOAT && rhs->type == TYPE_INT) ||
        (lhs->type == TYPE_INT && rhs->type == TYPE_CHAR)) {
        return true;
    }
    
    return false;
}
```

### 3. Function Call Validation

```c
bool validate_function_call(SymbolEntry* func, SymbolEntry** args, 
                           int arg_count) {
    if (func->kind != KIND_FUNCTION) {
        return false;
    }
    
    if (func->info.function.param_count != arg_count) {
        return false;
    }
    
    // Check parameter types
    for (int i = 0; i < arg_count; i++) {
        if (!type_check_assignment(&func->info.function.params[i], 
                                  args[i])) {
            return false;
        }
    }
    
    return true;
}
```

---

## Symbol Table in Different Compilation Phases

### 1. Lexical Analysis Phase

```c
// During tokenization
Token* create_identifier_token(char* lexeme) {
    Token* token = malloc(sizeof(Token));
    token->type = TOKEN_IDENTIFIER;
    token->lexeme = strdup(lexeme);
    
    // Check if it's a keyword
    if (is_keyword(lexeme)) {
        token->type = get_keyword_token_type(lexeme);
    } else {
        // Add to symbol table if not present
        SymbolEntry* entry = hash_lookup_symbol(global_symbol_table, lexeme);
        if (entry == NULL) {
            entry = create_placeholder_entry(lexeme);
            hash_insert_symbol(global_symbol_table, entry);
        }
        token->symbol_entry = entry;
    }
    
    return token;
}
```

### 2. Syntax Analysis Phase

```c
// During parsing
void parse_declaration() {
    SymbolType type = parse_type();
    char* name = parse_identifier();
    
    // Create symbol table entry
    SymbolEntry* entry = malloc(sizeof(SymbolEntry));
    entry->name = strdup(name);
    entry->type = type;
    entry->kind = KIND_VARIABLE;
    entry->scope = current_scope;
    
    // Calculate storage information
    entry->info.variable.size = get_type_size(type);
    entry->info.variable.offset = allocate_storage(entry->info.variable.size);
    
    if (hash_insert_symbol(symbol_table, entry) == -1) {
        syntax_error("Redeclaration of variable");
    }
}
```

### 3. Semantic Analysis Phase

```c
// Type checking and semantic validation
void semantic_analysis() {
    for (int i = 0; i < HASH_TABLE_SIZE; i++) {
        SymbolEntry* current = symbol_table->buckets[i];
        while (current != NULL) {
            // Check for unused variables
            if (current->kind == KIND_VARIABLE && !current->is_used) {
                warning("Variable '%s' declared but never used", 
                       current->name);
            }
            
            // Check for uninitialized variables
            if (current->kind == KIND_VARIABLE && !current->is_initialized) {
                warning("Variable '%s' used without initialization", 
                       current->name);
            }
            
            current = current->next;
        }
    }
}
```

### 4. Code Generation Phase

```c
// Generate code using symbol table information
void generate_variable_access(SymbolEntry* entry) {
    switch (entry->storage) {
        case STORAGE_GLOBAL:
            emit("LOAD R1, %s", entry->name);
            break;
        case STORAGE_AUTO:
            emit("LOAD R1, %d(BP)", entry->info.variable.offset);
            break;
        case STORAGE_STATIC:
            emit("LOAD R1, static_%s", entry->name);
            break;
        case STORAGE_REGISTER:
            emit("MOV R1, %s", entry->info.variable.register_name);
            break;
    }
}
```

---

## Advanced Features

### 1. Cross-Reference Information

```c
typedef struct Reference {
    int line_number;
    int column;
    ReferenceType type; // DECLARATION, DEFINITION, USE
    struct Reference* next;
} Reference;

typedef struct CrossRefEntry {
    char* symbol_name;
    Reference* references;
    struct CrossRefEntry* next;
} CrossRefEntry;

// Add reference
void add_reference(CrossRefEntry* entry, int line, int col, 
                  ReferenceType type) {
    Reference* ref = malloc(sizeof(Reference));
    ref->line_number = line;
    ref->column = col;
    ref->type = type;
    ref->next = entry->references;
    entry->references = ref;
}
```

### 2. Error Handling

```c
typedef enum {
    ERROR_REDECLARATION,
    ERROR_UNDECLARED,
    ERROR_TYPE_MISMATCH,
    ERROR_SCOPE_VIOLATION
} SymbolError;

void report_symbol_error(SymbolError error, char* symbol, int line) {
    switch (error) {
        case ERROR_REDECLARATION:
            printf("Error at line %d: Redeclaration of '%s'\n", line, symbol);
            break;
        case ERROR_UNDECLARED:
            printf("Error at line %d: Undeclared identifier '%s'\n", line, symbol);
            break;
        case ERROR_TYPE_MISMATCH:
            printf("Error at line %d: Type mismatch for '%s'\n", line, symbol);
            break;
    }
}
```

### 3. Symbol Table Debugging

```c
// Print symbol table contents
void print_symbol_table(HashSymbolTable* table) {
    printf("Symbol Table Contents:\n");
    printf("%-20s %-10s %-10s %-10s %-10s\n", 
           "Name", "Type", "Kind", "Scope", "Storage");
    printf("--------------------------------------------------------\n");
    
    for (int i = 0; i < HASH_TABLE_SIZE; i++) {
        SymbolEntry* current = table->buckets[i];
        while (current != NULL) {
            printf("%-20s %-10s %-10s %-10d %-10s\n",
                   current->name,
                   type_to_string(current->type),
                   kind_to_string(current->kind),
                   current->scope,
                   storage_to_string(current->storage));
            current = current->next;
        }
    }
}
```

---

## Performance Optimization

### 1. Memory Management

```c
// Memory pool for symbol entries
typedef struct SymbolPool {
    SymbolEntry* pool;
    int* free_list;
    int pool_size;
    int free_count;
} SymbolPool;

SymbolEntry* allocate_symbol(SymbolPool* pool) {
    if (pool->free_count == 0) {
        return NULL; // Pool exhausted
    }
    
    int index = pool->free_list[--pool->free_count];
    return &pool->pool[index];
}
```

### 2. Hash Table Optimization

```c
// Dynamic hash table resizing
void resize_hash_table(HashSymbolTable* table) {
    int old_size = HASH_TABLE_SIZE;
    SymbolEntry** old_buckets = table->buckets;
    
    // Double the size
    table->buckets = calloc(old_size * 2, sizeof(SymbolEntry*));
    table->count = 0;
    
    // Rehash all entries
    for (int i = 0; i < old_size; i++) {
        SymbolEntry* current = old_buckets[i];
        while (current != NULL) {
            SymbolEntry* next = current->next;
            current->next = NULL;
            hash_insert_symbol(table, current);
            current = next;
        }
    }
    
    free(old_buckets);
}
```

---

## Summary

### Key Components of Symbol Table Management

1. **Data Structure:** Hash table for O(1) average lookup
2. **Scope Management:** Stack-based scope tracking
3. **Type Information:** Complete type system support
4. **Storage Allocation:** Memory layout and addressing
5. **Error Handling:** Comprehensive error detection
6. **Cross-References:** Usage tracking and analysis

### Symbol Table Lifecycle

1. **Creation:** Initialize empty symbol table
2. **Population:** Add symbols during parsing
3. **Validation:** Type checking and semantic analysis
4. **Utilization:** Code generation and optimization
5. **Cleanup:** Memory deallocation

### Best Practices

1. **Use hash tables** for large programs
2. **Implement proper scope management**
3. **Maintain comprehensive type information**
4. **Provide good error messages**
5. **Optimize for common operations**
6. **Support debugging and analysis tools**

### Implementation Comparison

| Method | Lookup Time | Memory | Complexity | Best For |
|--------|-------------|---------|------------|----------|
| Linear List | O(n) | Low | Simple | Small programs |
| Hash Table | O(1) avg | Medium | Moderate | Large programs |
| BST | O(log n) | Medium | Complex | Ordered access |

The symbol table is a critical component that enables compilers to maintain semantic consistency, perform type checking, generate efficient code, and provide meaningful error messages throughout the compilation process. 