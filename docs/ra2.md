## CPPGM Reading Assignment B (target)

### Overview

After reading this assignment description, read the Assigned Reading Material enumerated below.

After this assignment you will have the knowledge necessary to hand-assemble, without using any tools, a native binary Linux x86-64 program that interfaces directly with a x86-64 CPU and a Linux Kernel. You will use this knowledge in the construction of your backend, as it is required to output such programs.

### Prerequisites

You should complete Programming Assignment 8 before starting this assignment. PA8 introduces the PA8 Mock Program Image format which should start to give you a feel for the issues you will face in linking a program. This mock format will be replaced by the real ELF format in following assignments. Also the mock function stubs will be replaced by real x86-64 machine code which you will learn how to assemble in this assignment. You will also need to learn how to assemble a Linux system call in order to generate the implementation of the `syscall` function, which is the fundamental interface between a program and the kernel. Much of your standard library will be implemented on top of system calls, but your first use of a system call will be to terminate your program after exit of main.

You are also expected to have an existing understanding of operating systems and computer architecture. In particular you should know about registers, memory, CPU cores, CPU caches, processes, threads, virtual memory, paging, segmentation, scheduling, interrupts. While you will not be implementing an operating system kernel or CPU as part of this course, you will be interfacing directly with them. You should also understand basic Unix concepts such as the VFS abstraction layer, signals and process management.

If you feel as though you are missing these OS prerequisites then we recommend reading the Dinosaur Book: ["Operating System Concepts 9th Edition, Silberschatz/Galvin/Gagne"](www.os-book.com), or a similar operating systems textbook.

### ELF x86-64 Executable

ELF stands for Executable and Linkable Format. It is the format used for a standard native executable on Linux.

A Linux program consists of an initial file header called the ELF header, followed by an array of program segment headers, followed by the code and data of your program.

A Linux program is executed by the system call `execve`. For example, when you execute in the shell:

    $ ./foo

The shell program finds the file `foo`, checks that you have permission to execute it (the executable bit), and if so creates a new process for the program, and then calls the system call `execve`. We will discuss system calls in detail later in this assignment, for now you can think of them as kernel functions.

`execve` is given as a parameter the filepath of the `foo` file and any command-line arguments. The kernel opens the file and reads the ELF header and program segment headers. There are several types of program segment headers, but the only one we need is called `PT_LOAD`. A `PT_LOAD` segment instructs the kernel to load part of the program image from the file into memory. The header describes a range of source bytes in the file, and a range of destination bytes in memory. It also specifies read/write/execute permissions for that memory. The kernel then copies that part of the file into memory, and gives that memory the specified permissions. If the specified destination is longer than the source, the remainder is zero-initialized.

The kernel also automatically sets up a stack for your program. It pushes the argc and argv and parameters for use by main on to the stack, and the stack pointer register (RSP) is set appropriately.

The file descriptors 0, 1 and 2 are also setup (actually held over). They describe the standard input file (stdin = 0), standard output file (stdout = 1) and standard error file (stderr = 2).

The ELF header also specifies a memory address called the entry point. After loading is complete the kernel jumps into user-space at the specified address and starts executing the machine code. This will be the machine code you have loaded from the program image.

The ELF header is 64 bytes long and has the following format:

    struct ElfHeader
    {
        unsigned char ident[16] =
        {
            0x7f, 'E', 'L', 'F', // magic bytes
            2, // 64-bit architecture
            1, // two's compliment, little-endian
            1, // ELF specification version 1.0
            0, // System V ABI
            0, // ABI Version
            0, 0, 0, 0, 0, 0, 0 // Unused padding
        };

        short int type = 2; // executable file type
        short int machine = 0x3E; // x86-64 Architecture
        int version = 1; // ELF specification version 1.0
        long int entry = ???; // entry point virtual memory address

        long int phoff = ???; // start of program segment header array file offset
        long int shoff = 0; // no sections

        int processor_flags = 0; // no processor-specific flags
        short int ehsize = 64; // ELF header is 64 bytes long
        short int phentsize = 56; // program header table entry size
        short int phnum = ???; // number of program headers       
        short int shentsize = 0; // no section header table entry size
        short int shnum = 0; // no sections
        short int shstrndx = 0; // no section header string table index
    };

