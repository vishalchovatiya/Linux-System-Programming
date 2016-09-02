Command Format   : `readelf -a [Executable] > readELF.txt`

### Intro to ELF 

- ELF is the file format used for object files (.o's), binaries, shared libraries and core dumps in Linux.
- ELF has the same layout for all architectures, however endianness and word size can differ; relocation types, symbol types and the like may have platform-specific values, and of course the contained code is arch specific.
- An ELF file provides 2 views on the data it contains: A linking view and an execution view. Those two views can be accessed by two headers: Section header table & Program header.

### Keyword Understanding

> **ELF Header**

- Neither the `Section Header` nor the `Program Header` have fixed positions, they can be located anywhere in an ELF file. To find them the ELF header is used, which is located at the very start of the file.
- The first bytes contain the elf magic "\x7fELF", followed by the class ID (32 or 64 bit ELF file), the data format ID (little endian/big endian), the machine type, etc.
```
greek0@iphigenie:~$ readelf -h /bin/bash
    ELF Header:
      Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
      Class:                             ELF32
      Data:                              2's complement, little endian
      Version:                           1 (current)
      OS/ABI:                            UNIX - System V
      ABI Version:                       0
      Type:                              EXEC (Executable file)
      Machine:                           Intel 80386
      Version:                           0x1
      Entry point address:               0x805be30
      Start of program headers:          52 (bytes into file)
      Start of section headers:          675344 (bytes into file)
      Flags:                             0x0
      Size of this header:               52
      Size of program headers:           32
      Number of program headers:         8
      Size of section headers:           40
      Number of section headers:         26
      Section header string table index: 25
```


> **Section Headers OR Section Header Table**

- Gives an complete overview on the sections contained in the ELF file
```
 greek0@iphigenie:~$ readelf -S /bin/bash
 There are 26 section headers, starting at offset 0xa4e10:
 Section Headers:
   [Nr] Name             Type        Addr      Off   Size   ES Flg Lk Inf Al
   [ 0]                  NULL        00000000 00000 000000 00      0   0  0
   [ 1] .interp          PROGBITS    08048134 00134 000013 00   A  0   0  1
   [ 2] .note.ABI-tag    NOTE        08048148 00148 000020 00   A  0   0  4
   [ 3] .hash            HASH        08048168 00168 002e48 04   A  4   0  4
   [ 4] .dynsym          DYNSYM      0804afb0 02fb0 007890 10   A  5   1  4
   [ 5] .dynstr          STRTAB      08052840 0a840 0074e2 00   A  0   0  1
   [ 6] .gnu.version     VERSYM      08059d22 11d22 000f12 02   A  4   0  2
   [ 7] .gnu.version_r   VERNEED     0805ac34 12c34 000090 00   A  5   2  4
   [ 8] .rel.dyn         REL         0805acc4 12cc4 000040 08   A  4   0  4
   [ 9] .rel.plt         REL         0805ad04 12d04 0005a8 08   A  4  11  4
   [10] .init            PROGBITS    0805b2ac 132ac 000017 00  AX  0   0  4
   [11] .plt             PROGBITS    0805b2c4 132c4 000b60 04  AX  0   0  4
   [12] .text            PROGBITS    0805be30 13e30 077154 00  AX  0   0 16
   [13] .fini            PROGBITS    080d2f84 8af84 00001a 00  AX  0   0  4
   [14] .rodata          PROGBITS    080d2fa0 8afa0 015198 00   A  0   0 32
   [15] .eh_frame_hdr    PROGBITS    080e8138 a0138 00002c 00   A  0   0  4
   [16] .eh_frame        PROGBITS    080e8164 a0164 00009c 00   A  0   0  4
   [17] .ctors           PROGBITS    080e9200 a0200 000008 00  WA  0   0  4
   [18] .dtors           PROGBITS    080e9208 a0208 000008 00  WA  0   0  4
   [19] .jcr             PROGBITS    080e9210 a0210 000004 00  WA  0   0  4
   [20] .dynamic         DYNAMIC     080e9214 a0214 0000d8 08  WA  5   0  4
   [21] .got             PROGBITS    080e92ec a02ec 000004 04  WA  0   0  4
   [22] .got.plt         PROGBITS    080e92f0 a02f0 0002e0 04  WA  0   0  4
   [23] .data            PROGBITS    080e95e0 a05e0 004764 00  WA  0   0 32
   [24] .bss             NOBITS      080edd60 a4d44 004bc8 00  WA  0   0 32
   [25] .shstrtab        STRTAB      00000000 a4d44 0000cc 00      0   0  1
```

> **Program Headers OR Program Headers Table**

- Contains information for the kernel on how to start the program
- The `LOAD` directives determinate what parts of the ELF file get mapped into memory. 
- The `INTERP` directive specifies an ELF interpreter, which is normally `/lib/ld-linux.so.2` on Linux systems.
- The `DYNAMIC` entry points to the `.dynamic` section which contains information used by the ELF interpreter to setup the binary.

```
greek0@iphigenie:~$ readelf -l /bin/bash
Elf file type is EXEC (Executable file)
Entry point 0x805be30
There are 8 program headers, starting at offset 52

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  PHDR           0x000034 0x08048034 0x08048034 0x00100 0x00100 R E 0x4
  INTERP         0x000134 0x08048134 0x08048134 0x00013 0x00013 R   0x1
      [Requesting program interpreter: /lib/ld-linux.so.2]
  LOAD           0x000000 0x08048000 0x08048000 0xa0200 0xa0200 R E 0x1000
  LOAD           0x0a0200 0x080e9200 0x080e9200 0x04b44 0x09728 RW  0x1000
  DYNAMIC        0x0a0214 0x080e9214 0x080e9214 0x000d8 0x000d8 RW  0x4
  NOTE           0x000148 0x08048148 0x08048148 0x00020 0x00020 R   0x4
  GNU_EH_FRAME   0x0a0138 0x080e8138 0x080e8138 0x0002c 0x0002c R   0x4
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x4

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .note.ABI-tag .dynsym .dynstr .gnu.version .gnu.version_r .rel.dyn .rel.plt ...
   03     .ctors .dtors .jcr .dynamic .got .got.plt .data .bss
   04     .dynamic
   05     .note.ABI-tag
   06     .eh_frame_hdr
   07
```

> **Shared Library(.so)**

- Combination of multiple objects files
- Single Copy loaded in memory shared by multiple process(that's why shared object)

> **Sections**

- Sections contain information needed during linking time

> **Segments**

- Segments contain information needed at run time

> **Relocation Table OR Relocation Section** 

- Relocation is the process of connecting symbolic references(functions,variable,etc names) with symbolic definitions(function, variable,etc definitions). 
- For example, when a program calls a function(at runtime), the associated call instruction must transfer control to the proper destination address at execution. 

```
    greek0@iphigenie:~$ readelf -r /bin/bash

    Relocation section '.rel.dyn' at offset 0x12cc4 contains 8 entries:
     Offset     Info    Type            Sym.Value  Sym. Name
    080e92ec  00078006 R_386_GLOB_DAT    00000000   __gmon_start__
    080edd68  00035205 R_386_COPY        080edd68   stdout
    080edd6c  00035d05 R_386_COPY        080edd6c   stderr
    080edd70  00046405 R_386_COPY        080edd70   PC
    080edd74  00067405 R_386_COPY        080edd74   stdin
    080edd78  0006e305 R_386_COPY        080edd78   UP

    Relocation section '.rel.plt' at offset 0x12d04 contains 181 entries:
     Offset     Info    Type            Sym.Value  Sym. Name
    080e9368  00012c07 R_386_JUMP_SLOT   00000000   fileno
    080e936c  00013807 R_386_JUMP_SLOT   00000000   strcmp
    080e9370  00014107 R_386_JUMP_SLOT   0805b4a4   close
    080e9374  00015307 R_386_JUMP_SLOT   00000000   dlsym
    080e937c  00016a07 R_386_JUMP_SLOT   00000000   fprintf
    080e9388  00018307 R_386_JUMP_SLOT   00000000   fflush
    080e9390  00019c07 R_386_JUMP_SLOT   0805b524   unlink
    080e930c  00003307 R_386_JUMP_SLOT   00000000   regexec
    080e9328  00007a07 R_386_JUMP_SLOT   00000000   ferror
    080e9330  00008307 R_386_JUMP_SLOT   00000000   readdir64
    080e9334  00008f07 R_386_JUMP_SLOT   00000000   strchr
    080e9338  0000a507 R_386_JUMP_SLOT   00000000   fdopen
    080e9344  0000da07 R_386_JUMP_SLOT   00000000   getpid
    080e9360  00012207 R_386_JUMP_SLOT   00000000   write
    080e95cc  00078707 R_386_JUMP_SLOT   00000000   strcpy
    ...
    ...
````

> **Symbol Table : Two Use Case**

1. Symbol table in object/executable files will contain symbolic name of functions & variables with addresses which is used by Linker to resolve any unresolved references during linking. 
2. There's also the symbol table in a shared library/DLL produced by the linker(at compile time) which is used by dynamic linker to do run-time linking & resolving open references to those names to the location where the library is loaded in memory.

**Note**: While reverse engineering an executable, many tools refer to the symbol table to check what addresses have been assigned to global variables and known functions. If the symbol table has been stripped or cleaned out before being converted into an executable, tools will find it harder to determine addresses or understand anything about the program.

> **Dynamic Section & Dynamic linking with the ELF interpreter**

- First the dynamic linker (contained within the interpreter) looks at the `Dynamic section`, whose address is stored in the Program Header.
- There it finds the `NEEDED` entries determining which libraries have to be loaded before the program can be run, the `REL` entries giving the address of the relocation tables, the `VER` entries which contain symbol versioning information, etc.
- So the dynamic linker loads the needed libraries and performs relocations (either directly at program startup or later(lazy resolution)).
- Finally control is transferred to the address given by the symbol `_start` in the binary. Normally some `gcc/glibc` startup code lives there, which in the end calls `main()`.
```
    greek0@iphigenie:~$ readelf -d /bin/bash
    Dynamic section at offset 0xa0214 contains 22 entries:
      Tag        Type                         Name/Value
     0x00000001 (NEEDED)                     Shared library: [libncurses.so.5]
     0x00000001 (NEEDED)                     Shared library: [libdl.so.2]
     0x00000001 (NEEDED)                     Shared library: [libc.so.6]
     0x0000000a (STRSZ)                      29922 (bytes)
     0x0000000b (SYMENT)                     16 (bytes)
     0x00000003 (PLTGOT)                     0x80e92f0
     0x00000002 (PLTRELSZ)                   1448 (bytes)
     0x00000014 (PLTREL)                     REL
     0x00000017 (JMPREL)                     0x805ad04
     0x00000011 (REL)                        0x805acc4
     0x00000012 (RELSZ)                      64 (bytes)
     0x6ffffffe (VERNEED)                    0x805ac34
     0x6fffffff (VERNEEDNUM)                 2
     0x6ffffff0 (VERSYM)                     0x8059d22
     0x00000000 (NULL)                       0x0
```

> **Procedure Linkage Table(.plt)**

- is Table of addresses *resides in text segment* used to store address of all functions needed at runtime (address not known at the time of linking)
- The PLT uses what is called lazy resolution. Means it resolves procedure address once when it calls method
- *How PLT works* -
  1. A function func is called and the compiler translates this to a call to func@plt.
  2. The program jumps to the PLT. The PLT points to the GOT. If the function has not been previously called, the GOT points back into the PLT to a resolver routine, otherwise it points to the function itself.
  3. If the function has not been previously called, PLT resolve routine & update the GOT entry with actual address of the function.


> **Global Offset Table(.got)**

- is Table of addresses *resides in data segment* used to store relative address of variables & produres which is mapped with absolute address
- *How PLT works* -
  1. If some instruction in text segment, wants to refer to a variable it must normally use an absolute memory address.
  2. Instead of referring to the absolute memory address, it refers to the GOT, whose location is known. 

**NOTE** : By this kind of address tables we can effectively use relocating of objects, with just updating entry each time relocation performed

> **Program loading in the kernel**

- The execution of a program starts inside the kernel, in the exec system call. There the file type is looked up and the appropriate handler is called. 
- The `binfmt-elf` handler then loads the `ELF header` and the `Program Header`, followed by lots of sanity checks.
- The kernel then loads the parts specified in the `LOAD` directives in the `Program Header` into memory. If an `INTERP` entry is present, the interpreter is loaded too. 
- Statically linked binaries can do without an interpreter; dynamically linked programs always need `/lib/ld-linux.so` as interpreter because it includes some startup code, loads shared libraries needed by the binary, and performs relocations.

Finally control can be transfered to the program, to the interpreter, if present, otherwise to the binary itself.
```
    greek0@iphigenie:~$ readelf -l /bin/bash
    Program Headers:
      Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
      PHDR           0x000034 0x08048034 0x08048034 0x00100 0x00100 R E 0x4
      INTERP         0x000134 0x08048134 0x08048134 0x00013 0x00013 R   0x1
          [Requesting program interpreter: /lib/ld-linux.so.2]
      LOAD           0x000000 0x08048000 0x08048000 0xa0200 0xa0200 R E 0x1000
      LOAD           0x0a0200 0x080e9200 0x080e9200 0x04b44 0x09728 RW  0x1000
      DYNAMIC        0x0a0214 0x080e9214 0x080e9214 0x000d8 0x000d8 RW  0x4
      GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x4
    ...
```
