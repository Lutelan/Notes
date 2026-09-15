###### Looking inside a Object File
- The code for the compiled file is in this example is 
```
#include <stdio.h>

#define FORMAT_STRING "%s"
#define MESSAGE       "Hello, world!\n"

int
main(int argc, char *argv[]) {
  printf(FORMAT_STRING, MESSAGE);
  return 0;
}
```
- To only obtain the object files use the following command
```
$ gcc -c main.c
```
- To obtain the data in the `.rodata` section do this [[ELF Linker Sections]]
```
$ objdump -sj. rodata compilation_example.o

compilation_example.o:     file format elf64-x86-64

Contents of section .rodata:
 0000 48656c6c 6f2c2077 6f726c64 2100      Hello, world!. 
```
- Disassemble the object file in intel assembler syntax as follows
```
$ objdump -M intel -d compilation_example.o

compilation_example.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 <main>:
   0:	55                   	push   rbp
   1:	48 89 e5             	mov    rbp,rsp
   4:	48 83 ec 10          	sub    rsp,0x10
   8:	89 7d fc             	mov    DWORD PTR [rbp-0x4],edi
   b:	48 89 75 f0          	mov    QWORD PTR [rbp-0x10],rsi
   f:	48 8d 05 00 00 00 00 	lea    rax,[rip+0x0]        # 16 <main+0x16>
  16:	48 89 c7             	mov    rdi,rax
  19:	e8 00 00 00 00       	call   1e <main+0x1e>
  1e:	b8 00 00 00 00       	mov    eax,0x0
  23:	c9                   	leave
  24:	c3                   	ret
```
- The first command gives information about the `.rodata` section which is the "read only data" section and contains strings.
- The thing to note is that the the "Hello, world!" string loaded into `rdi` at line 16 points to a weird location in the stack, and the call to the `printf` function is also at a offset after the main function at `0x1e`, which makes no sense.
- Why does the call to `printf/puts` point to in the middle of main, this is because data and code references from object files have no been resolved.
- The linker fills up this information in the object file.
- Looking at the relocation symbols in object file using `readelf`
```
$ readelf --relocs compilation_example.o

Relocation section '.rela.text' at offset 0x188 contains 2 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000000012  000300000002 R_X86_64_PC32     0000000000000000 .rodata - 4
00000000001a  000500000004 R_X86_64_PLT32    0000000000000000 puts - 4

Relocation section '.rela.eh_frame' at offset 0x1b8 contains 1 entry:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000000020  000200000002 R_X86_64_PC32     0000000000000000 .text + 0
```
- The symbols tell the linker to resolve the reference to the string to whatever address it ends up in `.rodata`.
- The next tells how to resolve the `puts` function in the code.
- You can see that the offsets are 4 bytes ahead in the offset at which they are called to account or opcode size.


