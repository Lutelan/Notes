- Overall at the high level most executable binaries even windows PE files have the following structure
![[Executable.png]]
- Loading and running a binary is a complex process and the way a binary is on disk is not similar to the way it is in virtual memory it is collapsed to make storage more efficient.
- When you run a binary the operating system sets up a new process[[Windows Processes]], for it and a [[Virtual Memory]] or Virtual address space for it.
- The OS maps a _interpreter_ into the processes virtual memory. The _interpreter_ is a user space program which knows how to load a binary and perform relocation.
- It is `ld-linux.so` on Linux and `ntdll.dll` on windows.
- Linux binaries have an `.interp` section that specifies the path to the interpreter that is to be used to load the binary,
```
$ readelf -p .interp a.out

String dump of section '.interp':
  [     0]  /lib64/ld-linux-x86-64.so.2
```
- The interpreter loads the binary into virtual address space, it parses the file to find out which dynamic libraries the file uses and maps these into the virtual address space of the process using something like `mmap`, and fills in the addresses for the library functions.
- However in reality the library addresses are only resolved when required and this is known as _lazy binding_
- After this is done, the interpreter looks up the entry point of the binary and transfers control to it, beginning the execution of the binary.