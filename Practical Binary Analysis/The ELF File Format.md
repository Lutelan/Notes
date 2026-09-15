### The Executable Header
- Every ELF file starts with an executable header, which is just a structured series of bytes telling you that it’s an ELF file, what kind of ELF file it is, and where in the file to find all the other contents.
- Its format is located in `usr/include/elf.h` or in the ELF specification, a 64-bit ELF header looks like this,
```
typedef struct {
	unsigned char e_ident[16]; /* Magic Number and Other info        */
	uint16_t e_type;           /* Object file type                   */
	uint16_t e_machine;        /* Architecture                       */
	uint32_t e_version;        /* Object File Version                */
	uint64_t e_entry;          /* Entry point virtual address        */
	uint64_t e_phoff;          /* Program header table file offset   */
	uint64_t e_shoff;          /* Section header table file offset   */
	uint32_t e_flags;          /* Processor-specific flags           */
	uint16_t e_ehsize;         /* ELF header size in bytes           */
	uint16_t e_phentsize;      /* Program header table entry size    */
	uint16_t e_phnum;          /* Program header table entry count   */
	uint16_t e_shentsize;      /* Section header table entry count   */
	uint16_t e_shnum;          /* Section header table entry count   */
	uint16_t e_shstrndx;       /* Section header string table index  */
} Elf64_Ehdr;
```
- We can read the executable header of a ELF file as follows
```
$ readelf -h a.out
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1050
  Start of program headers:          64 (bytes into file)
  Start of section headers:          13984 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```
- ###### The `e_ident` array
	- This array is a 16-byte array at the start of the header
	- `e_ident` starts with a 4-byte _magic value_ identifying the ELF binary.
	- The _magic value_ is `0x7f` followed by the ASCII characters _E_, _L_, and _F_, this helps tools like `file` or interpreters identify the file.
	- After this indexed 4-15 you have bytes referred to as EI_CLASS, EI_DATA, EI_VERSION, EI_OSABI, EI_ABIVERSION, EI_PAD, respectively.
	- The EI_PAD consists of many bytes(9-15) they are all set to 0's and are for padding or for future use.
	- The EI_CLASS byte denotes which ELF specification is of the binary file. For 32-bit machines it is ELFCLASS32(which is 1) and on 64-bit machines it is ELFCLASS64(equal to 2).
	- The EI_DATA byte indicates the endianness of the binary. A value of ELFDATA2LSB(equal to 1) indicates little-endian, & a value of ELFDATA2MSB(equal to 2) means big-endian.
	- The EI_VERSION byte indicates the version of the ELF specification when used creating the binary. Currently the only valid value if EV_CURRENT which is defined to be 1.
	- The EI_OSABI and EI_ABIVERSION denotes information about the _application binary interface_(ABI) and the operating system(OS).
	- If EI_OSABI is a non-zero value it means some ABI or OS specific extensions are present in the ELF file, the default value of 0 indicates the Unix System V ABI. 
	- The EI_ABIVERSION indicates the version of the ABI defined in EI_OSABI, this is usually 0 when the default EI_OSABI value is used.

- ###### The `e_type`, `e_machine` and `e_version` Fields
	- The `e_type` specifies the type of the binary. The most common values are ET_REL(a relocatable object file), ET_EXEC(an executable binary), & ET_DYN(a dynamic library).
	- The `e_machine` field denotes the architecture that the binary is intended to run on. This is usually EM_X86_64(64-bit) or EM_386(32-bit), or EM_ARM(for ARM binaries).
	- The `e_version` field indicates the version of the ELF Specification used when creating the binary, the only possible value is 1.

- ###### The `e_entry` Field
	- The `e_entry` field denotes the _entry point_ of the binary, the virtual address at which execution should start, the execution stars at `0x1050`, this is where the interpreter transfers control to the binary.

- ###### The `e_phoff` and `e_shoff` Fields
	- The `e_phoff` & `e_shoff` fields denote the file offsets to the beginning of the program header table & section header table.
	- They can be set to 0 to indicate that their are no program header or section header

- ###### The `e_flags` Field
	- The `e_flags` field provides space for architecture specific flags when the binary is compiled, for x86 binaries it is not of interest.

- ###### The `e_ehsize` field
	- The `e_ehsize` field contains the size of the executable header in bytes.

- ###### The `e_*entsize` and `e_*num` fields
	- The `e_phentsize`, `e_phnum` and `e_shentsize`, `e_shnum` provide information about the size of the program and section headers as well as the number of headers in each table.

