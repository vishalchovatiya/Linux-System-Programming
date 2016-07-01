> **Example of POSIX-semaphore**

```
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

int a,b;
sem_t sem;

void ScanNumbers( void *ptr)
{
        while(1)
        {
                printf("%s",(char*)ptr);
                scanf("%d %d", &a, &b);

                sem_post(&sem);
                usleep( 100 * 1000  );
        }
}

void SumAndPrint( void *ptr)
{
        while(1)
        {
                sem_wait(&sem);
                printf("%s %d\n", (char*)ptr, a+b);
        }
}

int main()
{
        pthread_t thread1;
        pthread_t thread2;

        char *Msg1 = "Enter Number Two No\n";
        char *Msg2 = "sum = ";

        // int sem_init(sem_t *sem /* pointer to semaphore variable*/, 
                        int pshared /* If = 0: Semaphore can be used in threads only, else in process*/, 
                        unsigned int value /* initial value of the semaphore counter*/);  
        // return value 0 on successful & -1 on failure
        
        sem_init( &sem, 0, 0);          // Can also use `sem = sem_open( "SemaphoreName", O_CREAT, 0777, 0);`

        pthread_create( &thread1, NULL, (void*) ScanNumbers, (void*)Msg1);
        pthread_create( &thread2, NULL, (void*) SumAndPrint, (void*)Msg2);

        pthread_join(thread1, NULL);
        pthread_join(thread2, NULL);

        printf("Wait For Both Thread To Finish\n");

        sem_destroy(&sem);          // Can also use `sem_unlink( "SemaphoreName");`

        return 0;
}
```
- sem_init() : Initialize Semaphore
- sem_destroy() : releases all resources
- sem_wait() : Wait for semaphore to acquire
- sem_post() : Release semaphore
- sem_trywait() : Only works when caller does not have to wait
- sem_getvalue() : Reads the counter value of the semaphore
- sem_open() : Connects to, & optionally creates, a named semaphore( like sem_init() )
- sem_unlink() : Ends connection to an open semaphore & causes the semaphore to be removed when the last process closes it( like sem_destroy()) 


> **Things to remember**

- Semaphore's internal implementation is like memory mapped file

- Two standard of semaphore mechanism

1) **POSIX-semaphore**: sem_init(), sem_destroy(), sem_wait(), sem_post(), sem_trywait(), sem_getvalue(), sem_open(), sem_unlink()

2) **System-V-semaphore**: semget(), semop(), semctl()



