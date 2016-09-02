Command Format   : `readelf -a [Executable] > readELF.txt`

### Intro to ELF 

- ELF is the file format used for object files (.o's), binaries, shared libraries and core dumps in Linux.
- ELF has the same layout for all architectures, however endianness and word size can differ; relocation types, symbol types and the like may have platform-specific values, and of course the contained code is arch specific.
- An ELF file provides 2 views on the data it contains: A linking view and an execution view. Those two views can be accessed by two headers: the section header table and the program header table.

### Keyword Understanding

> **ELF Header**

- Neither the `Section Header` nor the `Program Header` have fixed positions, they can be located anywhere in an ELF file. To find them the ELF header is used, which is located at the very start of the file.
- The first bytes contain the elf magic "\x7fELF", followed by the class ID (32 or 64 bit ELF file), the data format ID (little endian/big endian), the machine type, etc.
- At the end of the ELF header are then pointers to the SHT and PHT.
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
- The `DYNAMIC` entry points to the .dynamic section which contains information used by the ELF interpreter to setup the binary.

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

> **Symbol Table : Two Meaning**

1. Symbol table in object/executable files will contain symbolic name of functions & variables with addresses which is used by Linker to resolve any unresolved references during linking. 
2. There's also the symbol table in a shared library/DLL produced by the linker(at compile time) which is used by dynamic linker to do run-time linking & resolving open references to those names to the location where the library is loaded in memory.

**Note**: While reverse engineering an executable, many tools refer to the symbol table to check what addresses have been assigned to global variables and known functions. If the symbol table has been stripped or cleaned out before being converted into an executable, tools will find it harder to determine addresses or understand anything about the program.

- When searching for a symbol the dynamic linker looks through the dynamic symbol table `.dynsym`, so all symbols present there are usable by other programs.
```
Symbol table '.dynsym' contains 7 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.2.5 (2)
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fclose@GLIBC_2.2.5 (2)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main@GLIBC_2.2.5 (2)
     4: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fgets@GLIBC_2.2.5 (2)
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
     6: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fopen@GLIBC_2.2.5 (2)

Symbol table '.symtab' contains 68 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000400238     0 SECTION LOCAL  DEFAULT    1
     2: 0000000000400254     0 SECTION LOCAL  DEFAULT    2
     3: 0000000000400274     0 SECTION LOCAL  DEFAULT    3
     4: 0000000000400298     0 SECTION LOCAL  DEFAULT    4
     5: 00000000004002b8     0 SECTION LOCAL  DEFAULT    5
     6: 0000000000400360     0 SECTION LOCAL  DEFAULT    6
     7: 00000000004003b0     0 SECTION LOCAL  DEFAULT    7
....
....
```

> **Dynamic Section**

- 

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

====================== Output Starts ====================================
```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x400510
  Start of program headers:          64 (bytes into file)
  Start of section headers:          4360 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         7
  Size of section headers:           64 (bytes)
  Number of section headers:         22
  Section header string table index: 19
```

```
Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         00000000004001c8  000001c8
       0000000000000034  0000000000000000   A       0     0     1
  [ 2] .hash             HASH             0000000000400200  00000200
       000000000000004c  0000000000000004   A       3     0     8
  [ 3] .dynsym           DYNSYM           0000000000400250  00000250
       0000000000000150  0000000000000018   A       4     1     8
  [ 4] .dynstr           STRTAB           00000000004003a0  000003a0
       00000000000000b5  0000000000000000   A       0     0     1
  [ 5] .rela.plt         RELA             0000000000400458  00000458
       0000000000000060  0000000000000018   A       3     7     8
  [ 6] .init             PROGBITS         00000000004004b8  000004b8
       0000000000000003  0000000000000000  AX       0     0     1
  [ 7] .plt              PROGBITS         00000000004004c0  000004c0
       0000000000000050  0000000000000010  AX       0     0     16
  [ 8] .text             PROGBITS         0000000000400510  00000510
       000000000000011c  0000000000000000  AX       0     0     16
  [ 9] .fini             PROGBITS         000000000040062c  0000062c
       0000000000000003  0000000000000000  AX       0     0     1
  [10] .rodata           PROGBITS         0000000000400630  00000630
       0000000000000013  0000000000000000   A       0     0     8
  [11] .eh_frame         PROGBITS         0000000000400648  00000648
       0000000000000064  0000000000000000   A       0     0     8
  [12] .init_array       INIT_ARRAY       0000000000600e58  00000e58
       0000000000000008  0000000000000000  WA       0     0     8
  [13] .fini_array       FINI_ARRAY       0000000000600e60  00000e60
       0000000000000008  0000000000000000  WA       0     0     8
  [14] .jcr              PROGBITS         0000000000600e68  00000e68
       0000000000000008  0000000000000000  WA       0     0     8
  [15] .dynamic                    0000000000600e70  00000e70
       0000000000000190  0000000000000010  WA       4     0     8
  [16] .got.plt          PROGBITS         0000000000601000  00001000
       0000000000000038  0000000000000008  WA       0     0     8
  [17] .bss              NOBITS           0000000000601038  00001038
       0000000000000008  0000000000000000  WA       0     0     4
  [18] .comment          PROGBITS         0000000000000000  00001038
       000000000000002c  0000000000000001  MS       0     0     1
  [19] .shstrtab         STRTAB           0000000000000000  00001064
       00000000000000a4  0000000000000000           0     0     1
  [20] .symtab           SYMTAB           0000000000000000  00001688
       00000000000004c8  0000000000000018          21    35     8
  [21] .strtab           STRTAB           0000000000000000  00001b50
       00000000000001b0  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), l (large)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)

There are no section groups in this file.
```

```
Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040
                 0x0000000000000188 0x0000000000000188  R E    8
  INTERP         0x00000000000001c8 0x00000000004001c8 0x00000000004001c8
                 0x0000000000000034 0x0000000000000034  R      1
      [Requesting program interpreter: /home/vishal/workspace/musl/lib/ld-musl-x86_64.so.1]
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x00000000000006ac 0x00000000000006ac  R E    200000
  LOAD           0x0000000000000e58 0x0000000000600e58 0x0000000000600e58
                 0x00000000000001e0 0x00000000000001e8  RW     200000
          0x0000000000000e70 0x0000000000600e70 0x0000000000600e70
                 0x0000000000000190 0x0000000000000190  RW     8
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     10
  GNU_RELRO      0x0000000000000e58 0x0000000000600e58 0x0000000000600e58
                 0x00000000000001a8 0x00000000000001a8  R      1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp 
   02     .interp .hash .dynsym .dynstr .rela.plt .init .plt .text .fini .rodata .eh_frame 
   03     .init_array .fini_array .jcr .dynamic .got.plt .bss 
   04     .dynamic 
   05     
   06     .init_array .fini_array .jcr .dynamic 

```

```
Dynamic section at offset 0xe70 contains 20 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [./libSharedMusl.so]
 0x0000000000000001 (NEEDED)             Shared library: [./lib/libc.so]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so]
 0x000000000000000c (INIT)               0x4004b8
 0x000000000000000d (FINI)               0x40062c
 0x0000000000000019 (INIT_ARRAY)         0x600e58
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x600e60
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x0000000000000004 (HASH)               0x400200
 0x0000000000000005 (STRTAB)             0x4003a0
 0x0000000000000006 (SYMTAB)             0x400250
 0x000000000000000a (STRSZ)              181 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x601000
 0x0000000000000002 (PLTRELSZ)           96 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x400458
 0x0000000000000000 (NULL)               0x0

```

```
Relocation section '.rela.plt' at offset 0x458 contains 4 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000601018  000100000007 R_X86_64_JUMP_SLO 0000000000000000 puts + 0
000000601020  000800000007 R_X86_64_JUMP_SLO 0000000000000000 print1 + 0
000000601028  000a00000007 R_X86_64_JUMP_SLO 0000000000000000 print2 + 0
000000601030  000c00000007 R_X86_64_JUMP_SLO 0000000000000000 __libc_start_main + 0

The decoding of unwind sections for machine type Advanced Micro Devices X86-64 is not currently supported.

```

```
Symbol table '.dynsym' contains 14 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts
     2: 00000000004004b8     0 NOTYPE  GLOBAL DEFAULT    6 _init
     3: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
     4: 0000000000400510     0 NOTYPE  GLOBAL DEFAULT    8 _start
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
     6: 0000000000601038     0 NOTYPE  GLOBAL DEFAULT   17 __bss_start
     7: 000000000040062c     0 NOTYPE  GLOBAL DEFAULT    9 _fini
     8: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND print1
     9: 0000000000601038     0 NOTYPE  GLOBAL DEFAULT   16 _edata
    10: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND print2
    11: 0000000000601040     0 NOTYPE  GLOBAL DEFAULT   17 _end
    12: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main
    13: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _Jv_RegisterClasses

Symbol table '.symtab' contains 51 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 00000000004001c8     0 SECTION LOCAL  DEFAULT    1 
     2: 0000000000400200     0 SECTION LOCAL  DEFAULT    2 
     3: 0000000000400250     0 SECTION LOCAL  DEFAULT    3 
     4: 00000000004003a0     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000400458     0 SECTION LOCAL  DEFAULT    5 
     6: 00000000004004b8     0 SECTION LOCAL  DEFAULT    6 
     7: 00000000004004c0     0 SECTION LOCAL  DEFAULT    7 
     8: 0000000000400510     0 SECTION LOCAL  DEFAULT    8 
     9: 000000000040062c     0 SECTION LOCAL  DEFAULT    9 
    10: 0000000000400630     0 SECTION LOCAL  DEFAULT   10 
    11: 0000000000400648     0 SECTION LOCAL  DEFAULT   11 
    12: 0000000000600e58     0 SECTION LOCAL  DEFAULT   12 
    13: 0000000000600e60     0 SECTION LOCAL  DEFAULT   13 
    14: 0000000000600e68     0 SECTION LOCAL  DEFAULT   14 
    15: 0000000000600e70     0 SECTION LOCAL  DEFAULT   15 
    16: 0000000000601000     0 SECTION LOCAL  DEFAULT   16 
    17: 0000000000601038     0 SECTION LOCAL  DEFAULT   17 
    18: 0000000000000000     0 SECTION LOCAL  DEFAULT   18 
    19: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    20: 0000000000600e68     0 OBJECT  LOCAL  DEFAULT   14 __JCR_LIST__
    21: 0000000000400540     0 FUNC    LOCAL  DEFAULT    8 deregister_tm_clones
    22: 0000000000400570     0 FUNC    LOCAL  DEFAULT    8 register_tm_clones
    23: 00000000004005b0     0 FUNC    LOCAL  DEFAULT    8 __do_global_dtors_aux
    24: 0000000000601038     1 OBJECT  LOCAL  DEFAULT   17 completed.6337
    25: 0000000000600e60     0 OBJECT  LOCAL  DEFAULT   13 __do_global_dtors_aux_fin
    26: 00000000004005d0     0 FUNC    LOCAL  DEFAULT    8 frame_dummy
    27: 0000000000600e58     0 OBJECT  LOCAL  DEFAULT   12 __frame_dummy_init_array_
    28: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS test.c
    29: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    30: 00000000004006a8     0 OBJECT  LOCAL  DEFAULT   11 __FRAME_END__
    31: 0000000000600e68     0 OBJECT  LOCAL  DEFAULT   14 __JCR_END__
    32: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS 
    33: 0000000000600e70     0 OBJECT  LOCAL  DEFAULT   15 _
    34: 0000000000601000     0 OBJECT  LOCAL  DEFAULT   16 _GLOBAL_OFFSET_TABLE_
    35: 0000000000601038     0 OBJECT  GLOBAL HIDDEN    16 __TMC_END__
    36: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts
    37: 0000000000400630     0 OBJECT  GLOBAL HIDDEN    10 __dso_handle
    38: 00000000004004b8     0 NOTYPE  GLOBAL DEFAULT    6 _init
    39: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
    40: 0000000000400510     0 NOTYPE  GLOBAL DEFAULT    8 _start
    41: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
    42: 0000000000601038     0 NOTYPE  GLOBAL DEFAULT   17 __bss_start
    43: 0000000000400600    41 FUNC    GLOBAL DEFAULT    8 main
    44: 000000000040062c     0 NOTYPE  GLOBAL DEFAULT    9 _fini
    45: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND print1
    46: 0000000000601038     0 NOTYPE  GLOBAL DEFAULT   16 _edata
    47: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND print2
    48: 0000000000601040     0 NOTYPE  GLOBAL DEFAULT   17 _end
    49: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main
    50: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _Jv_RegisterClasses

```

```
Histogram for bucket list length (total of 3 buckets):
 Length  Number     % of total  Coverage
      0  0          (  0.0%)
      1  0          (  0.0%)      0.0%
      2  1          ( 33.3%)     15.4%
      3  0          (  0.0%)     15.4%
      4  1          ( 33.3%)     46.2%
      5  0          (  0.0%)     46.2%
      6  0          (  0.0%)     46.2%
      7  1          ( 33.3%)    100.0%

No version information found in this file.
```
