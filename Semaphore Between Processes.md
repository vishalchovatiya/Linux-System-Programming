> **Example**
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <semaphore.h>
#include <errno.h>
#include <fcntl.h>
#include <sys/wait.h>

const char *semName = "asdfsd";

static void parent(void)
{
	sem_t *sem_id = sem_open(semName, O_CREAT, 0600, 0);

	if(sem_id == SEM_FAILED) 
	{
		perror("parent sem_open Failed\n");
		return;
	}

	printf("Parent: Wait for Child to Print\n");

	if( sem_wait(sem_id) < 0 )
		printf("Failed sem_wait\n"); 

	
	printf("Parent: Child Printed! ");

	if( sem_unlink( semName) < 0 )
		printf("\nFailed sem_unlink()\n");
	else
		printf(" Exiting\n");

}

static void child(void)
{
	sem_t *sem_id = sem_open(semName, O_CREAT, 0600, 0);

	if(sem_id == SEM_FAILED) 
	{
		perror("child sem_open Failed\n");
		return;
	}

	printf("Child: I am done! Release Semaphore\n");

	if( sem_post(sem_id) < 0 )
		printf("Failed sem_post()\n");
}

int main(int argc, char *argv[])
{
	pid_t pid;
	pid=fork();

	if(pid < 0) 
	{
		perror("fork");
		exit(EXIT_FAILURE);
	}

	if(!pid) 
	{
		child();    
	} 
	else 
	{
		int status;
		parent();
		wait(&status);
	}

	printf("Done with sem_open\n");

	return 0;
}
```
