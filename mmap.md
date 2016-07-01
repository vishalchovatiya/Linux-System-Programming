> **Example**

- Following sample program is passed a filename as an argument. It opens the file, get the file status parameters, maps it, closes it, prints the file byte-by-byte to standard out, and then unmaps the file from memory.

```
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>


int main (int argc, char *argv[])
{
        if (argc < 2)           return 1;

        int fd = open ( argv[1], O_RDONLY);

        struct stat sb;
        fstat (fd, &sb);  // Fill File Status Structure
        
        //void * mmap (void *addr /* Address suggestion to the kernel of where best to map the file*/,
        //      size_t len /* Length of Mapping */,
        //     int prot /* Memory Protection Flags */,
        //     int flags /* Type of Mapping like shared, private, etc*/,
        //     int fd /* File descriptor */,
        //     off_t offset /* Offset to start mapping from in file, must be multiple of page size*/);
        
        void *p = mmap (0, sb.st_size, PROT_READ, MAP_SHARED, fd, 0);   // Map File in Memory

        close (fd);

        int len;
        for (len = 0; len < sb.st_size; len++)
                putchar ( * ((char*) p + len ));

        munmap (p, sb.st_size);   // Un-Map Mapped File from memory

        return 0;
}
```
> **Points to Note**

1. mmap() is system call used to maps files or devices into memory
2. Linux provides the mremap( ) system call for expanding or shrinking the size of a given mapping.
3. POSIX defines the mprotect( ) interface to allow programs to change the permissions of existing regions of memory.
4. Synchronizing a File with a Mapping : A system call msync( ) flushes back to disk any changes made to a file mapped via mmap( ), synchronizing the mapped file with the mapping

> **Advantage of mmap() over open(), read() & write()**

1. mmap is great if you have multiple processes accessing data in a read only fashion from the same file which saves many system calls or context switching overheads
2. Useful for inter process communication. You can mmap a file as read/write in the processes that need to communicate and then use sychronization primitives in the mmapped region (this is what the MAP_HASSEMAPHORE flag is for).

> **Limitation of mmap()**

1. Not best fit for mapping large file as mmap has to  find a contiguous block of addresses in your process's address space that is large enough to fit the entire range of the file being mapped. In this case you may have to map the file in smaller chunks than you would like to make it fit.
2. Awkwardness with mmap as a replacement for read / write is that you have to start your mapping on offsets of the page size


