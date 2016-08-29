### Example

```
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
#define SIZE 5
int nums[SIZE] = {0, 1, 2, 3, 4};
int main()
{
        int i;
        pid_t pid;
        pid = vfork();
        if(pid == 0){  /* Child process */
                for(i = 0; i < SIZE; i++){
                        nums[i] *= -i;
                        printf("CHILD: %d \n", nums[i]);
                }
                _exit(0);
        }
        else if (pid > 0){  /* Parent process */
                wait(NULL);
                for(i = 0; i < SIZE; i++)
                        printf("PARENT: %d \n", nums[i]);
        }
        return 0;
}

```

### Sample Output

```
CHILD: 0
CHILD: -1
CHILD: -4
CHILD: -9
CHILD: -16
PARENT: 0
PARENT: -1
PARENT: -4
PARENT: -9
PARENT: -16
```

### What is vfork ? 

- is a special case of clone. It is used to create new processes without copying the page tables of the parent process.
- calling thread is suspended until the child call execve or _exit.

### Points To Remember

- vfork() is an obsolete optimization. 
- Before good memory management, fork() made a full copy of the parent's memory, so it was pretty expensive. 
- since in many cases a fork() was followed by exec(), which discards the current memory map and creates a new one, it was a needless expense. 
- Nowadays, fork() doesn't copy the memory; it's simply set as "copy on write", so fork()+exec() is just as efficient as vfork()+exec()
- In Some OSs, vfork shares same address space
- vfork & fork internally calls clone
