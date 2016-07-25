### Server.c
```
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define SHMSZ     27    //Size of shared memory

int main()
{
        char c;
        int shmid;
        int shmflg = IPC_CREAT | 0666;  // flags like create, access permission
        key_t key = 5678;               // Name of Shared Memory Segment
        char *shm, *s;

        if ((shmid = shmget(key, SHMSZ, shmflg)) < 0) {         //Create the segment
                perror("shmget");
                return 1;
        }

        if ((shm = shmat(shmid, NULL, 0)) == (void *) -1) {     //Attach segment to data space
                perror("shmat");
                return 1;
        }

        s = shm;                // Fill memory to read by other process

        for (c = 'a'; c <= 'z'; c++)
                *s++ = c;
        *s = NULL;

        while (*shm != '*')     // Wait for other process acknowledgement
                sleep(1);

        return 0;
}
```
### Client.c
```
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define SHMSZ     27

int main()
{
        int shmid;
        int shmflg = 0666;  // flags like create, access permission
        key_t key = 5678;
        char *shm, *s;

        if ((shmid = shmget(key, SHMSZ, shmflg)) < 0) {         // Locate the segment
                perror("shmget");
                return 1;
        }

        if ((shm = shmat(shmid, NULL, 0)) == (void *) -1) {     // Attach segment to data space
                perror("shmat");
                return 1;
        }

        for (s = shm; *s != NULL; s++)  // Read what other process writes here
                putchar(*s);
        putchar('\n');

        *shm = '*';                     // Acknowledge other proces

        return 0;
}
```
### Points to be note
- shmget() : Creates a shared memory segment, The key argument could be semaphore ID
- shmat() : Shared segment can be attached to a process address space using this API
- It can be detached using shmdt() (see shmop()), A shared segment can be attached multiple times by the same process
- The original owner of a shared memory segment can assign ownership to another user with shmctl()