- ###### The `e_shstrndx` Field
	- The `e_shstrndx` field contains the index(in the section header table), of the header associated with a special _string table_ section called `.shstrtab`.
	- This section is dedicated to storing NULL-terminated ASCII strings, which store the names of all the sections in the binary.
	- We can view the data in the section using `readelf` as follows
	```
	$ readelf -x .shstrtab a.out
	
	Hex dump of section '.shstrtab':
	  0x00000000 002e7379 6d746162 002e7374 72746162 ..symtab..strtab
	  0x00000010 002e7368 73747274 6162002e 696e7465 ..shstrtab..inte
	  0x00000020 7270002e 6e6f7465 2e676e75 2e70726f rp..note.gnu.pro
	  0x00000030 70657274 79002e6e 6f74652e 676e752e perty..note.gnu.
	  0x00000040 6275696c 642d6964 002e6e6f 74652e41 build-id..note.A
	  0x00000050 42492d74 6167002e 676e752e 68617368 BI-tag..gnu.hash
	  0x00000060 002e6479 6e73796d 002e6479 6e737472 ..dynsym..dynstr
	  0x00000070 002e676e 752e7665 7273696f 6e002e67 ..gnu.version..g
	  0x00000080 6e752e76 65727369 6f6e5f72 002e7265 nu.version_r..re
	  0x00000090 6c612e64 796e002e 72656c61 2e706c74 la.dyn..rela.plt
	  0x000000a0 002e696e 6974002e 706c742e 676f7400 ..init..plt.got.
	  0x000000b0 2e746578 74002e66 696e6900 2e726f64 .text..fini..rod
	  0x000000c0 61746100 2e65685f 6672616d 655f6864 ata..eh_frame_hd
	  0x000000d0 72002e65 685f6672 616d6500 2e696e69 r..eh_frame..ini
	  0x000000e0 745f6172 72617900 2e66696e 695f6172 t_array..fini_ar
	  0x000000f0 72617900 2e64796e 616d6963 002e676f ray..dynamic..go
	  0x00000100 742e706c 74002e64 61746100 2e627373 t.plt..data..bss
	  0x00000110 002e636f 6d6d656e 7400              ..comment.
	```

### Section Headers
- The code and data in an ELF binary are logically divided into contiguous non-overlapping chunks called _sections_.
- Every section has a _section header_ which indicates the properties of the section and allows you to locate bytes in the section.
- The section header for all sections is contained in the _section header table_.
- Sections are only required for the linker and hence files which do not need to be linked do not require a _section header table_ and the `e_shoff` field is set to 0.
- To load & execute a binary we need a different organisation of code and data, for ELF files these are called _segments_ and are used during execution.
- The structure of the ELF section header is as follows
```
typedef struct {
	uint32_t sh_name;         /* Section Name (string tbl index)             */
	uint32_t sh_type;         /* Section type                                */
	uint64_t sh_flags;        /* Section flags                               */
	uint64_t sh_addr;         /* Section virtual addr at execuution          */
	uint64_t sh_offset;       /* Section file offset                         */
	uint64_t sh_size;         /* Section size in bytes                       */
	uint32_t sh_link;         /* Link to another section                     */
	uint32_t sh_info;         /* Additional section information              */
	uint64_t sh_addralign;    /* Section Alignment                           */
	uint64_t sh_entsize;      /* Entry size if secion holds table            */
} Elf64_Shdr;
```
- ###### The `sh_name` Field
	- The field `sh_name`, if set contains an index into _string table_. If the index is zero, it means the section does not have a name.

- ###### The `sh_type` Field
	- Every section has a type and this is indicated by an integer field called `sh_type`.
	- Type SHT_PROGBITS contains program data, such as machine instructions or constants.
	- You have SHT_SYMTAB for static symbol tables and SHT_DYNSYM for dynamic symbol tables used. 
	- Sections with type SHT_REL and SHT_RELA are particularly important for the linker as they contain relocation entries in a specific format.
	- Sections of type SHT_DYNAMIC contain information needed for dynamic linking.

- ###### The `sh_flags` Field
	- Section flags describe additional information about a section.
	- The most important are SHF_WRITE, SHF_ALLOC, and SHF_EXECINSTR.
	- SHF_WRITE indicates that the section is writable during run time, this makes it easy to distinguish between sections with static data and others.
	- SHF_ALLOC indicates that the contents of the section are to be loaded into virtual memory when the binary is executed.
	- SHF_EXECINSTR tells you that a section contains executable instructions which is important during disassembly.

- ###### The `sh_addr`, `sh_offset`, and `sh_size` Fields
	- The `sh_addr`, `sh_offset` and `sh_size` fields describe the Virtual Address, file offset and size of the section respectively.
	- Sections which are not loaded into memory during execution are have a `sh_addr` field of 0.

- ###### The `sh_link` Field
	- Sometimes there are relationships between sections which the linker needs to know about.
	- For instance an SHT_SYMTAB, SHT_DYNSYM or SHT_DYNAMIC section has an associated string table section, which contains the symbolic names for the symbols in context.
	- The `sh_link` field makes these relationships explicit by denoting the index of section (in the section header table) of the related section.

