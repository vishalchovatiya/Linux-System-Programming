> **Example**

```
#include<stdio.h>
#include<pthread.h>
#include<semaphore.h>

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

        sem_init( &sem, 0, 0);

        pthread_create( &thread1, NULL, (void*) ScanNumbers, (void*)Msg1);
        pthread_create( &thread2, NULL, (void*) SumAndPrint, (void*)Msg2);

        pthread_join(thread1, NULL);
        pthread_join(thread2, NULL);

        printf("Wait For Both Thread To Finish\n");

        sem_destroy(&sem);

        return 0;
}
```
- Two types of implementation of semaphore
        1) POSIX-semaphore: sem_init(),sem_wait(),sem_trywait(),sem_post(),sem_getvalue(),sem_destroy()
        2) System-V-semaphore: semget(),semop(),semctl()
- 

