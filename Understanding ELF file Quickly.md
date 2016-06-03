Command Format   : `readelf -a [Executable] > readELF.txt`

Command Executed : `readelf -a testMuslShared > readELF.txt`

This file name : readELF.txt

## Keyword Understanding

> Shared Library(.so)
Combination of multiple objects, Single Copy loaded in memory shared by multiple process(that's why shared object)

> Sections
Link time info

> Segments 
Run time info

> Relocation Records 
Contain object files's references, Linker uses the relocation records to find all of the addresses that need to be filled in process image.

> Symbol Table : Two Meaning

1. 
- An object/executable files will contain a symbol table of identifiers of function & variables with addresses. 
- Linker will use these symbol tables to resolve any unresolved references during linking.
- A symbol table may only exist during the translation process, or it may be embedded in the output of that process for later exploitation
2. 
- There's also the symbol table in a shared library or DLL. 
- This is produced by the linker & serves to name all the functions and data items that are visible to users of the library. 
- This allows the system to do run-time linking, resolving open references to those names to the location where the library is loaded in memory.

**Note**: While reverse engineering an executable, many tools refer to the symbol table to check what addresses have been assigned to global variables and known functions. If the symbol table has been stripped or cleaned out before being converted into an executable, tools will find it harder to determine addresses or understand anything about the program.

> .dynamic
The structure residing at the beginning of the section holds the addresses of other dynamic linking information.

> Procedure Linkage Table(.plt)

*Procedure Linkage Table stores indirect links into the GoT*
- is Table of addresses resides in text segment
- used to store address of all function/procedure needed runtime (address not known at the time of linking)
- The PLT uses what is called lazy resolution. Means it resolves procedure address when it really needs
*How PLT works* -
  1. A function func is called and the compiler translates this to a call to func@plt.
  2. The program jumps to the PLT. The PLT points to the GOT. If the function hasnâ€™t been previously called, the GOT points back into the PLT to a resolver routine, otherwise it points to the function itself.
  3. If the function hasnâ€™t been previously called, PLT resolve routine & update the GOT entry with actual address of the function.


> Global Offset Table(.got)

- is Table of addresses resides in data segment.
- If some instruction in text segment, wants to refer to a variable it must normally use an absolute memory address. 
- Instead of referring to the absolute memory address, it refers to the GOT, whose location is known. 
- By GOT we can relocate references needed by text segment


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
