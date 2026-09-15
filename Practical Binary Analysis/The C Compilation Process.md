- _Compilation_ is the process of translating human readable source code into machine code which processors can execute.
- The C Compilation process looks as follows
![[C Compilation Process.png]]
- Compilation consists of 4 phases, _preprocessing_, _compilation_, _assembly_, and _linking_.

###### The Preprocessing Phase
- C files contain macros(using `#define`), and `#include` directives on which the source file depends.
- The preprocessing stage expands any `#define` or `#include` directives in the source file so all that is left is pure C code ready to compile.
- We can observe the preprocessed state of a C file, using the following gcc command
```
gcc -E -P <file.c>
```

##### The Compilation Phase
- The compilation process takes the preprocessed code and converts it to assembly language, to look at the assembly one can use the following command
```
gcc -S -masm=intel shitty_hellow.c
```
##### The Assembly Phase
- The assembly phase takes in the assembly language files and outputs a set of _object files_ also called _modules_.
- Object files contain machine instructions that are in principle executable by the processor but we have some more steps.
- Look at the object files as follows
```
$ gcc -c file.c
$ file file.o
compilation_example.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```
- We notice the word _relocatable_ what does that mean?
- Relocatable files don’t rely on being placed at any particular address in memory; rather, they can be moved around at will without this breaking any assumptions in the code.
- Object files are compiled independently from each other, so the assembler has no way of knowing the memory addresses of other object files when assembling an object file. 

##### The Linking Phase
- The Linker combines all the object files into a single binary executable.
- Since object files are relocatable they contain references to external functions and variables.
- Before linking the addresses at which these referenced functions and variables will be placed is not known, the object files only contain _relocation symbols_, that specify how the functions should be resolved.
- The linker’s job is to take all the object files belonging to a program and merge them into a single coherent executable.
The output of a file linked binary on which file is ran looks like this
```
a.out: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=25d8444a5ef127269e3553af833637179fbcf88e, for GNU/Linux 3.2.0, not stripped
```
> Note: The "interpreter" is what resolves the dynamically linked libraries when the program is ran.
