# Custom Heap Manager & Garbage Collector (Single-File Implementation)

This project is a self-contained, low-level implementation of a dynamic memory allocator and an automatic garbage collector (GC) written in C. It mimics the behavior of the standard heap but adds a **Mark-and-Sweep** GC algorithm to automatically reclaim unreachable memory.

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
* **Free List Inference:** Can analyze the heap state to generate a linked list of available memory gaps.

## Technical Implementation

The heap is structured as a contiguous byte array. Instead of using system `malloc` for every object, this allocator reserves one large block (`64KB`) and sub-allocates from it.

### The Allocation Record
Every allocation is tracked via a metadata record at the start of the heap:
* **2 Bytes:** Offset (Where the data lives in the heap)
* **2 Bytes:** Size (How big the object is)
* **1 Byte:** Pointer Count (How many pointers are inside this object, used for the GC to traverse children)

## Usage

Since this is a standalone implementation, you can compile it directly without linking external object files.

### 1. Compilation
Use `gcc` to compile the single source file.

```bash
gcc -o gc_allocator gc_allocator.c
