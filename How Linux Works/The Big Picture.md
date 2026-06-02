The Linux system is split in multiple layers of abstraction each consisting of components which perform different tasks.

-  The lowest layer is _hardware_, which consists the processor, RAM, disks, ports etc.
- The next layer above it is the _kernel_, which is the core of the operating system, the kernel is software which exists in memory and tells the CPU where to look for its next task.
- Finally we have the _Processes_, the running programs managed by the kernel, called the _user space_, these include the GUI, Servers any bash cells etc.

Now there is a difference between how the kernel and user processes run, the kernel runs in _kernel mode_, and the user processes run in _user mode_.

Code running in the kernel mode has complete access to the processor and main memory, which is powerful but also dangerous, _kernel space_ is the memory which only the kernel can access, on the other hand the user processes run in user mode and have a limited access to resources.

> The Linux kernel can run kernel threads which look like processes but have access to kernel space.

##### The Kernel
The Kernel is responsible for four general system areas
- __Processes__: The kernel tells which process are allowed to use the CPU
- __Memory__: The kernel keeps track of memory, what is currently allocated to processes and what is shared, free.
- __Device Drivers__: Kernel acts as a interface between hardware and processes, using device drivers.
- __System calls and support__: Processes use system calls to the kernel to communicate with it.

##### Process Management
_Process Management_ involves starting, pausing, resuming and scheduling process, on most operating systems many processes seem to run "simultaneously", however this is not the truth.

On a single core CPU, only one process can occupy the CPU at a time, each process uses the CPU for a small fraction of a second and then pauses, and this repeats, the act of process giving up control of the CPU to take another process is called a _context switch_.

Each piece of time called a _time slice_, gives a process time to do a significant task, allowing simulation of multi tasking, the kernel is responsible for context switching.

> The CPU interrupts the process based on an internal timer, and switches to kernel mode. The kernel saves the state of the CPU and memory, then it performs any tasks which might have come up during the preceding time slice, the kernel chooses another process to run, prepares the CPU and memory, gives the CPU the length of the time slice and then the kernel switches the CPU into user mode and this process repeats.

##### Memory Management 
The kernel manages memory during context switches, the following conditions must hold to allow for proper memory management

- The kernel has its own memory section that user processes cannot access.
- Each user process needs its own section of memory.
- One process cannot access another the private memory of another process.
- User processes can share memory
- Some memory in user processes is user only.
- The disk can use more memory then physically present by using disk space as auxiliary.

Modern CPUs have a _memory management unit(MMU)_, that enables a memory access scheme called _virtual memory_.

The processes do not access memory directly by its physical location, rather the kernel sets up process as if it has the whole system to itself, when the processes access memory the MMU maps the virtual address to physical address using a memory address map.

The kernel maintains and alters this memory map as required, such as changing it when processes change.

> The implementation of the memory address map is called the page table

##### Device Drivers and Management
A device is typically accessible only in kernel mode since improper access could crash the machine. Different devices even if they perform the same task could have different interfaces, thus the kernel usually has device drivers integrated into it to make a software developers job easier.

###### System Calls and Support
System calls (_syscalls_), perform specific task that a user process cannot do well or can't do at all.
Two system calls `fork()` and `exec()`, are important to understanding how processes start.
- __fork()__: When a process calls fork, the kernel creates a near identical copy of the process.
- __exec()__: When a process calls exec(program), the kernel loads and starts program, replace the current process.
Other than `init` all user space processes are a result of `fork()`. 

> The kernel supports user processes with features other than system calls, the most common are _pseudodevices_, which look like devices to user process but are purely software, the usually do exist in kernel due to practical reason such as `/dev/random`, which is not secure in user space.

### User Space
The main memory which the kernel allocates for user processes is user space, it may also be called the collection of memory in which user processes run.
### Users
A _user_ is an entity that can run processes and own files, it has a username. The kernel identifies users by user IDs which are just numeric identifiers.

Users exist to support permissions and boundaries, user space process have a _owner_ and run as the _owner_. A user can modify its owned process but not of other users.

A linux system has many users besides actual humans using the system, most important one being root which doesn't need to abide to the above rule and can alter others users processes and access any file.