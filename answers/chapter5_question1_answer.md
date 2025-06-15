# Storage Allocation Strategies

This document provides a comprehensive explanation of various storage allocation strategies used in runtime environments, their characteristics, and trade-offs.

---

## a) Various Storage Allocation Strategies

### Overview

**Storage allocation** refers to how memory is organized and managed during program execution. The choice of allocation strategy affects program performance, memory usage, and implementation complexity.

### 1. Static Storage Allocation

#### Definition

**Static allocation** assigns fixed memory locations to variables at compile time. Memory addresses are determined before program execution and remain constant throughout the program's lifetime.

#### Characteristics

- **Compile-time allocation:** Memory addresses determined during compilation
- **Fixed size:** Variable sizes must be known at compile time
- **Global scope:** Typically used for global variables and constants
- **No runtime overhead:** No allocation/deallocation during execution
- **Simple implementation:** Straightforward memory management

#### Memory Layout

```
High Memory
+------------------+
| Program Code     |
+------------------+
| Static Variables |  ← Fixed addresses
| - Global vars    |
| - Constants      |
| - String literals|
+------------------+
Low Memory
```

#### Examples

```c
// Static allocation examples
int global_var = 100;           // Global variable
static int file_scope_var;      // File-scope static
const char* message = "Hello";  // String literal

void function() {
    static int counter = 0;     // Function-scope static
    counter++;                  // Address never changes
}
```

#### Advantages

1. **Fast access:** Direct memory addressing, no indirection
2. **No fragmentation:** Memory layout is fixed
3. **Simple implementation:** No complex memory management
4. **Predictable:** Memory usage known at compile time
5. **No runtime overhead:** No allocation/deallocation costs

#### Disadvantages

1. **Inflexible:** Cannot handle dynamic data structures
2. **Memory waste:** All declared memory is allocated regardless of usage
3. **No recursion support:** Cannot handle recursive function calls efficiently
4. **Limited scalability:** Cannot adapt to varying input sizes
5. **Early binding:** All decisions made at compile time

#### Use Cases

- **Embedded systems:** Where memory is limited and predictable
- **System programming:** Device drivers, kernel code
- **Constants and literals:** Read-only data
- **Global configuration:** Application-wide settings

### 2. Stack Storage Allocation

#### Definition

**Stack allocation** uses a Last-In-First-Out (LIFO) data structure to manage memory for function calls, local variables, and temporary data. Memory is allocated and deallocated automatically as functions are called and return.

#### Characteristics

- **LIFO ordering:** Last allocated is first deallocated
- **Automatic management:** Allocation/deallocation handled by runtime
- **Function-based:** Each function call gets a stack frame
- **Fast allocation:** Simple pointer arithmetic
- **Limited lifetime:** Variables exist only during function execution

#### Stack Frame Structure

```
High Address
+------------------+
| Previous Frame   |
+------------------+
| Parameters       | ← Passed by caller
+------------------+
| Return Address   | ← Where to return after function
+------------------+
| Saved Registers  | ← Caller's register values
+------------------+
| Local Variables  | ← Function's local data
+------------------+
| Temporaries      | ← Intermediate computations
+------------------+
| Dynamic Link     | ← Pointer to previous frame
+------------------+ ← Current Stack Pointer (SP)
| Next Frame       |
+------------------+
Low Address
```

#### Stack Operations

```c
// Function call sequence
void caller() {
    int x = 10;           // Local variable in caller's frame
    int result = func(x); // Call function
    // Stack frame for func() is created and destroyed
}

int func(int param) {
    int local = param * 2; // Local variable in func's frame
    return local;          // Frame destroyed on return
}
```

#### Stack Pointer Management

```assembly
; Function call
PUSH param        ; Push parameters
CALL func         ; Push return address and jump
; Function prologue
PUSH BP           ; Save old frame pointer
MOV BP, SP        ; Set new frame pointer
SUB SP, locals    ; Allocate space for locals
; Function body
; ...
; Function epilogue
MOV SP, BP        ; Deallocate locals
POP BP            ; Restore old frame pointer
RET               ; Return to caller
```

