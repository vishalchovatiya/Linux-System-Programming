
### malloc.c

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

### main.c

```
#include<stdio.h>
#include<stdlib.h>

int main(){

        malloc(1);

        return 0;
}
```

### Compilation & Run
```
$ gcc -o malloc.so -shared -fPIC malloc.c -D_GNU_SOURCE
$ gcc -o main main.c malloc.so -ldl
$ ./main
Our Malloc
```

### How it works
- When you compile your main.c by `gcc -o main main.c malloc.so -ldl`, you specify malloc.so explicitly on first order. We can verify this by `ldd` command
```
$ ldd main
        linux-vdso.so.1 =>  (0x00007fff37bf4000)
        malloc.so.1 (0x00007fc5df598000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007fc5df37d000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fc5defbb000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fc5df79b000)
```
- So when you call malloc it will refer first occurence of sybol which is in our malloc.so file

- But if you set `LD_LIBRARY_PATH` for our malloc.so, then compile & run will give you different result
```
$ export LD_LIBRARY_PATH=$(pwd)
$ echo $LD_LIBRARY_PATH
/home/vishal/workspace/next
$ gcc -o main main.c -ldl
$ ./main
$

```

