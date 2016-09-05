> **Example**

- Program simply creates two process & print messages in synchronized manner

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <semaphore.h>
#include <errno.h>
#include <fcntl.h>
#include <sys/wait.h>

const char *semName = "asdfsd";

void parent(void)
{
	sem_t *sem_id = sem_open(semName, O_CREAT, 0600, 0);

	if(sem_id == SEM_FAILED) 
	{
		perror("Parent  : [sem_open] Failed\n");
		return;
	}

	printf("Parent  : Wait for Child to Print\n");

	if( sem_wait(sem_id) < 0 )
		printf("Parent  : [sem_wait] Failed\n"); 


	printf("Parent  : Child Printed! \n");

	if (sem_close(sem_id) != 0) {
		perror("Parent	: [sem_close] Failed\n");
		return ;
	} 

	if( sem_unlink(semName) < 0 ){
		printf("Parent  : [sem_unlink] Failed\n");
		return ;
	}
}

void child(void)
{
	sem_t *sem_id = sem_open(semName, O_CREAT, 0600, 0);

	if(sem_id == SEM_FAILED) 
	{
		perror("Child   : [sem_open] Failed\n");
		return;
	}

	printf("Child	: I am done! Release Semaphore\n");

	if( sem_post(sem_id) < 0 )
		printf("Child   : [sem_post] Failed \n");
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
		printf("Child	: Done with sem_open \n");	
	} 
	else 
	{
		int status;
		parent();
		wait(&status);
		printf("Parent  : Done with sem_open \n");	
	}


	return 0;
}
```
- sem_open() : Connects to, & optionally creates, a named semaphore( like sem_init() )
- sem_unlink() : Ends connection to an open semaphore & causes the semaphore to be removed when the last process closes it( like sem_destroy()) 
- sem_wait() : Wait for semaphore to acquire
- sem_post() : Release semaphore