#### Advantages

1. **Automatic management:** No explicit allocation/deallocation needed
2. **Fast allocation:** Simple pointer increment/decrement
3. **Good locality:** Recently allocated data is cache-friendly
4. **Supports recursion:** Natural handling of recursive calls
5. **No fragmentation:** Contiguous allocation and deallocation
6. **Efficient:** Minimal overhead for allocation/deallocation

#### Disadvantages

1. **Limited size:** Stack size is typically limited (few MBs)
2. **LIFO constraint:** Cannot deallocate arbitrary blocks
3. **Scope limitation:** Variables only exist during function execution
4. **Stack overflow:** Risk of exceeding stack limits
5. **No sharing:** Data cannot be shared between function calls
6. **Fixed lifetime:** Cannot extend variable lifetime beyond function

#### Use Cases

- **Local variables:** Function-scope variables
- **Function parameters:** Argument passing
- **Return addresses:** Function call management
- **Temporary values:** Intermediate computations
- **Expression evaluation:** Operator stack in calculators

### 3. Heap Storage Allocation

#### Definition

**Heap allocation** provides dynamic memory management where memory blocks can be allocated and deallocated in any order during program execution. It supports variable-sized data structures and flexible memory usage.

#### Characteristics

- **Dynamic allocation:** Memory requested at runtime
- **Arbitrary order:** Allocation and deallocation in any sequence
- **Variable size:** Can handle data structures of unknown size
- **Explicit management:** Programmer controls allocation/deallocation
- **Flexible lifetime:** Objects can outlive their creating function

#### Heap Structure

```
High Address
+------------------+
| Free Block       |
+------------------+
| Allocated Block  | ← User data
+------------------+
| Free Block       |
+------------------+
| Allocated Block  | ← User data
+------------------+
| Free Block       |
+------------------+
Low Address
```

#### Memory Block Structure

```
+------------------+
| Block Header     | ← Size, status, links
+------------------+
| User Data        | ← Actual allocated memory
| ...              |
+------------------+
| Block Footer     | ← Optional, for coalescing
+------------------+
```

#### Allocation Algorithms

##### 1. First Fit
```c
// Find first block large enough
Block* first_fit(size_t size) {
    Block* current = free_list;
    while (current != NULL) {
        if (current->size >= size) {
            return current;  // First suitable block
        }
        current = current->next;
    }
    return NULL;  // No suitable block found
}
```

##### 2. Best Fit
```c
// Find smallest block that fits
Block* best_fit(size_t size) {
    Block* best = NULL;
    Block* current = free_list;
    size_t best_size = SIZE_MAX;
    
    while (current != NULL) {
        if (current->size >= size && current->size < best_size) {
            best = current;
            best_size = current->size;
        }
        current = current->next;
    }
    return best;
}
```

##### 3. Worst Fit
```c
// Find largest available block
Block* worst_fit(size_t size) {
    Block* worst = NULL;
    Block* current = free_list;
    size_t worst_size = 0;
    
    while (current != NULL) {
        if (current->size >= size && current->size > worst_size) {
            worst = current;
            worst_size = current->size;
        }
        current = current->next;
    }
    return worst;
}
```

#### Fragmentation Issues

##### External Fragmentation
```
Before allocation:
[Free 100] [Used 50] [Free 80] [Used 30] [Free 120]

After requesting 90 bytes:
[Free 100] [Used 50] [Free 80] [Used 30] [Free 120]
                                         ↑
                              Only this block fits
Total free: 300 bytes, but largest contiguous: 120 bytes
```

##### Internal Fragmentation
```
Request: 60 bytes
Available block: 64 bytes (minimum allocation unit)
Result: 4 bytes wasted internally
```

#### Garbage Collection

