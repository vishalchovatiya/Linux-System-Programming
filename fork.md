> **Example**

```
#include <stdio.h>
#include <unistd.h>

int main(void)
{
        pid_t pid = fork();

        if( pid > 0 )
                printf("Parent Process: ParentPID = %d, ChildPID  = %d\n", getpid(), pid);
        else
                printf("Child  Process: ChildPID  = %d, ParentPID = %d\n", getpid(), pid);

        sleep(2);

        return 0;
}
```

> **Output**
Parent Process: ParentPID = 2640, ChildPID  = 2641
Child  Process: ChildPID  = 2641, ParentPID = 0

> **Points to Remember**
