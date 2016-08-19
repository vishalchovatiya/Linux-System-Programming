### Example

```
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
#include<syscall.h>

pthread_barrier_t barrier;

void *func1(void *arg)
{
	int var = 0;
	printf("Thread1: my id is %d and my parent id is %d\n", syscall(SYS_gettid), getpid());
	while(var < 5) {
		printf("Thread1: var is %d\n", var);
		var++;
		sleep(1);
	}
	pthread_barrier_wait(&barrier);
	printf("Thread1 exiting.....\n");
	pthread_exit(0);
}

void *func2(void *arg)
{
	int var = 0;
	printf("Thread2: my id is %d and my parent id is %d\n", syscall(SYS_gettid), getpid());
	while(var < 7) {
		printf("Thread2: var is %d\n", var);
		var++;
		sleep(1);
	}
	pthread_barrier_wait(&barrier);
	printf("Thread2 exiting.....\n");
	pthread_exit(0);
}

void *func3(void *arg)
{
	int var = 0;
	printf("Thread3: my id is %d and my parent id is %d\n", syscall(SYS_gettid), getpid());
	while(var < 9) {
		printf("Thread3: var is %d\n", var);
		var++;
		sleep(1);
	}
	pthread_barrier_wait(&barrier);
	printf("Thread3 exiting.....\n");
	pthread_exit(0);
}

int main()
{
	pthread_t thread1, thread2, thread3;

	pthread_barrier_init(&barrier, NULL, 3);
	
	printf("I am parent: %d\n", getpid());

	if (pthread_create(&thread1, NULL, func1, NULL) != 0) {
		perror("[FAIL]: pthread_create for thread1 failed\n");
		return -1;
	}

	if (pthread_create(&thread2, NULL, func2, NULL) != 0) {
		perror("[FAIL]: pthread_create for thread2 failed\n");
		return -1;
	}

	if (pthread_create(&thread3, NULL, func3, NULL) != 0) {
		perror("[FAIL]: pthread_create for thread3 failed\n");
		return -1;
	}

	if (pthread_join(thread1, NULL) != 0) {
		perror("[FAIL]: pthread_join failed thread1\n");
		return -1;
	} else
		printf("pthread_join success for thread1\n");

	if (pthread_join(thread2, NULL) != 0) {
		perror("[FAIL]: pthread_join failed thread2\n");
		return -1;
	}else
               printf("pthread_join success for thread2\n");

	if (pthread_join(thread3, NULL) != 0) {
		perror("[FAIL]: pthread_join failed thread3\n");
		return -1;
	}else
               printf("pthread_join success for thread3\n");

	printf("[PASS]: barrier successful\n");
}

```