##### Reference Counting
```c
typedef struct Object {
    int ref_count;
    // ... object data
} Object;

void retain(Object* obj) {
    obj->ref_count++;
}

void release(Object* obj) {
    obj->ref_count--;
    if (obj->ref_count == 0) {
        free(obj);  // Deallocate when no references
    }
}
```

##### Mark and Sweep
```c
// Mark phase: mark all reachable objects
void mark_phase() {
    for (Object* root : roots) {
        mark_reachable(root);
    }
}

void mark_reachable(Object* obj) {
    if (obj && !obj->marked) {
        obj->marked = true;
        for (Object* child : obj->references) {
            mark_reachable(child);
        }
    }
}

// Sweep phase: deallocate unmarked objects
void sweep_phase() {
    for (Object* obj : all_objects) {
        if (!obj->marked) {
            free(obj);
        } else {
            obj->marked = false;  // Reset for next cycle
        }
    }
}
```

#### Advantages

1. **Flexibility:** Can allocate any size at any time
2. **Dynamic sizing:** Handles unknown data sizes
3. **Sharing:** Objects can be shared between functions
4. **Persistence:** Objects can outlive creating function
5. **Efficient for large objects:** Better than stack for big data
6. **Supports complex data structures:** Trees, graphs, etc.

#### Disadvantages

1. **Fragmentation:** Both internal and external fragmentation
2. **Slower allocation:** Complex algorithms needed
3. **Memory leaks:** Risk of forgetting to deallocate
4. **Overhead:** Extra memory for bookkeeping
5. **Cache performance:** Poor locality of reference
6. **Complexity:** Requires sophisticated management

#### Use Cases

- **Dynamic data structures:** Linked lists, trees, graphs
- **Variable-sized arrays:** Arrays with runtime-determined size
- **Object-oriented programming:** Dynamic object creation
- **String manipulation:** Variable-length strings
- **Database systems:** Variable-sized records
- **Graphics programming:** Dynamic image buffers

---

## b) Stack vs Heap Allocation: Differences, Merits, and Demerits

### Detailed Comparison

| Aspect | Stack Allocation | Heap Allocation |
|--------|------------------|-----------------|
| **Speed** | Very fast (pointer arithmetic) | Slower (search algorithms) |
| **Management** | Automatic | Manual (or garbage collected) |
| **Size Limit** | Limited (typically 1-8 MB) | Limited by available memory |
| **Fragmentation** | None | Both internal and external |
| **Lifetime** | Function scope | Arbitrary |
| **Access Pattern** | LIFO only | Random access |
| **Thread Safety** | Each thread has own stack | Shared, needs synchronization |
| **Debugging** | Easier (automatic cleanup) | Harder (memory leaks possible) |

### Stack Allocation - Detailed Analysis

#### Merits

1. **Performance Benefits:**
   ```c
   void fast_function() {
       int array[1000];  // Allocated in ~1 CPU cycle
       // Use array...
   }  // Automatically deallocated
   ```

2. **Memory Safety:**
   - No memory leaks possible
   - Automatic cleanup on function return
   - No dangling pointers within scope

3. **Cache Efficiency:**
   ```c
   void cache_friendly() {
       int a, b, c, d;  // Allocated contiguously
       // Good spatial locality
   }
   ```

4. **Simplicity:**
   - No explicit memory management
   - Compiler handles everything
   - Predictable behavior

#### Demerits

1. **Size Limitations:**
   ```c
   void problematic() {
       int huge_array[1000000];  // May cause stack overflow
   }
   ```

2. **Scope Restrictions:**
   ```c
   int* dangerous_function() {
       int local = 42;
       return &local;  // Dangling pointer! local is destroyed
   }
   ```

3. **No Dynamic Sizing:**
   ```c
   void limited(int n) {
       int array[n];  // VLA, but size must be known at call time
   }
   ```

### Heap Allocation - Detailed Analysis

#### Merits

1. **Flexibility:**
   ```c
   int* create_array(int size) {
       return malloc(size * sizeof(int));  // Any size, any time
   }
   ```

