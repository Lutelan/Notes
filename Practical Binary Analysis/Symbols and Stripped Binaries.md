- High level source code such as C code has human readable function and variable names.
- When compiler a compiler emits _symbols_ which correspond to functions and variables which help in debugging a executable.
- To read symbols use a tool like `readelf`, reading a compiled executable using `readelf` gives the following output
```
$ readelf --syms a.out
Symbol table '.dynsym' contains 7 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _[...]@GLIBC_2.34 (2)
     2: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterT[...]
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.2.5 (3)
     4: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
     6: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND [...]@GLIBC_2.2.5 (3)

Symbol table '.symtab' contains 36 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS Scrt1.o
     2: 000000000000037c    32 OBJECT  LOCAL  DEFAULT    4 __abi_tag
----------------------snip--------------------------------------------
    28: 0000000000004020     0 NOTYPE  GLOBAL DEFAULT   26 _end
    29: 0000000000001050    34 FUNC    GLOBAL DEFAULT   15 _start
    30: 0000000000004018     0 NOTYPE  GLOBAL DEFAULT   26 __bss_start
    31: 0000000000001139    26 FUNC    GLOBAL DEFAULT   15 main
    32: 0000000000004018     0 OBJECT  GLOBAL HIDDEN    25 __TMC_END__
    33: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
    34: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize@G[...]
    35: 0000000000001000     0 FUNC    GLOBAL HIDDEN    12 _init
```
- We will look at understanding all the symbols later however on line 31 you see the `main` function and its address, it is the `main` function we defined
- Symbolic functions emitted by compiler vary greatly, for ELF binaries symbols are emitted in the DWARF format, windows PE binaries use the proprietary PDB(Microsoft portable debugging) format. 
- However in most situations such as malware or proprietary binaries things are stripped to prevent reverse engineering
###### Stripping a Binary
- We can strip binaries using the `strip` command, we can confirm this using the `file` command.
```
$ strip --strip-all a.out
$ file a.out
a.out: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=16a96bacbce664c74b79b822f2f801caf9a10aab, for GNU/Linux 3.2.0, stripped
```
- Running `readelf` on this does not give us much information
```
$ readelf --syms a.out

Symbol table '.dynsym' contains 7 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _[...]@GLIBC_2.34 (2)
     2: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterT[...]
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.2.5 (3)
     4: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
     6: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND [...]@GLIBC_2.2.5 (3)
```
- Most of the symbols have been removed the ones left are for dynamic `libc` functions which are of no use.