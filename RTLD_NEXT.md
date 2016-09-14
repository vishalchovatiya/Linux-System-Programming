### Intro

- Let's understand by example
- If there is four shared library is loaded dynamically named as `A.so`, `B.so`, `C.so` & `D.so`. In which `C.so` having following code :
```
        if ((fptr = (int (*)())dlsym(RTLD_NEXT, "funcXYZ")) == NULL) {
                (void) printf("dlsym: %s\n", dlerror());
                exit (1);
        }
        
        return ((*fptr)());
```
- Then `funcXYZ` will be searched for in object `D.so` which just loaded after `C.so` & pointer to function is returned

### What is RTLD_NEXT use for ?

- RTLD_NEXT allows one to provide a wrapper around a function defined in another shared library.
- In other words, you can exploit methods defined in another shared library

### Let's Understand with example

#### malloc.c

```
#include    <stdio.h>
#include    <dlfcn.h>

void *malloc(size_t size)
{
        static void * (* fptr)() = 0;

        /* look up of malloc, the first time we are here */
        if (fptr == 0) {
                fptr = (void * (*)())dlsym(RTLD_NEXT, "malloc");
                if (fptr == NULL) {
                        printf("dlsym: %s\n", dlerror());
                        return (0);
                }
        }

        printf("Our Malloc\n");

        return (*fptr);
}
```

#### main.c

```
#include<stdio.h>
#include<stdlib.h>

int main(){

        malloc(1);

        return 0;
}
```

#### Compilation & Run
```
$ gcc -o malloc.so -shared -fPIC malloc.c -D_GNU_SOURCE
$ gcc -o main main.c malloc.so -ldl
$ ./main
Our Malloc
```

### How it works
- When you compile your `main.c` by `gcc -o main main.c malloc.so -ldl`, you specify `malloc.so` explicitly on first order. We can verify this by `ldd` command
```
$ ldd main
        linux-vdso.so.1 =>  (0x00007fff37bf4000)
        malloc.so.1 (0x00007fc5df598000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007fc5df37d000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fc5defbb000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fc5df79b000)
```
- So when you call malloc it will refer first occurence of symbol which is in our `malloc.so` file

- But if you specify `libc.so.6` explicitly on before `malloc.so`, then compile & run will give you different result
```
$ export LD_LIBRARY_PATH=$(pwd)
$ echo $LD_LIBRARY_PATH
/home/vishal/workspace/next
$ gcc -o main main.c /usr/lib64/libc.so.6 ./malloc.so.1 -ldl
$ ./main
$

```
- Now It calls malloc function defined in `ld-linux-x86-64.so.2`  which is the dynamic linker/loader insted of `libc.so.6`
- Well dynamic linker/loader has its own copy of `malloc()` and `free()`. Why? Because `ld-linux` has to allocate memory from the heap before it loads `libc.so.6`. 
- But why does `malloc.so` forward calls to `ld-linux` instead of `libc` ? The answer comes down to how `dlsym()` searches for symbols when `RTLD_NEXT` is specified. `RTLD_NEXT` will find the next occurrence of a function in the search order after the current library. 
- To understand this better, take a look at `ldd` output for the new `main` binary:
```
$ ldd main
        linux-vdso.so.1 =>  (0x00007fff053fb000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f0f1a235000)
        ./malloc.so.1 (0x00007f0f1a032000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f0f19e2e000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f0f1a60e000)
```
- Unlike earlier, `malloc.so` comes after `libc.so.6`. So when `dlsym()` is called inside `malloc.so` to search for functions, it skips `libc.so.6` since it precedes `malloc.so` in the search order list. That means the searches continue through to `ld-linux-x86-64.so.2` where they find linker/loader’s malloc/free and return pointers to those functions. And so, `malloc.so` ends up forwading calls to `ld-linux` instead of `libc`!




