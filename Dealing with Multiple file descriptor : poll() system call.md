### Why use poll()

- Suppose you have to deal with multiple clients connected at the same time. A natural question, then, is: how can you read from multiple file descriptors (sockets) at once? 
- Do you need to make some really annoyingly multi-threaded code to handle each client that's connected? 
- Do you have to go into some stupid loop constantly checking each socket to see if data's available? 
- You can resolve this issue efficiently by polling file descriptor(Sockets here).

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

### Points to note

- The idea behind multiplexing is that the operating system (kernel) knows when data is ready on a socket. You hand the operating system an array of file descriptors and say "tell me when something happens on one of these". 
- Your code blocks (stops executing) until there is data ready for you. At that point, you iterate through your array of file descriptors to determine which one has data ready.