- ###### The `sh_info` Field
	- The `sh_info` field contains additional information about the section. The meaning of the additional information varies depending on the section type.
	- For instance for relocation sections `sh_info` denotes the index of the section to which relocation are to be applied.

- ###### The `sh_addralign` Field
	- Some sections needed to aligned in memory in a particular way for efficient memory access.
	- These alignment requirements are specified in the `sh_addralign` field. For instance if this field is set to 16 the base address of the loaded section must be a multiple of 16.
	- The values of 1 or 0 indicate no special alignment needs.

- ###### The `sh_entsize` Field
	- Some sections such as symbol tables or relocation tables contain a table of well-defined structures. For such sections the `sh_entsize` field indicates the size in bytes of each entry in the table.
	- When the field is unused it is set to 0.
### Sections
- We can view all the sections of a binary using `readelf` as follows
```
There are 31 section headers, starting at offset 0x36a0:

Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        0000000000000318 000318 00001c 00   A  0   0  1
  [ 2] .note.gnu.property NOTE            0000000000000338 000338 000020 00   A  0   0  8
  [ 3] .note.gnu.build-id NOTE            0000000000000358 000358 000024 00   A  0   0  4
  [ 4] .note.ABI-tag     NOTE            000000000000037c 00037c 000020 00   A  0   0  4
  [ 5] .gnu.hash         GNU_HASH        00000000000003a0 0003a0 000024 00   A  6   0  8
  [ 6] .dynsym           DYNSYM          00000000000003c8 0003c8 0000a8 18   A  7   1  8
  [ 7] .dynstr           STRTAB          0000000000000470 000470 00008d 00   A  0   0  1
  [ 8] .gnu.version      VERSYM          00000000000004fe 0004fe 00000e 02   A  6   0  2
  [ 9] .gnu.version_r    VERNEED         0000000000000510 000510 000030 00   A  7   1  8
  [10] .rela.dyn         RELA            0000000000000540 000540 0000c0 18   A  6   0  8
  [11] .rela.plt         RELA            0000000000000600 000600 000018 18  AI  6  24  8
  [12] .init             PROGBITS        0000000000001000 001000 000017 00  AX  0   0  4
  [13] .plt              PROGBITS        0000000000001020 001020 000020 10  AX  0   0 16
  [14] .plt.got          PROGBITS        0000000000001040 001040 000008 08  AX  0   0  8
  [15] .text             PROGBITS        0000000000001050 001050 00010e 00  AX  0   0 16
  [16] .fini             PROGBITS        0000000000001160 001160 000009 00  AX  0   0  4
  [17] .rodata           PROGBITS        0000000000002000 002000 000012 00   A  0   0  4
  [18] .eh_frame_hdr     PROGBITS        0000000000002014 002014 00002c 00   A  0   0  4
  [19] .eh_frame         PROGBITS        0000000000002040 002040 0000ac 00   A  0   0  8
  [20] .init_array       INIT_ARRAY      0000000000003dd0 002dd0 000008 08  WA  0   0  8
  [21] .fini_array       FINI_ARRAY      0000000000003dd8 002dd8 000008 08  WA  0   0  8
  [22] .dynamic          DYNAMIC         0000000000003de0 002de0 0001e0 10  WA  7   0  8
  [23] .got              PROGBITS        0000000000003fc0 002fc0 000028 08  WA  0   0  8
  [24] .got.plt          PROGBITS        0000000000003fe8 002fe8 000020 08  WA  0   0  8
  [25] .data             PROGBITS        0000000000004008 003008 000010 00  WA  0   0  8
  [26] .bss              NOBITS          0000000000004018 003018 000008 00  WA  0   0  1
  [27] .comment          PROGBITS        0000000000000000 003018 00001f 01  MS  0   0  1
  [28] .symtab           SYMTAB          0000000000000000 003038 000360 18     29  18  8
  [29] .strtab           STRTAB          0000000000000000 003398 0001e9 00      0   0  1
  [30] .shstrtab         STRTAB          0000000000000000 003581 00011a 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```
- For each section `readelf` shows the relevant basic information, including the index, name and type of the section.
- You can see the virtual address, file offset, and size in bytes of each section. We can also observe the type and the various entries in the section headers.
- ###### The `.init` and `.fini` Sections
	- The `.init` section contains executable code that performs initialisation tasks and needs to run before any other code in the binary is executed.
	- We can tell it contains executable code by the SHF_EXECINSTR flag, denotes by a X by `readelf`.
	- The system executes executes the code in the `.init` section before transferring control to the main entry point of the binary.
	- The `.fini` section is analogous to the `.init` section except it runs after the program completes, essentialy functioning as a kind of destructor.
- ###### The `.text` Sections
	- The `.text` section is where the main code of the program resides