2. **Persistence:**
   ```c
   typedef struct Node {
       int data;
       struct Node* next;
   } Node;
   
   Node* create_list() {
       Node* head = malloc(sizeof(Node));
       // List persists after function returns
       return head;
   }
   ```

3. **Large Objects:**
   ```c
   void handle_big_data() {
       char* buffer = malloc(100 * 1024 * 1024);  // 100MB
       // Process large dataset
       free(buffer);
   }
   ```

4. **Sharing:**
   ```c
   void share_data() {
       int* shared = malloc(sizeof(int) * 100);
       pass_to_function1(shared);
       pass_to_function2(shared);  // Same data shared
       free(shared);
   }
   ```

#### Demerits

1. **Memory Management Complexity:**
   ```c
   void memory_leak_example() {
       char* buffer = malloc(1024);
       if (error_condition) {
           return;  // Memory leak! forgot free()
       }
       free(buffer);
   }
   ```

2. **Performance Overhead:**
   ```c
   // Heap allocation is much slower
   for (int i = 0; i < 1000000; i++) {
       int* p = malloc(sizeof(int));  // Expensive!
       *p = i;
       free(p);  // Also expensive!
   }
   ```

3. **Fragmentation Problems:**
   ```c
   // This pattern causes fragmentation
   void* ptrs[1000];
   for (int i = 0; i < 1000; i++) {
       ptrs[i] = malloc(random_size());
   }
   for (int i = 0; i < 1000; i += 2) {
       free(ptrs[i]);  // Creates holes
   }
   ```

---

## c) Dynamic Storage Allocation

### Definition and Concepts

**Dynamic Storage Allocation** is a memory management technique where memory is allocated and deallocated during program execution based on runtime requirements. It provides flexibility to handle varying memory needs efficiently.

### Key Characteristics

1. **Runtime Decision Making:** Memory allocation decisions made during execution
2. **Variable Sizes:** Can handle objects of unknown or varying sizes
3. **Flexible Lifetime:** Objects can be created and destroyed at any time
4. **Efficient Memory Usage:** Memory allocated only when needed

### Implementation Techniques

#### 1. Free List Management

```c
typedef struct FreeBlock {
    size_t size;
    struct FreeBlock* next;
} FreeBlock;

FreeBlock* free_list = NULL;

void* dynamic_malloc(size_t size) {
    // Add header size
    size_t total_size = size + sizeof(size_t);
    
    // Find suitable free block
    FreeBlock** current = &free_list;
    while (*current != NULL) {
        if ((*current)->size >= total_size) {
            FreeBlock* block = *current;
            *current = block->next;  // Remove from free list
            
            // Split block if too large
            if (block->size > total_size + sizeof(FreeBlock)) {
                FreeBlock* remainder = (FreeBlock*)((char*)block + total_size);
                remainder->size = block->size - total_size;
                remainder->next = *current;
                *current = remainder;
                block->size = total_size;
            }
            
            // Store size in header
            *(size_t*)block = block->size;
            return (char*)block + sizeof(size_t);
        }
        current = &(*current)->next;
    }
    
    // No suitable block found, request from OS
    return request_from_os(total_size);
}

void dynamic_free(void* ptr) {
    if (ptr == NULL) return;
    
    // Get block header
    FreeBlock* block = (FreeBlock*)((char*)ptr - sizeof(size_t));
    
    // Add to free list
    block->next = free_list;
    free_list = block;
    
    // Coalesce adjacent free blocks
    coalesce_free_blocks();
}
```

#### 2. Buddy System