Each program segment header is 56 bytes long and has the following format:

    struct ProgramSegmentHeader
    {
        int type = 1; // PT_LOAD: loadable segment

        static constexpr int executable = 1 << 0;
        static constexpr int writable = 1 << 1;
        static constexpr int readable = 1 << 2;

        int flags = executable | writable | readable; // segment permissions

        long int offset = ???; // source file offset
        long int vaddr = ???; // destination (virtual) memory address
        long int paddr = 0; // unused, doesn't use physical memory
        long int filesz = ???; // source length
        long int memsz = ???; // destination length
        long int align = 0; // unused, alignment of file/memory
    };

Many of these fields have fixed or unused values for our purposes. This is because ELF files can also be used to store relocatable object files, shared libraries and core dumps - and not just programs. They are also used by a range of different target platforms and not just Linux x86-64\. We have hard-coded the values that you don't need to change for Linux x86-64 programs. The values you need to fill out are marked `???`.

If you are interested in learning more information about the ELF format you can consult the man page `elf(5)`:

    $ man 5 elf

#### ELF Warmup Exercise (ungraded)

We ask you to perform the following exercise to get a feel for the ELF format:

> Write a C++ application called `infinite_loop_maker` that outputs a 122 byte Linux program called `infinite_loop` that goes into an infinite loop on launching.

This should consist of:

1.  1 x ElfHeader (64 bytes)
2.  1 x ProgramSegmentHeader (56 bytes)
3.  2 bytes of x86-64 machine code (2 bytes)

For a total of 122 bytes.

The machine code is the two bytes 0xEB and 0xFE. The first byte is the opcode `JMP`, and the second byte is an immediate signed char `-2` operand (as you should recall in 2s-compliment -1 is 0xFF, -2 is 0xFE, -3 is 0xFD, and so on). The instruction means jump backwards 2 bytes, which then reruns the instruction, jumps back again, and so on in an infinite loop.

The recommended design is to use an `std::ofstream` in binary mode to write the file. You should copy and paste the above `ElfHeader` and `ProgramSegmentHeader` into a program and fill out the `???` with actual values.

We recommend loading the entire ELF file into memory at offset 0x400000 (4 MB), including the ELF header and all. This seems to be the traditional position for the program image in production toolchains. Also, by loading everything it makes it easy to go from file offset to memory offset by simply adding or subtracting 0x400000 as appropriate. For example the byte at offset 0x1234 in the file will be at byte 0x401234 in memory.

Once you have filed out your `ElfHeader` and `ProgramSegmentHeader` as you wish then write them to the file, followed by the 2 bytes of machine code.

After you have closed the file you will want to set the executable bit. You can do so with the following `RABSetFileExecutable` function:

    // bootstrap system call interface, used by RABSetFileExecutable
    extern "C" long int syscall(long int n, ...) throw ();

    // RABSetFileExecutable: sets file at `path` executable
    // returns true on success
    bool RABSetFileExecutable(const string& path)
    {
        int res = syscall(/* chmod */ 90, path.c_str(), 0755);

        return res == 0;
    }

The function calls system call 90, called `chmod(2)`. You may recognize this from the command-line utility of the same name.

Run your program:

    $ ./infinite_loop_maker

And then run the produced program:

    $ ./infinite_loop

The program should never return, as expected. To terminate it you can use Ctrl-C. If it doesn't work (for example outputs `Killed` on launching) you can try debugging it with `objdump`, `readelf` and/or `gdb`. You can also post to forum.cppgm.org with tag `rab` if you are stuck.

Note that the `ElfHeader`, `ProgramSegmentHeader` and `RABSetFileExecutable` entities given here are considered part of the course starter code and you may use them in any future course assignments.

### x86-64 Primer

x86-64 machine code logically consists of a sequence of instructions. Each instruction is encoded into a series of bytes between 1 and 16 in length. Using our example from above `JMP rel8=-2`, there was one instruction that was 2 bytes in length.

