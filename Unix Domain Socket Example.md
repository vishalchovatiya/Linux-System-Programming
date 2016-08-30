### Server
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/socket.h>
#include <sys/un.h>

static void displayError(const char *);

const char socketPath[]  = "/tmp/my_sock";	/* Socket With Absolute Pathname */

int main(int argc, char **argv, char **envp) {
	int sockfd;  				/* Socket File Descriptor */
	int connfd;  				/* Connection Fide Descriptor On Whch We Communicate */
	struct sockaddr_un socketAddr;		/* AF_UNIX */

	/* Create a AF_UNIX (aka AF_LOCAL) socket */
	if ( (sockfd = socket(AF_UNIX,SOCK_STREAM,0)) < 0 )
		displayError("socket");

	/* Here we remove name of socket from file system, in case it existed from a prior run.*/
	unlink(socketPath);

	memset( &socketAddr, 0, sizeof socketAddr);
	socketAddr.sun_family = AF_UNIX;
	strncpy( socketAddr.sun_path, socketPath,  sizeof socketAddr.sun_path-1);

	if ( bind(sockfd, (struct sockaddr *)&socketAddr, sizeof(struct sockaddr)) < 0 )
		displayError("Could not bind");

	if( ( listen( sockfd, 10)  ) < 0 )
		displayError("Could not listen");

	if( ( connfd = accept( sockfd, (struct sockaddr*)NULL, NULL) ) < 0 )
		displayError("Could not accept");

	write(connfd, "Server Message\n", strlen("Server Message\n"));

	/* Close & unlink our socket path */
	close(sockfd) ;
	unlink(socketPath);

	return 0;
}

/* This function reports the error and exits back to the shell: */
static void displayError(const char *on_what) {
	perror(on_what);
	exit(1);
}

```
### Client
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/socket.h>
#include <sys/un.h>

static void displayError(const char *on_what);

const char socketPath[]  = "/tmp/my_sock";	

int main(int argc, char **argv, char **envp) {
	int n = 0;
	char recvBuff[1024];
	int sockfd;  				
	struct sockaddr_un socketAddr;			

	/* Create a AF_UNIX (aka AF_LOCAL) socket: */
	if ( (sockfd = socket(AF_UNIX,SOCK_STREAM,0)) < 0 )
		displayError("socket");

	memset( &socketAddr, 0, sizeof socketAddr);
	socketAddr.sun_family = AF_UNIX;
	strncpy( socketAddr.sun_path, socketPath,  sizeof socketAddr.sun_path-1);

	if( connect(sockfd, (struct sockaddr *)&socketAddr, sizeof(socketAddr)) < 0)
		displayError("Connect Failed");

	while ( (n = read(sockfd, recvBuff, sizeof(recvBuff)-1)) > 0)
	{
		recvBuff[n] = 0;
		if(fputs(recvBuff, stdout) == EOF)
			displayError("fputs error");
	}

	if(n < 0)
		displayError("Read error");

	/* Close and unlink our socket path: */
	close(sockfd) ;
	unlink(socketPath);

	return 0;
 }


/* This function reports the error and exits back to the shell: */
static void displayError(const char *on_what) {
	perror(on_what);
	exit(1);
}
```