###### Examining a Complete Binary Executable
- View the disassembly of the binary executable as follows:
- Disassembly
	```
	$ objdump -M intel -d a.out
	
	a.out:     file format elf64-x86-64
	
	
	Disassembly of section .init:
	
	0000000000001000 <_init>:
	    1000:	48 83 ec 08          	sub    rsp,0x8
	    1004:	48 8b 05 c5 2f 00 00 	mov    rax,QWORD PTR [rip+0x2fc5]        # 3fd0 <__gmon_start__@Base>
	    100b:	48 85 c0             	test   rax,rax
	    100e:	74 02                	je     1012 <_init+0x12>
	    1010:	ff d0                	call   rax
	    1012:	48 83 c4 08          	add    rsp,0x8
	    1016:	c3                   	ret
	
	Disassembly of section .plt:
	
	0000000000001020 <puts@plt-0x10>:
	    1020:	ff 35 ca 2f 00 00    	push   QWORD PTR [rip+0x2fca]        # 3ff0 <_GLOBAL_OFFSET_TABLE_+0x8>
	    1026:	ff 25 cc 2f 00 00    	jmp    QWORD PTR [rip+0x2fcc]        # 3ff8 <_GLOBAL_OFFSET_TABLE_+0x10>
	    102c:	0f 1f 40 00          	nop    DWORD PTR [rax+0x0]
	
	0000000000001030 <puts@plt>:
	    1030:	ff 25 ca 2f 00 00    	jmp    QWORD PTR [rip+0x2fca]        # 4000 <puts@GLIBC_2.2.5>
	    1036:	68 00 00 00 00       	push   0x0
	    103b:	e9 e0 ff ff ff       	jmp    1020 <_init+0x20>
	
	Disassembly of section .plt.got:
	
	0000000000001040 <__cxa_finalize@plt>:
	    1040:	ff 25 9a 2f 00 00    	jmp    QWORD PTR [rip+0x2f9a]        # 3fe0 <__cxa_finalize@GLIBC_2.2.5>
	    1046:	66 90                	xchg   ax,ax
	
	Disassembly of section .text:
	
	0000000000001050 <_start>:
	    1050:	31 ed                	xor    ebp,ebp
	    1052:	49 89 d1             	mov    r9,rdx
	    1055:	5e                   	pop    rsi
	    1056:	48 89 e2             	mov    rdx,rsp
	    1059:	48 83 e4 f0          	and    rsp,0xfffffffffffffff0
	    105d:	50                   	push   rax
	    105e:	54                   	push   rsp
	    105f:	45 31 c0             	xor    r8d,r8d
	    1062:	31 c9                	xor    ecx,ecx
	    1064:	48 8d 3d ce 00 00 00 	lea    rdi,[rip+0xce]        # 1139 <main>
	    106b:	ff 15 4f 2f 00 00    	call   QWORD PTR [rip+0x2f4f]        # 3fc0 <__libc_start_main@GLIBC_2.34>
	    1071:	f4                   	hlt
	    1072:	66 2e 0f 1f 84 00 00 	cs nop WORD PTR [rax+rax*1+0x0]
	    1079:	00 00 00 
	    107c:	0f 1f 40 00          	nop    DWORD PTR [rax+0x0]
	
	0000000000001080 <deregister_tm_clones>:
	    1080:	48 8d 3d 91 2f 00 00 	lea    rdi,[rip+0x2f91]        # 4018 <__TMC_END__>
	    1087:	48 8d 05 8a 2f 00 00 	lea    rax,[rip+0x2f8a]        # 4018 <__TMC_END__>
	    108e:	48 39 f8             	cmp    rax,rdi
	    1091:	74 15                	je     10a8 <deregister_tm_clones+0x28>
	    1093:	48 8b 05 2e 2f 00 00 	mov    rax,QWORD PTR [rip+0x2f2e]        # 3fc8 <_ITM_deregisterTMCloneTable@Base>
	    109a:	48 85 c0             	test   rax,rax
	    109d:	74 09                	je     10a8 <deregister_tm_clones+0x28>
	    109f:	ff e0                	jmp    rax
	    10a1:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]
	    10a8:	c3                   	ret
	    10a9:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]
	
	00000000000010b0 <register_tm_clones>:
	    10b0:	48 8d 3d 61 2f 00 00 	lea    rdi,[rip+0x2f61]        # 4018 <__TMC_END__>
	    10b7:	48 8d 35 5a 2f 00 00 	lea    rsi,[rip+0x2f5a]        # 4018 <__TMC_END__>
	    10be:	48 29 fe             	sub    rsi,rdi
	    10c1:	48 89 f0             	mov    rax,rsi
	    10c4:	48 c1 ee 3f          	shr    rsi,0x3f
	    10c8:	48 c1 f8 03          	sar    rax,0x3
	    10cc:	48 01 c6             	add    rsi,rax
	    10cf:	48 d1 fe             	sar    rsi,1
	    10d2:	74 14                	je     10e8 <register_tm_clones+0x38>
	    10d4:	48 8b 05 fd 2e 00 00 	mov    rax,QWORD PTR [rip+0x2efd]        # 3fd8 <_ITM_registerTMCloneTable@Base>
	    10db:	48 85 c0             	test   rax,rax
	    10de:	74 08                	je     10e8 <register_tm_clones+0x38>
	    10e0:	ff e0                	jmp    rax
	    10e2:	66 0f 1f 44 00 00    	nop    WORD PTR [rax+rax*1+0x0]
	    10e8:	c3                   	ret
	    10e9:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]
	
	00000000000010f0 <__do_global_dtors_aux>:
	    10f0:	f3 0f 1e fa          	endbr64
	    10f4:	80 3d 1d 2f 00 00 00 	cmp    BYTE PTR [rip+0x2f1d],0x0        # 4018 <__TMC_END__>
	    10fb:	75 2b                	jne    1128 <__do_global_dtors_aux+0x38>
	    10fd:	55                   	push   rbp
	    10fe:	48 83 3d da 2e 00 00 	cmp    QWORD PTR [rip+0x2eda],0x0        # 3fe0 <__cxa_finalize@GLIBC_2.2.5>
	    1105:	00 
	    1106:	48 89 e5             	mov    rbp,rsp
	    1109:	74 0c                	je     1117 <__do_global_dtors_aux+0x27>
	    110b:	48 8b 3d fe 2e 00 00 	mov    rdi,QWORD PTR [rip+0x2efe]        # 4010 <__dso_handle>
	    1112:	e8 29 ff ff ff       	call   1040 <__cxa_finalize@plt>
	    1117:	e8 64 ff ff ff       	call   1080 <deregister_tm_clones>
	    111c:	c6 05 f5 2e 00 00 01 	mov    BYTE PTR [rip+0x2ef5],0x1        # 4018 <__TMC_END__>
	    1123:	5d                   	pop    rbp
	    1124:	c3                   	ret
	    1125:	0f 1f 00             	nop    DWORD PTR [rax]
	    1128:	c3                   	ret
	    1129:	0f 1f 80 00 00 00 00 	nop    DWORD PTR [rax+0x0]
	
	0000000000001130 <frame_dummy>:
	    1130:	f3 0f 1e fa          	endbr64
	    1134:	e9 77 ff ff ff       	jmp    10b0 <register_tm_clones>
	
	0000000000001139 <main>:
	    1139:	55                   	push   rbp
	    113a:	48 89 e5             	mov    rbp,rsp
	    113d:	48 83 ec 10          	sub    rsp,0x10
	    1141:	89 7d fc             	mov    DWORD PTR [rbp-0x4],edi
	    1144:	48 89 75 f0          	mov    QWORD PTR [rbp-0x10],rsi
	    1148:	48 8d 05 b5 0e 00 00 	lea    rax,[rip+0xeb5]        # 2004 <_IO_stdin_used+0x4>
	    114f:	48 89 c7             	mov    rdi,rax
	    1152:	e8 d9 fe ff ff       	call   1030 <puts@plt>
	    1157:	b8 00 00 00 00       	mov    eax,0x0
	    115c:	c9                   	leave
	    115d:	c3                   	ret
	
	Disassembly of section .fini:
	
	0000000000001160 <_fini>:
	    1160:	48 83 ec 08          	sub    rsp,0x8
	    1164:	48 83 c4 08          	add    rsp,0x8
	    1168:	c3                   	ret
	
	```

- You can see the binary has a lot more code than the executable file. There are multiple sections now with names `.init`, `.plt`, `.text` [[ELF Linker Sections]].
- The `.text` section is the main code section, and contains the `main` function as well as the `_start` function which prepares the memory for execution of `main` and cleans up after `main`. 
- The incomplete code references have been removed and the code for the `puts` function is available.
- If we look at the stripped binary the functions are still their however the names have been changed and mangled and we cant identify functions except for say `_libc_start_main` which are needed.