An encoded instruction consists of between 1 and 10 different parts. Each part encodes some semantic information about the instruction in a (possibly not contiguous) series of bit positions. Which parts are used in an instruction and what they mean generally depends on the instruction. You'll study the details of the encoding in the assigned reading material.

Roughly speaking an instruction consists of an opcode, zero or more operands and sometimes prefix modifiers. An operand can be a register, a memory address or an immediate. A register is encoded with a simple bit field that gives the registers number. An immediate is a value encoded in the same format as a PA2 literal and placed directly in the encoded instruction (always at the end of the instruction).

A memory address is more complicated. It is encoded as one of a set number of expressions usually involving registers. The simplest expression is just the dereference of a register. For example the expression `[reg]` means the memory address held in the register `reg`. Expressed in C++ you can think about it like this:

    unsigned long int reg; // the register reg

    char* ptr = (char*) reg; // use the register as an address

    char& operand = *ptr; // dereference the register

There are more complicated memory address expressions encodable using two or more registers, a multipicative factor and a fixed offset called a displacement. For example `[reg1 + reg2*8 + 42]`. The C++ view of this is:

    unsigned long int reg1, reg2; // the registers involved

    char* ptr = (char*) (reg + reg2*8 + 42); // use the expression as an address

    char& operand = *ptr; // dereference the register

These complex expressions can be encoded as a single instruction. For our purposes they are not that interesting and only used to increase machine code density and improve performance at nano-scale.

#### Instruction Set

There are hundreds of x86-64 instructions in total but we will only need to use a small subset of them.

Firstly, there are the base general-purpose instructions, and then there are a number of extensions (x87 FPU, MMX, SSE, SSE2, SSE3, SSSE3, SSE4, AESNI, AVX, etc). An extension adds new registers and new instructions. Of the extensions, you will only need to use the x87 FPU to perform floating point operations.

Secondly, of the general-purpose instructions, many of them are for the operating system to use, and not useful for us in user-space. Many are effectively deprecated. Also, many support features we don't need such as binary decimal arithmetic, and others.

#### Registers

We will list the important 64-bit addressable general-purpose registers here. There are 16 of them:

    RAX    RDI    R8    R12
    RBX    RSI    R9    R13
    RCX    RBP    R10   R14
    RDX    RSP    R11   R15

In general you can address them interchangably. There are a few instructions that are hardcoded to one register - but the usual register bit field is 4 bits, and will select one of these 16 registers. Also the memory address expressions use these 16 registers. Most of the time you will be dealing with these registers directly.

There is also a non-addressable register called RIP, and a bit field register called RFLAGS. There are other registers as well, but they can be ignored.

The RSP stores the stack pointer. It is set by the kernel on entry to your program. Usually RBP is used to store the base pointer. When you enter a function traditionally you will point RBP at the bottom of the activation frame to give a fixed reference into the frame.

On x86-64 the stack grows downwards, so RSP will have a lower value than RBP. Also, if you "push" something onto the stack RSP decreases.

RIP is the instruction pointer. You can't address it directly, but you can change it with the JMP instruction and related conditional jumps. It points to the current instruction being executed.

The RFLAGS register holds results from previous instructions that are used in later instructions. For example after a CMP x y operation, it will set bits in this register. These bits are then implicitly used by a following conditional jump instruction like JNE which means Jump Not Equal. You don't write to RFLAGS directly.

Each of the 16 registers has 64-bit, 32-bit, 16-bit and 8-bit forms that are useful. We list them here:

    64-bit    32-bit    16-bit    8-bit
    RAX       EAX       AX        AL
    RBX       EBX       BX        BL
    RCX       ECX       CX        CL
    RDX       EDX       DX        DL
    RDI       EDI       DI        DIL
    RSI       ESI       SI        SIL
    RBP       EBP       BP        BPL
    RSP       ESP       SP        SPL
    R8        R8D       R8W       R8B
    R9        R9D       R9W       R9B
    R10       R10D      R10W      R10B
    R11       R11D      R11W      R11B
    R12       R12D      R12W      R12B
    R13       R13D      R13W      R13B
    R14       R14D      R14W      R14B
    R15       R15D      R15W      R15B