```c
#define MAX_ORDER 10
#define MIN_BLOCK_SIZE 64

typedef struct BuddyBlock {
    int order;
    int is_free;
    struct BuddyBlock* next;
} BuddyBlock;

BuddyBlock* free_lists[MAX_ORDER + 1];

void* buddy_malloc(size_t size) {
    // Find required order
    int order = 0;
    size_t block_size = MIN_BLOCK_SIZE;
    while (block_size < size && order < MAX_ORDER) {
        block_size *= 2;
        order++;
    }
    
    // Find available block
    BuddyBlock* block = find_free_block(order);
    if (block == NULL) {
        return NULL;  // No memory available
    }
    
    // Split larger blocks if necessary
    while (block->order > order) {
        split_block(block);
    }
    
    block->is_free = 0;
    return block;
}

void buddy_free(void* ptr) {
    BuddyBlock* block = (BuddyBlock*)ptr;
    block->is_free = 1;
    
    // Try to merge with buddy
    while (block->order < MAX_ORDER) {
        BuddyBlock* buddy = find_buddy(block);
        if (buddy == NULL || !buddy->is_free || buddy->order != block->order) {
            break;
        }
        
        // Merge with buddy
        block = merge_blocks(block, buddy);
    }
    
    // Add to appropriate free list
    add_to_free_list(block);
}
```

#### 3. Slab Allocation

```c
typedef struct Slab {
    void* memory;
    int object_size;
    int total_objects;
    int free_objects;
    unsigned char* free_bitmap;
    struct Slab* next;
} Slab;

typedef struct SlabCache {
    int object_size;
    Slab* partial_slabs;
    Slab* full_slabs;
    Slab* empty_slabs;
} SlabCache;

void* slab_alloc(SlabCache* cache) {
    Slab* slab = cache->partial_slabs;
    if (slab == NULL) {
        slab = cache->empty_slabs;
        if (slab == NULL) {
            slab = create_new_slab(cache->object_size);
        } else {
            // Move from empty to partial
            cache->empty_slabs = slab->next;
        }
        slab->next = cache->partial_slabs;
        cache->partial_slabs = slab;
    }
    
    // Find free object in slab
    int index = find_free_object(slab);
    set_bit(slab->free_bitmap, index);
    slab->free_objects--;
    
    // Move to full list if necessary
    if (slab->free_objects == 0) {
        remove_from_partial(cache, slab);
        add_to_full(cache, slab);
    }
    
    return (char*)slab->memory + (index * slab->object_size);
}
```

### Advanced Dynamic Allocation Techniques

#### 1. Memory Pools

```c
typedef struct MemoryPool {
    void* memory;
    size_t block_size;
    size_t total_blocks;
    size_t free_blocks;
    void* free_list;
} MemoryPool;

MemoryPool* create_pool(size_t block_size, size_t num_blocks) {
    MemoryPool* pool = malloc(sizeof(MemoryPool));
    pool->memory = malloc(block_size * num_blocks);
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->free_blocks = num_blocks;
    
    // Initialize free list
    char* current = (char*)pool->memory;
    for (size_t i = 0; i < num_blocks - 1; i++) {
        *(void**)current = current + block_size;
        current += block_size;
    }
    *(void**)current = NULL;
    pool->free_list = pool->memory;
    
    return pool;
}

void* pool_alloc(MemoryPool* pool) {
    if (pool->free_list == NULL) {
        return NULL;  // Pool exhausted
    }
    
    void* block = pool->free_list;
    pool->free_list = *(void**)block;
    pool->free_blocks--;
    return block;
}

void pool_free(MemoryPool* pool, void* ptr) {
    *(void**)ptr = pool->free_list;
    pool->free_list = ptr;
    pool->free_blocks++;
}
```

#### 2. Garbage Collection Integration

```c
typedef struct GCObject {
    int marked;
    size_t size;
    struct GCObject* next;
} GCObject;

GCObject* gc_objects = NULL;

void* gc_malloc(size_t size) {
    // Trigger GC if memory pressure is high
    if (should_collect_garbage()) {
        garbage_collect();
    }
    
    GCObject* obj = malloc(sizeof(GCObject) + size);
    obj->marked = 0;
    obj->size = size;
    obj->next = gc_objects;
    gc_objects = obj;
    
    return (char*)obj + sizeof(GCObject);
}

void garbage_collect() {
    // Mark phase
    mark_reachable_objects();
    
    // Sweep phase
    GCObject** current = &gc_objects;
    while (*current != NULL) {
        if ((*current)->marked) {
            (*current)->marked = 0;  // Reset for next cycle
            current = &(*current)->next;
        } else {
            GCObject* to_free = *current;
            *current = (*current)->next;
            free(to_free);
        }
    }
}
```

