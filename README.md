# Custom Heap Manager & Garbage Collector in C

This project is a low-level implementation of a dynamic memory allocator and an automatic garbage collector (GC) written in C. It mimics the behavior of the standard heap but adds a **Mark-and-Sweep** GC algorithm to automatically reclaim unreachable memory.

This demonstrates core systems concepts including pointer arithmetic, heap fragmentation management, and memory introspection.

## Key Features

### 1. Custom Allocation (`p4malloc` / `p4calloc`)
* Implements a custom allocation strategy using a **First-Fit** algorithm.
* Manages a raw memory arena (`p4heap`) by manually manipulating byte offsets.
* Handles memory alignment (8-byte alignment) to ensure architectural compatibility.

### 2. Garbage Collection (`p4gc`)
* **Algorithm:** Mark-and-Sweep.
* **Root Scanning:** Takes a list of "live roots" (stack pointers) and recursively traverses the graph of allocated objects.
* **Reclamation:** Identifies unmarked (unreachable) objects and frees them, compacting the metadata list.

### 3. Heap Introspection
* **Metadata Management:** Uses a custom 5-byte header for every allocation to track offset, size, and pointer references.
* **Free List Inference:** Can analyze the heap state to generate a linked list of available memory gaps (`infer_free_list`).

## Technical Implementation

The heap is structured as a contiguous byte array. Instead of using system `malloc` for every object, this allocator reserves one large block and sub-allocates from it.

### The Allocation Record
Every allocation is tracked via a metadata record at the start of the heap:
* **2 Bytes:** Offset (Where the data lives in the heap)
* **2 Bytes:** Size (How big the object is)
* **1 Byte:** Pointer Count (How many pointers are inside this object, used for the GC to traverse children)

### The GC Cycle
1.  **Mark:** The `p4gc` function iterates through `live_roots`. For every reachable object, it sets a bit in a temporary bitmask. If an object contains pointers, the GC recursively marks those children.
2.  **Sweep:** The function iterates through the allocation list. Any object that was *not* marked in the first phase is passed to `p4free` to reclaim space.

## Usage

### Dependencies
* `p4machine.h`: Defines the `p4heap` structure and constants (e.g., `P4HEAP_TOTAL_SIZE`).

### Example Workflow
```c
#include "p4machine.h"

int main() {
    // 1. Initialize the heap
    p4heap *heap = p4heap_create();

    // 2. Allocate memory (16 bytes, containing 0 pointers)
    void *ptr1 = p4malloc(heap, 16, 0);

    // 3. Allocate memory that points to other memory (16 bytes, 2 pointers)
    void **ptr2 = p4malloc(heap, 16, 2);
    ptr2[0] = ptr1; // Link objects

    // 4. Run Garbage Collector
    // (Assuming we pass a list of stack variables as roots)
    p4gc(heap, my_stack_roots);
    
    return 0;
}