An n-bit version of an m-bit register, where m > n, is an alias for the lower n bits of the m-bit register.

For example if RAX is set to 0x8877665544332211 then:

    RAX = 0x8877665544332211
    EAX =         0x44332211
    AX  =             0x2211
    AL  =               0x11

Likewise for the other 15 sets of registers. This is useful in various contexts.

Finally we discuss the x87 FPU. The x87 FPU has its own set of registers, but they function in a different way than the general purpose registers. You use the x87 FPU as a stack machine. Its internal registers form a stack of `long doubles`, so if you want to perform an addition of floating pointer numbers `z = x + y` you can do so roughly like this:

    FLD x  // floating load
    FLD y  // floating load
    FADD   // floating add
    FST z  // floating store

Where x, y and z are memory addresses. FLD pushes its operand onto the floating point register stack. FADD pops the top two adds them then pushes the result. FST pops the result off the stack and into the memory address z. These are not the exact opcodes and syntax for this, but we are just conveying the general idea of how to use it. The x87 FPU can also perform conversions from the integral types, floats and doubles to long doubles and visa versa as operands are pushed onto and off the stack. Internally the registers are long doubles.

### System Calls

The primary way Linux programs communicate with the Linux kernel is via system calls. You can think of system calls as the Linux Kernel API for userland programs. Note that your standard C and C++ library will reside above and use this API.

System calls are very similar to functions. The difference is that rather than jumping to a code address within the user-space code, to call a system call you use a special x86-64 CPU instruction called `SYSCALL`. The parameters of the system call are placed in certain specific registers, and one of the parameters is the _system call number_ which is an integer that designates which of the system calls you want to call.

When a system call starts the process goes from user-space to kernel-space. The kernel then executes the system call. Once completed a return value is placed in a register and control returns to the user-space program. So it should be easy to visualize how such an action can be wrapped with a C function as follows:

    int syscall(int number, ...);

There are a little over 300 system calls. We list some of the more important ones here:

    NUMBER   SYSCALL   DESCRIPTION
    56       clone    create a process
    59       execve   execute program
    60       exit     terminate current process
    62       kill     send a signal to a process

`clone` is a very important system call. It is used to create new threads and processes. Under Linux a thread and a process are actually the same thing, they just have different settings passed to the `clone` system call. There is an older system call called `fork`, but it is largely a less flexible version of `clone`. In fact in the C library the function `fork` can and is often implemented by calling `clone`.

`execve` is the system call used to load a program. When you execute an application with the shell it calls `execve` on the program file.

`exit` is the system call you will use on exit of the `main` function when the program is complete. The parameter passed to the call becomes the exit status of the program.

    9        mmap      create a memory map

`mmap` is also a very important system call. It is used to make changes to the virtual memory. You will use it later to implement the heap (dynamic storage duration), as it can allocate large chunks of new zero-initialized memory, but it is also used for a large range of other purposes too. In fact when loading a program the kernel calls it to load your program into memory. It is also used by the kernel to allocate the stack.

    0        read      read from a file
    1        write     write to a file
    2        open      open a file
    3        close     close a file
    4        stat      get file status

A file on linux includes not only regular files, but all sorts of readable/writable things like sockets, pipes, devices. Standard input, standard output and standard error are files. Their file descriptors are already open on program entry and are numbered 0, 1 and 2 respectively. You use `read` to read from standard input, and `write` to write to standard output and standard error.

There are many other system calls for managing memory, the filesystem, networking and other things. For a list of of system calls enter:

    $ man 2 syscalls

and to find the description of a specific system call `foo` enter:

    $ man 2 foo

### System Call Linux x86-64 Calling Conventions

System calls have a maximum of 6 parameters. They are passed in the following registers:

    RAX = system call number
    RDI = parameter 1
    RSI = parameter 2
    RDX = parameter 3
    R10 = parameter 4
    R8  = parameter 5
    R9  = parameter 6

