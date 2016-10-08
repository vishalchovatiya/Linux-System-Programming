Pre-requisites : Memory Layout Of C Program OR Understanding Of Process Address Space 

### Allocating Memory on the Heap

- A process can allocate memory by increasing the size of the heap.
- Heap is a variable-size segment of contiguous virtual memory that  begins just after the uninitialized data segment of a process and grows & shrinks as memory is allocated and freed. 
- The current limit of the heap is referred to as the **program break** which is just at the end of the uninitialized data segment in process address space.
- Resizing the heap (i.e., allocating or deallocating memory) is actually as simple as telling the kernel to adjust its idea of  where the process’s program break is.
- To allocate memory, C programs normally use the malloc family of functions, which we describe shortly. However, we begin with a description of brk() and sbrk(), upon which the malloc functions are based.

### `brk()` & `sbrk()`

```
#include <unistd.h>

int brk(void end_data_segment); 

void *sbrk(intptr_t increment); 
```

- The `brk()` is system call which sets the program break to the location specified by `end_data_segment`. Since virtual memory is located in units of pages, `end_data_segment` is effectively rounded up to the next page boundary.
- A call to `sbrk()` adjusts the program break by adding increment to it. On Linux, `sbrk()` is a library function implemented on top of `brk()`. On  success,  `sbrk()`  returns  the  previous address  of  the  program  break.  In  other  words, if we have increased the program break, then the return value is a pointer to the start of the newly allocated block of memory. The  call  `sbrk(0)` returns  the  current  setting  of  the  program  break  without changing it. This can be useful if we want to track the size of the heap, perhaps in order to monitor the behavior of a memory allocation package.
- After the program break is increased, the program may access any address in the newly allocated area, but no physical memory pages are allocated yet. The kernel automatically allocates new physical pages on the first attempt by the process to access addresses in those pages.

### malloc 

- When you malloc, it first checks how much memory you requested.
- There are 2 ways to get memory from system : mmap, brk
- When you request some byte to be allocate by malloc it checks for `MMAP_THRESHOLD` limit(this also depends upon library implementations). If you request more than that limit, then it uses mmap system call to get the required memory.
- Else it use brk syscall, increament the program break size & gives you pointer to start of newly allocate contiguos block
- Whichever procedure it follows, it actually allocates a bit more memory than you asked for. This extra memory is used to store information such as the size of the allocated block, and a link to the next free/used block in a chain of blocks, and sometimes some guard data(that helps the system to detect if you write past the end of your allocated block). 
- Also, most allocators will round up the total size and/or the start of your part of the memory to a multiple of bytes (e.g. on a 64-bit system may align the data to a multiple of 64 bits (8 bytes) as accessing data from non-aligned addresses can be more difficult and inefficient for the processor/bus), so you may also end up with some "padding" (unused bytes).

### free

- As we know so far that we allocate the memory just by increasing program's break.
- And upon your intellect, you must be thinking that free will lower this program break.
- But its actually not, free simply adds the block of memory to list of free blocks that are recycled by future calls to `malloc()`. But why ?
- Well, the block of memory being freed is typically somewhere in middle of the heap, rather than at the end, so that lowering the program break is not possible.

### Will malloc allocates contiguously in memory or can it be scattered?

- Yes And No Both, Here is why
- In a typical OS, there exists the concepts of virtual memory and physical memory.
- Each process has its own virtual memory range which is contigous.
- When you allocate memory through malloc, program break is increased with respect to virtual memory so your allocated byte is contiguos.
- But the OS, behind the scenes, is busy mapping virtual memory allocations onto real blocks of physical memory which do not be contigious because this is how VMM(Virtual Memory Manager) works.


### How does free know how much to free?

- If you still can not figure out answer to this, you may read above topics again.
- Anyway. When you call malloc(), you specify the amount of memory to allocate. The amount of memory actually used is slightly more than this, and includes extra information that records (at least) how big the block is. 
- You can't (reliably) access that other information - and nor should you.
- When you call free(), it simply looks at the extra information to find out how big the block is.
