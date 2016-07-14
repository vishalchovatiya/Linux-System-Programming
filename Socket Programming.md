### Points

- On a single system, Two process can communicate through
    1. Pipes
    2. Message queues
    3. Shared memory
- To communicate between two process over network, you need SOCKET
- Socket = End point of communication between two systems on a network OR Combination of IP & Port Number


### Server Example

```
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>

void ErrorAndExit(const char *str);

#define IP       "127.0.0.1"    /* Local Host OR Should be your local IP for test */
#define PORT     5000

int main(int argc, char *argv[])
{
        int listenfd = 0, connfd = 0;
        struct sockaddr_in serv_addr = {0};

        if( (listenfd = socket(AF_INET, SOCK_STREAM, 0) ) < 0 )
                ErrorAndExit("Could not create socket");

        serv_addr.sin_family = AF_INET;
        serv_addr.sin_addr.s_addr = inet_addr(IP);
        serv_addr.sin_port = htons(PORT);

        if( (bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) ) < 0 )
                ErrorAndExit("Could not bind on Given IP");

        if( (listen(listenfd, 10)  ) < 0 )
                ErrorAndExit("Could not listen");

        while(1)
        {
                if( ( connfd = accept(listenfd, (struct sockaddr*)NULL, NULL) ) < 0 )
                        ErrorAndExit("Could not accept");

                write(connfd, "Server Message", strlen("Server Message"));

                close(connfd);
                sleep(1);
        }
}

void ErrorAndExit(const char *str)
{
        printf("Error : %s\n", str);
        exit(EXIT_FAILURE);
}

```

### Client Example

```
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <netdb.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <arpa/inet.h>

void ErrorAndExit(const char *str);

#define IP       "127.0.0.1"    /* Local Host OR Should be your local IP for test */
#define PORT     5000

int main()
{
        int sockfd = 0, n = 0;
        char recvBuff[1024];
        struct sockaddr_in serv_addr = {0};

        memset(recvBuff, '0',sizeof(recvBuff));

        if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
                ErrorAndExit("Could not create socket");

        serv_addr.sin_family = AF_INET;
        serv_addr.sin_port = htons(PORT);

        if(inet_pton(AF_INET, IP, &serv_addr.sin_addr)<=0)
                ErrorAndExit("inet_pton error occured");

        if( connect(sockfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0)
                ErrorAndExit("Connect Failed");

        while ( (n = read(sockfd, recvBuff, sizeof(recvBuff)-1)) > 0)
        {
                recvBuff[n] = 0;
                if(fputs(recvBuff, stdout) == EOF)
                        ErrorAndExit("fputs error");
        }

        if(n < 0)
                ErrorAndExit("Read error");

        return 0;
}

void ErrorAndExit(const char *str)
{
        printf("Error : %s\n", str);
        exit(EXIT_FAILURE);
}
```