A system call is executed with the special SYSCALL intruction. Its opcode is `{0x0F, 0x05}`.

During the system call the kernel potentially clobbers registers RCX and R11, so they are in an undefined state after the system call.

After the syscall returns, RAX will contain the return value.

A value in the range -4095 to -1 inclusive indicates an error. It is the negative of the errno number from the documentation.

### Assigned Reading Material

The _Intel 64 and IA-32 Architectures Software Developer's Manual_ describes the interface and behaviour of an x86-64 CPU. It is split into three volumes numbered 1, 2 and 3\. Volume 2 is split into three parts 2A, 2B and 2C. Likewise, Volume 3 is split into three parts 3A, 3B and 3C.

There is a combined PDF of all three volumes available for download from the following link:

[http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)

Volume 1 describes the basic architecture and programming environment of the processor. You are assigned to read the following chapters from Volume 1:

*   Volume 1, Chapter 3: Basic Execution Environment
*   Volume 1, Chapter 4: Data Types
*   Volume 1, Chapter 5: Instruction Set Summary
*   Volume 1, Chapter 7: Programming With General-Purpose Instructions
*   Volume 1, Chapter 8: Programming with the x87 FPU
*   Volume 1, Appendix A EFLAGS Cross-Reference
*   Volume 1, Appendix B EFLAGS Condition Codes

Volume 2 describes each x86-64 Instruction in alphabetical order. You are assigned to read the following chapters from Volume 2:

*   Volume 2, Chapter 2: Instruction Format
*   Volume 2, Chapter 3.1: Interpreting the Instruction Reference Pages
*   Volume 2, Appendix B: Instruction Formats and Encodings

You may find it helpful after completing the above to make another quick pass over the ABI document. It is not required or recommended to implement the ABI internally within your implementation. The only parts of the ABI that are required are those visible to the kernel (type size, alignment, struct layout, etc). Internally within your implementation you can (and are encouraged to) use an original design, but it is worth being aware of the rest of the parts of the ABI, at least for design ideas and so you can relate to production toolchains like gcc and clang. The ABI document is here: [www.x86-64.org/documentation_folder/abi.pdf](http://www.x86-64.org/documentation_folder/abi.pdf)

### Instructor Notes / Motivation (optional)

For many of you your experience of assembly programming will so far involve writing a small toy assembly language production in a RISC instruction set like MIPS, and few will have actually written an assembler before, and certainly not for a CISC instruction set like x86\. What is worse is that x86, like C++, has evolved over decades and has the burden of backward compatibility. You must keep in mind that this burden of backward compatibility is a fundamental property of dominant production technologies. In order to stay dominant the designers can simply not make major breaking changes, so when they make a bad design decision (which they inevitably do because they're only human) we have to live with it, or at least work around it, for years to come. We are learning about C++, Linux and x86 - not because they are ideal and perfect - but because they are the dominant base technologies across all of software engineering. Each of the three dominates at least two of embedded, desktop and server.

You'll find after writing an x86-64 assembler that understanding all the different x86 assembly syntaxes (and there are several) to be very easy. You will use this in several places. The first and foremost is in debugging, dropping to disassembly will become very natural and you will be able to quickly read the disassembly without going backward and forward to the reference on each instruction. Scarily you will even be able to disassemble in your head some parts of instructions by recognizing the hex machine code on the left when the assembler syntax on the right is trying to be too clever. Another place you will use this is to write very high performance assembly "kernels" (as in computation kernels, not operating system kernels). Typically in performance critical code you can isolate the bottleneck to one function. You will be able to replace that function with hand-written assembly, perhaps using one of the x86-64 SIMD extension instruction sets. While there are many advances in compiler construction and optimization, we haven't solved the problems to a point where the compiler can always do a better job than a skilled assembly programmer using CPU-specific features (and even if we had, which we haven't, someone has to write and maintain those theoretic amazingly intelligent compilers anyway).

Having said all that, this assignment marks the lower bound of the course syllabus, and this is as low-level and close to the metal as we will get.
