> **Example**

```
#include<stdio.h>
#include<signal.h>
#include<unistd.h>

void sig_handler(int signo)
{
  if (signo == SIGINT)
    printf("received SIGINT\n");
}

int main(void)
{
        if (signal(SIGINT, sig_handler) == SIG_ERR)
                printf("\ncan't catch SIGINT\n");

        while(1)
                sleep(1);
        return 0;
}
```
> **Points to remember**

- Signals = software interrupts.
- The command kill -l on the bash would give us the following.

```
 1) SIGHUP            2) SIGINT         3) SIGQUIT     4) SIGILL     5) SIGTRAP
 6) SIGABRT           7) SIGBUS         8) SIGFPE      9) SIGKILL   10) SIGUSR1
11) SIGSEGV          12) SIGUSR2       13) SIGPIPE    14) SIGALRM   15) SIGTERM
16) SIGSTKFLT        17) SIGCHLD       18) SIGCONT    19) SIGSTOP   20) SIGTSTP
21) SIGTTIN          22) SIGTTOU       23) SIGURG     24) SIGXCPU   25) SIGXFSZ
26) SIGVTALRM        27) SIGPROF       28) SIGWINCH   29) SIGIO     30) SIGPWR
31) SIGSYS           34) SIGRTMIN      35) SIGRTMIN+1 36) SIGRTMIN+2 37) SIGRTMIN+3
38) SIGRTMIN+4       39) SIGRTMIN+5    40) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8
43) SIGRTMIN+9       44) SIGRTMIN+10   45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14      49) SIGRTMIN+15   50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11      54) SIGRTMAX-10   55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6       59) SIGRTMAX-5    60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1       64) SIGRTMAX
```

- Signals are also delivered to a process with the help of kill command. The manual page (man kill) of kill command says that the default and easier version of kill command is the kill pid. Where pid is the process ID that is found via the ps command. The default signal is SIGTERM (15). Alternatively a signal number is specified to the kill command such as kill -2 1291 making a delivery of SIGINT(2) signal to the process ID 1291
- Every thread has its own private signal mask(APIs like pthread_sigmask() etc) can be used to capture or block particular signal
- Some of the most important APIs to implement signal mechanisms are sigaction, signal and signalfd.
- signal() does not block other signals from arriving while the current signal is being executed. Thus when more than one signal occur at the same time, it becomes more problematic to understand and perform actions. If it is on the same data, this might even get more complex. The sigaction() blocks the other signals while the handler is being executed.
