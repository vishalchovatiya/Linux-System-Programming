
> ```ln -sf [target] [source]```

Create Soft & Hard Links

*Soft Links(-s Parameter)* : point to the file name

*Hard Links(By Default)*   : point to the file contents(share the same inode)

> ```nm -n [ELF_file]```

-n option sort symbols by address

*Symbol Table Information*
<VirtualAddress>        <SymbolType>    <SymbolName>
0000000000601038        B                __bss_start

SymbolType : Lower case = local & Upper case = external

> ```ar -t [libname.a]```

Get the list of Object include in static lib

> ```ldd [libname.so]  OR ldd [Executable]```

Get the list of shared Object used
