To understanding function stack frame & Application Binary Interface(ABI), first we summing up [Wikipedia](https://en.wikipedia.org/wiki/Application_binary_interface) definition of ABI & later goes to understand stack frame. 

According to wikipedia, an application binary interface (ABI) is the interface between two program modules, one of which is often a library or operating system, at the level of machine code.

Above definition seems good but can't give me confidence to understand in simple words. So i dig deep to understand that practically & see what i found.

# What is ABI ?
- ABI is protocol followed by compiler, OS or library writer which defines the mechanisms by which functions are invoked, how parameters are passed to function, how return values are retrived, how a process invokes system calls, etc.
- ABI also defines what endianness is used, how stacks grow, sizes-layout and alignment of data types, exception propagation.
- In short, ABI is set of rules used to interact with particular architecture.

# Difference between ABI & API

- API : Interaction at source code level.
- ABI : Interaction at machine(binary) code level.

### Pre-requisites

- 

# Understading function stack frame(x86 architecture)

```
#include<stdio.h>

int func(int var)
{
        printf("var = %d\n", var);
        return var;
}

int main(){
        int a;
        a = 5;
        func(a);
        return 0;
}
```

```
$ gcc -O0 -g -o abi abi.c
$ gdb abi
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-80.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/vishal/musl-libc-1-1-14/abi...done.
(gdb) 
```

- `l` or `list` stands for list which shows source code list. You can use some variations like `list [function_name]`, `list [filename:function]`, etc.
```
(gdb) l
2
3       int func(int var)
4       {
5               printf("var = %d\n", var);
6               return var;
7       }
8
9       int main(){
10              int a;
11              a = 5;
(gdb) l
12              func(a);
13              return 0;
14      }
(gdb)
```

- setting break points

```
(gdb) b main
Breakpoint 1 at 0x40055c: file abi.c, line 11.
(gdb) run
Starting program: /home/vishal/musl-libc-1-1-14/abi

Breakpoint 1, main () at abi.c:11
11              a = 5;
```

- Now let’s use the `disassemble` command to show the assembly instructions for the current function. You can also pass a function name to disassemble to specify a different function to examine.

- Before going into depth we first understand basic register understanding of x86 architecture


# x86 Architecture Registers

```
x86 architecture registers

8 General-Purpose Registers (GPR), 
        RAX(or EAX) - Accumulator Register : arithmetic operations
        RCX(or RCX) - Counter register : shift/rotate instructions and loops
        RDX(or EDX) - Data register : arithmetic operations and I/O operations
        RBX(or EBX) - Base register : Used as a pointer to data (located in segment register DS, when in segmented mode)
        RSP(or ESP) - Stack Pointer register : Pointer to the top of the stack
        RBP(or EBP) - Base Pointer register : point to the base of the stack(AKA frame pointer)
        RSI(or ESI) - Source Index register : Used as a pointer to a source in stream operations
        RDI(or EDI) - Destination Index register : Used as a pointer to a destination in stream operations

prefix `R` & `E` is for 64-bit & 32-bit architecture. If there is not prefix & used as only two latter then register used as 16-bit width.

6 Segment Registers, 

    Stack Segment (SS). Pointer to the stack.
    Code Segment (CS). Pointer to the code.
    Data Segment (DS). Pointer to the data.
    Extra Segment (ES). Pointer to extra data ('E' stands for 'Extra').
    F Segment (FS). Pointer to more extra data ('F' comes after 'E').
    G Segment (GS). Pointer to still more extra data ('G' comes after 'F').


1 Flag Register 
	This register is 32/64-bit wide used to know the status of processor.

Instruction Pointer. 
	The EIP/RIP register contains the address of the next instruction to be executed if no branching is done. EIP/RIP can only be read through the stack after a call instruction.


x86_64 bit architecture specific 
    R0 = RAX.
    R1 = RCX.
    R2 = RDX.
    R3 = RBX.
    R4 = RSP.
    R5 = RBP.
    R6 = RSI.
    R7 = RDI.
    R8,R9,R10,R11,R12,R13,R14,R15 are the new registers and have no other names.
    R0D~R15D = lowermost 32 bits of each register. For example, R0D is EAX.
    R0W~R15W = lowermost 16 bits of each register. For example, R0W is AX.
    R0L~R15L = lowermost 8 bits of each register. For example, R0L is AL.

```


```
(gdb) disassemble
Dump of assembler code for function main:
   0x0000000000400554 <+0>:     push   %rbp
   0x0000000000400555 <+1>:     mov    %rsp,%rbp
   0x0000000000400558 <+4>:     sub    $0x10,%rsp
=> 0x000000000040055c <+8>:     movl   $0x5,-0x4(%rbp)
   0x0000000000400563 <+15>:    mov    -0x4(%rbp),%eax
   0x0000000000400566 <+18>:    mov    %eax,%edi
   0x0000000000400568 <+20>:    callq  0x400530 <func>
   0x000000000040056d <+25>:    mov    $0x0,%eax
   0x0000000000400572 <+30>:    leaveq
   0x0000000000400573 <+31>:    retq
End of assembler dump.
(gdb)
```

- first two lines is called function prologue, which stores the base pointer in stack & update new basepointer with next frame address. There is special area called callee save registers in stack frame to store these kind of special purpose registers.
- third line is creating space of new frame by subtracting 10 from stack pointer.
- fourth line will move value `5` to variable `a` which is stored at location 4 byte down to base pointer.
- Now `mov    -0x4(%rbp),%eax` will move the value stored in variable `a` to eax.
- Line `mov    %eax,%edi` & `callq  0x400530 <func>` will call the function `func`.
- The `leaveq` instruction copies the base pointer(%rbp) into the stack pointer(%rsp), which releases the stack space allocated to the stack frame. The old base pointer is then popped from the stack into the %rbp register, restoring the calling procedure's stack frame.


In Process
