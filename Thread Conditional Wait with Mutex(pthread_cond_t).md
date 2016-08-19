### Example

```
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<pthread.h>
#include<syscall.h>

int glovar, state = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

void *thread1_func(void *arg)
{
	printf("\n%s: my id is %d and my parent id is %d\n",
				arg, syscall(SYS_gettid), getpid());
	if (pthread_mutex_lock(&mutex) != 0) {
		perror("[FAIL]: Thread1: pthread_mutex_lock failed\n");
		exit(-1);
	} else
		printf("Thread1: pthread_mutex_lock success.....\n");

	while (!state) {
		printf(".....waiting for signal\n");
		pthread_cond_wait(&cond, &mutex);
	}
	if (pthread_mutex_unlock(&mutex) != 0) {
		perror("[FAIL]: Thread1: pthread_mutex_unlock failed\n");
		exit(-1);
	} else
		printf("Thread1: pthread_mutex_unlock success.....\n");

	printf("\nThread1: state value signaled high, thread1 starts execution\n");
	while (glovar < 5) {
		printf("Thread1: glovar is %d\n", glovar);
		glovar++;
		sleep(1);
	}
	pthread_exit("EXIT_SUCCESS");
}

void *thread2_func(void *arg)
{
	printf("\n%s: my id is %d and my parent id is %d\n", 
				arg, syscall(SYS_gettid), getpid());
	if (pthread_mutex_lock(&mutex) != 0) {
		perror("[FAIL]: Thread2: pthread_mutex_lock failed\n");
		exit(-1);
	} else
		printf("Thread2: pthread_mutex_lock success.....\n");

	printf("setting state value to high\n");
	state = 1;
	if (pthread_mutex_unlock(&mutex) != 0) {
		perror("[FAIL]: Thread2: pthread_mutex_unlock failed\n");
		exit(-1);
	} else
		printf("Thread2: pthread_mutex_unlock success.....\n");

	if (pthread_cond_signal(&cond) != 0) {
		perror("[FAIL]: pthread_cond_signal failed\n");
		exit(-1);
	} else
		printf("pthread_cond_signal success\n");

	pthread_exit("EXIT_SUCCESS");
}

int main()
{
	pthread_t thread1, thread2;
	char *msg1 = "I am thread1";
	char *msg2 = "I am thread2";
	void *recv_status;
	
	if(pthread_mutex_init(&mutex, NULL) != 0) {
		perror("[PASS]: mutex init failed\n");
		return -1;
	}

	printf("\n.....Execution starts from here.....\n");
	printf("Main Thread: my id is %d\n", getpid());

	if (pthread_create(&thread1, NULL, thread1_func, msg1) != 0) {
		perror("[FAIL]: error in creating thread1\n");
		return -1;
	}
	
	if (pthread_create(&thread2, NULL, thread2_func, msg2) != 0) {
		perror("[FAIL]: error in creating thread2\n");
		return -1;
	}

	if (pthread_join(thread1, (void *)&recv_status) != 0) {
		perror("[FAIL]: error in joining thread1\n");
		return -1;
	}
	printf("return status of thread1 is %s\n", recv_status);

	if (pthread_join(thread2, (void *)&recv_status) != 0) {
		perror("[FAIL]: error in joining thread2\n");
		return -1;
	}
	printf("return status of thread2 is %s\n", recv_status);
	
	if (pthread_mutex_destroy(&mutex) != 0) {
		perror("[FAIL]: error in destroy\n");
		return -1;
	}
	
	printf("[PASS]: Thread created and joined successfully\n");
	return 0;
}

```
