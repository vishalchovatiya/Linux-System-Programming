Pre-requisites : Memory Layout Of C Program OR Understanding Of Process Address Space 

### Allocating Memory on the Heap

- A process can allocate memory by increasing the size of the heap.
- Head is a variable-size segment of contiguous virtual memory that  begins just after the uninitialized data segment of a process and grows & shrinks as memory is allocated and freed. 
- The current limit of the heap is referred to as the **program break** which is just at the end of the uninitialized data segment in process address space.
- Resizing the heap (i.e., allocating or deallocating memory) is actually as simple as telling the kernel to adjust its idea of  where the process’s program break is.
- To allocate memory, C programs normally use the malloc family of functions, which we describe shortly. However, we begin with a description of brk() and sbrk(),upon which the malloc functions are based 

### `brk()` & `sbrk()`

```
#include <unistd.h>

int brk(void end_data_segment); 

void *sbrk(intptr_t increment); 
```

- The `brk()` system   call   sets   the   program   break   to   the   location   specified   by end_data_segment.Since virtual memory is located in units of pages, end_data_segment is effectively rounded up to the next page boundary
- A call to `sbrk()` adjusts the program break by adding increment to it. On Linux, `sbrk()` is a library function implemented on top of `brk()`. On  success,  `sbrk()`  returns  the  previous address  of  the  program  break.  In  other  words, if we have increased the program break, then the return value is a pointer to the start of the newly allocated block of memory. The  call  `sbrk(0)` returns  the  current  setting  of  the  program  break  without changing it. This can be useful if we want to track the size of the heap, perhaps in order to monitor the behavior of a memory allocation package.
- After the program break is increased, the program may access any address in the newly allocated area, but no physical memory pages are allocated yet. The kernel automatically allocates new physical pages on the first attempt by the process to access addresses in those pages.

### malloc 

- I am not going to discuss argments & return type of these functions rather we directly jump to How They Works
- There are 2 ways to get memory from system mmap, brk
- When you request some byte to allocate by malloc it checks for MMAP_THRESHOLD limit. If you request more than that limit, then it uses mmap system call to get the required memory 
- Else it use brk syscall & increament the program break size & gives you pointer to start of newly allocate contiguos block

### free


Q. Will malloc allocates contiguously in memory or can it be scattered?
A. Yes And No Both, Here is why
- In a typical OS, there exists the concepts of virtual memory and physical memory.
- Each process has its own virtual memory range which is contigous.
- When you allocate memory through malloc, program break is increased with respect to virtual memory so your allocated byte is contiguos
- But the OS, behind the scenes, is busy mapping virtual memory allocations onto real blocks of physical memory which do not be contigious because its how VMM works.




So you can still run into virtual memory fragmentation. Here, a process requests a block of memory and there isn't, in that particular processes virtual memory map, a block of contigious virtual memory such that the request can be satisfied.

That problem is the problem you're thinking of.
