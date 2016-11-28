To understanding function stack frame & Application Binary Interface(ABI), first we summing up [Wikipedia](https://en.wikipedia.org/wiki/Application_binary_interface) definition of ABI & later goes to understand stack frame. 
According to wikipedia, an application binary interface (ABI) is the interface between two program modules, one of which is often a library or operating system, at the level of machine code.
Above definition seems good but cant give me confidence to explain others in simple words. So i dig deep to understand that practically & see what i found.

# What is ABI ?
- ABI is protocol followed by compiler, OS or library writer which defines the mechanisms by which functions are invoked, how parameters are passed to function, how return values are retrived, how a process invokes system calls, etc.
- ABI also defines what endianness is used, how stacks grow, sizes-layout and alignment of data types, exception propagation.
- In short, ABI is set of rules used to interact with particular architecture.

# Difference between ABI & API

- API : Interaction at source code level.
- ABI : Interaction at machine(binary) code level.


# Understading function stack frame with ABI conventions for x86 architecture