### Performance Considerations

#### 1. Allocation Speed Comparison

```c
// Benchmark different allocation strategies
void benchmark_allocation() {
    clock_t start, end;
    const int iterations = 1000000;
    
    // Stack allocation (fastest)
    start = clock();
    for (int i = 0; i < iterations; i++) {
        int local_var = i;  // Stack allocation
    }
    end = clock();
    printf("Stack: %f seconds\n", (double)(end - start) / CLOCKS_PER_SEC);
    
    // Heap allocation (slower)
    start = clock();
    for (int i = 0; i < iterations; i++) {
        int* ptr = malloc(sizeof(int));
        *ptr = i;
        free(ptr);
    }
    end = clock();
    printf("Heap: %f seconds\n", (double)(end - start) / CLOCKS_PER_SEC);
    
    // Pool allocation (middle ground)
    MemoryPool* pool = create_pool(sizeof(int), iterations);
    start = clock();
    for (int i = 0; i < iterations; i++) {
        int* ptr = pool_alloc(pool);
        *ptr = i;
        pool_free(pool, ptr);
    }
    end = clock();
    printf("Pool: %f seconds\n", (double)(end - start) / CLOCKS_PER_SEC);
}
```

#### 2. Memory Fragmentation Analysis

```c
void analyze_fragmentation() {
    const int num_allocs = 1000;
    void* ptrs[num_allocs];
    
    // Allocate blocks of varying sizes
    for (int i = 0; i < num_allocs; i++) {
        size_t size = (rand() % 1000) + 1;
        ptrs[i] = malloc(size);
    }
    
    // Free every other block (creates fragmentation)
    for (int i = 0; i < num_allocs; i += 2) {
        free(ptrs[i]);
        ptrs[i] = NULL;
    }
    
    // Try to allocate large block
    void* large_block = malloc(50000);
    if (large_block == NULL) {
        printf("Fragmentation prevented large allocation\n");
    } else {
        printf("Large allocation succeeded\n");
        free(large_block);
    }
    
    // Cleanup
    for (int i = 1; i < num_allocs; i += 2) {
        free(ptrs[i]);
    }
}
```

---

## Summary

### Storage Allocation Strategy Comparison

| Strategy | Speed | Flexibility | Memory Efficiency | Complexity | Use Case |
|----------|-------|-------------|-------------------|------------|----------|
| **Static** | Fastest | Lowest | Poor (fixed) | Simplest | Global data, constants |
| **Stack** | Very Fast | Medium | Good | Simple | Local variables, recursion |
| **Heap** | Slowest | Highest | Variable | Complex | Dynamic data structures |

### Key Takeaways

1. **Static Allocation:**
   - Best for predictable, fixed-size data
   - Minimal runtime overhead
   - Limited flexibility

2. **Stack Allocation:**
   - Ideal for temporary, function-scope data
   - Automatic memory management
   - LIFO access pattern limitation

3. **Heap Allocation:**
   - Essential for dynamic, variable-sized data
   - Requires careful memory management
   - Supports complex data structures

4. **Dynamic Allocation:**
   - Provides runtime flexibility
   - Requires sophisticated algorithms
   - Trade-off between speed and flexibility

### Best Practices

1. **Use stack allocation** for temporary, small data
2. **Use static allocation** for global, constant data
3. **Use heap allocation** for dynamic, large, or shared data
4. **Implement memory pools** for frequent same-size allocations
5. **Consider garbage collection** for complex object lifetimes
6. **Profile and optimize** based on actual usage patterns

The choice of storage allocation strategy significantly impacts program performance, memory usage, and maintainability. Understanding the trade-offs enables developers to make informed decisions based on specific application requirements. 