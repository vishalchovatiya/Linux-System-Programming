## Points To Remember

crt0.o 		  =  Will contain the _start function that initializes the process
crtbegin.o	=  Call init
crtend.o	  =  Call fini
crti.o		  =  Header of init & fini (for push in stack)
crtn.o 		  =  Footer of init & fini (for pop in stack)

- There could be crt1.o, crt2.o & so on, depending upon implementation, crt0.c is runtime 0 & funs first
- glibc ports call this file 'start.S' while uClibc ports call this crt0.S or crt1.S
- C runtime files are linked in this order: crt0.o, crti.o, crtbegin.o, your-program-foo.o, crtend.o, crtn.o. 

### crt0.s 
```
.text 
.global  _start  
_start : 
       xor %rbp,%rbp 
       mov %rsp,%rdi 
.weak _DYNAMIC 
.hidden _DYNAMIC 
       lea _DYNAMIC(%rip),%rsi 
       andq $-16,%rsp 
       call  _start_c 
```
**Note**: Sometime this code could be in `crt_arch.h`

### crt1.c

```
#include <features.h>

#define START "_start"

#include "crt_arch.h"

int main();
void _init() __attribute__((weak));
void _fini() __attribute__((weak));
_Noreturn int __libc_start_main(int (*)(), int, char **,
        void (*)(), void(*)(), void(*)());

void _start_c(long *p)
{
        int argc = p[0];
        char **argv = (void *)(p+1);
        __libc_start_main(main, argc, argv, _init, _fini, 0);
}
```
### crti.s
```
.section .init
.global _init
_init:
        push %rax

.section .fini
.global _fini
_fini:
        push %rax
```
### crtn.s
```
.section .init
        pop %rax
        ret

.section .fini
        pop %rax
        ret
```

### Flow of x86 C program
```
_init & _fini(push)	  ./crt/x86_64/crti.s
1. _start		          ./arch/x86_64/crt_arch.h
2. _start_c		        ./crt/crt1.c
3. __libc_start_main	./src/env/__libc_start_main.c
4. main			          Our Program
_init & _fini(pop)	  ./crt/x86_64/crtn.s
```
