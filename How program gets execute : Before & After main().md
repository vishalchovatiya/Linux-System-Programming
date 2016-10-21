
As we already seen what happen behind the scene when you type `./a.out` & hit enter in [How Program Gets Run](https://github.com/VisheshPatel/Linux-System-Programming/blob/master/How%20Program%20Gets%20Run.md). In [How Program Gets Run](https://github.com/VisheshPatel/Linux-System-Programming/blob/master/How%20Program%20Gets%20Run.md), we mostly focus on kernel level activity and left topic when our binary start running on processor core. In this article we will focus on complete user space activity life cycle of our program & see How program gets execute : Before & After `main()`.

- Before moving to topic. we first clear fundamental architecture of the GNU/Linux operating system

## Architecture of the GNU/Linux operating system

```
             |---------  User Application        -----------------------------------|
             |                  |                                                   |
User Space---|              C Library                                               |
             |-------------     |                                                   |
                        Syscall Interface  -----------|                             |
                                |                     |                             |-----GNU/Linux OS
                              Kernel                  |                             |
                                |                     |---Kernel Space              |
                    Architecture Depedent Code        |                             |
                                |    -----------------|                             |
                        Hardware Plateform       -----------------------------------|
```
- To understand above diagram practically, we consider following simple hello world program.

#### hello.c
```
#include<stdio.h>
int main(){ printf("Hello World"); return 0;}
```
- Let's Compile, `gcc hello.c`.
- As you can see in diagram, user application is your program, for simplicity, it is `main()` function.
- When you use `printf` whose definition is included by `#include<stdio.h>` is linked at compile time explicitly with `libc.so.6`(you can check it by `ldd a.out`).
- This `printf` function is declared & defined in `libc.so.6` which is C Library in above diagram.
- `printf` function internally calls write system call & kernel execution is started.
- And finally output of your program is printed on standard output.

But there is also some functions which are called before & after main, let see.

## Before `main()`

- When execution control trasfer from kernel mode to user mode it first look for linker entry point.
- This linker entry point is nothing but simple architecture dependent function named as `_start` written in assembly code.
- You can find out this function code for x86 architecture in glibc(2.14) source code file [sysdeps/x86_64/elf/start.S](http://osxr.org/glibc/source/sysdeps/x86_64/elf/start.S?v=glibc-2.14).
- Generally this entry point is found in `crt1.o` after compilation which will add explicitly by compiler at compile time(you can check it by `objdump -Dw crt1.o > dump`, find `_start`).
- After getting command line argument, environment variables, address of `main`, `init`(constructor), `fini`(destructor) & `exit`, `_start` function calls `__libc_start_main` located in [csu/libc-start.c](https://github.com/lattera/glibc/blob/master/csu/libc-start.c).
```
...
        /* Call the user's main function, and exit with its value.
           But let the libc call main.    */
        call BP_SYM (__libc_start_main)
#endif
...
```
- `__libc_start_main` will do some necessary sanity checks, execute `_init` constructor & invoke our `main()` function.

```
...
#else
  /* Nothing fancy, just call the function.  */
  result = main (argc, argv, __environ MAIN_AUXVEC_PARAM);
#endif

exit (result);
...
```
- There are also lot of functions execute between entry point & main which i have not discussed here because they are library helping routines which will hamper simplicity of this article.

## After `main()`

- As seen clearly on above `__libc_start_main` piece of code, return value of main is passed to exit function.
- This library function `exit` will call some library helper functions, `_fini` destructor & `_exit` function which will call exit system call & our program terminates.
```
void
_exit (status)
     int status;
{
  while (1)
    {
#ifdef __NR_exit_group
      INLINE_SYSCALL (exit_group, 1, status);
#endif
      INLINE_SYSCALL (exit, 1, status);

#ifdef ABORT_INSTRUCTION
      ABORT_INSTRUCTION;
#endif
    }
}
```
## Execution Flow Summary 

- Flow of x86 C program with runtime(according to glibc 2.14)

Routine | File
------------ | -------------
_start		  |        crt1.o
__libc_start_main	|      libc-start.c
_init (constructor)	 | crti.s
main		|	          Our Program
exit		|	          exit.c
_fini (destructor)	 | crtn.s
_exit         |      Exit system call




http://stackoverflow.com/questions/16483235/c-run-function-before-after-main-ended

https://en.wikipedia.org/wiki/Entry_point


http://stackoverflow.com/questions/2766233/what-is-the-c-runtime-library-------Microsoft

https://en.wikipedia.org/wiki/Runtime_library

https://en.wikipedia.org/wiki/Crt0

