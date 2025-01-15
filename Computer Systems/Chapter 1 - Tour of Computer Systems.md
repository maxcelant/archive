### Program Translation
Made up of four stages: (preprocessor, compiler, assembler, and linker) are known collectively as the **compilation system**.

1. **Preprocessing Phase**: Modifies the C program, adds header files like `#include <stdio.h>` to the program text. Creates `hello.i` file.
2. **Compilation Phase**: Turns C into Assembly. Creates `hello.s` file.
3. **Assembly Phase:** Translates Assembly to machine language instructions. Packages them in a _relocatable object program_. Creates `hello.o` file.
4. **Linking Phase**: Links other object files with our object file. Like the `printf.o` object file. The result is the `hello` executable object file.

### System Organization
>![[Pasted image 20231126192126.png]]

When we execute the `./hello` executable, the instructions are loaded from disk to main memory. From there, the CPU can begin executing the machine language instructions of `hello`.

---

**Buses**: Transfer fixed-sized chunks of bytes called _words_ between components.

**I/O Devices**: Computers connection to outside world. 

**Main Memory**: Temporary storage device that holds both the program and the data it manipulates while the processor is executing the program. A collection of DRAM chips. Organized as a linear array of bytes. 

**Processor (CPU)**: Executes the instructions stored in main memory pointed at by the Program Counter. Possible operations it could carry out:
- **Load**: Copy a byte/word from main memory into a register (can overwrite what was previously there).
- **Store**: Copy a byte/word from a register into main memory (can overwrite what was previously there).
- **Operate**: Copy contents of two registers to ALU and perform some arithmetic operation.
- **Jump**: Extract a word from instruction itself and copy it into the PC, overwriting the previous PC pointer value.

**Register File**: Small storage device that consists of a collection of word-sized registers, each with its own name.

**Program Counter (PC)**: Within the CPU, A storage device (register) that points at the some machine-language instruction in main memory.

**Arithmetic Logic Unit (ALU):** Performs arithmetic operations on one or two registers and returns an output.

### Caches
A lot of copying is taking place to get the program from disk to main memory to the processor. 

**Processor-Memory Gap**: CPUs execute instructions really quick but getting those instructions isn't as fast.

**Cache Memories** is a way to solve this issue. They serve as temporary staging areas for info the CPU may need soon. Implemented by _Static Random Access Memory (SRAM)_. Static because it keeps the memory as long as it has power.

- **L1 Cache** is nearly as fast as using a register.
- **L2 Cache** is larger yet slower and connected to the CPU via a special bus.

**Locality**: Tendency of a CPU to access the same set of memory locations more than once within a localized region.

### Operating Systems
The Operating System is the layer that lives between our application and the hardware, and if we mean to communicate between them, we must go through the OS.

The operating system has two main purposes:
1. Protect the hardware from misuse.
2. Provide the application with simple and uniform mechanism for manipulating low-level hardware devices.

#### Processes
An abstraction for a running program. By concurrently, we mean that the instructions of one process are interleaved with the instructions of another process. This is called _context switching_. 

>![[Pasted image 20231202145654.png|  The OS switching between the shell and the running program.]]


**Kernel**: It manages this context switching. It is NOT a separate process. Instead it's a collection of code and data structures that the system uses to manage all processes. 

**Systems Calls**: Made by the processes to interact with the OS and the kernel handles this. This is things like printing, read, writing, etc. This is typically stuff that requires higher privileges that the program can't do on it's own.

#### Threads
A process can have multiple execution units aka threads. They share the code and global data of the process.

#### Virtual Memory
The space provided for a running process. Stuff like stack, heap, and read-only memory. Notice that the kernel gets it's own space, since it may need to interact with the program to make system calls.

> ![[Pasted image 20231202160358.png]]

#### Files
A sequence of bytes. All I/O devices are modeled as files. 

### Communication via Networks
Modern systems are often linked to other systems via networks. Think of a network as just another I/O device from which data can flow from one machine to another.

> ![[Pasted image 20231202163247.png]]

### Important Themes
#### Amdahl's Law
To significantly speed up the entire system, we must improve the speed of a very large fraction of the overall system.

#### Concurrency and Parallelism
**Concurrency**: A system with multiple, simultaneous activities.
**Parallelism**: The use of concurrency to make a system run faster.

#### Thread-Level Concurrency
**Multi-core** processors have several CPUs (or cores) integrated into one circuit chip.

> ![[Pasted image 20231202164547.png]]

**Hyperthreading** also called _simultaneous multi-threading_ that allows a single CPU to execute multiple flows of control. 

#### Instruction-Level Parallelism
Modern processors can execute multiple instructions at one time, a property known as instruction-level parallelism. 

**What is a clock cycle and how fast is it?**
It's like the heartbeat of the computer. Each tick allows the processor to perform a task or part of a task.

The speed of a clock cycle is measured in hertz (Hz), with modern CPUs typically operating in the gigahertz (GHz) range, which is billions of cycles per second. For example, a 3 GHz processor has a clock cycle that occurs 3 billion times per second.

#### Single-Instruction, Multiple-Data (SIMD) Parallelism
Special hardware that allows a single instruction to cause multiple operations to be performed in parallel. Like having an instruction that can add 8 pairs of single precision floating point numbers in parallel. SIMD instructions are mostly to speed up apps that process images, sound or video data.