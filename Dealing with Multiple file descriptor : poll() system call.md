### poll() system call

```
#include <fcntl.h>
#include <stdio.h>
#include <sys/poll.h>
#include <sys/time.h>
#include <unistd.h>

int main( int argc, char *argv[])
{
        char buf[1024];
        struct pollfd pfds[1] = { 0, POLLIN};   // No of files to poll for
        int timeout = 5000;                     // Time Out

        int ret = poll(pfds, 1, timeout);       // Polling for file discriptors provided with timeout

        if ( ret && pfds[0].revents & POLLIN) {
                int i = read(0, buf, 1024);
                buf[i] = 0;
                printf("You Typed : %s\n", buf);
        }
        else if( ret == 0 ){
                printf("Time Out!\n");
        }

        return 0;
}
```
