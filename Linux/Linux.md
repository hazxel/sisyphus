Unix哲学KISS：keep it simple, stupid。在Linux系统里，一切看上去十分复杂的逻辑功能，都用简单到不可思议的方式实现，甚至有些时候看上去很愚蠢。但仔细推敲，人们将会赞叹Linux的精巧设计，或许这就是大智若愚。




# Process

### create process：

##### 表层系统调用：

- fork: copy all the resources of father

  > Copy-on-write (COW): 为了降低开销，fork最初并不会真的产生两个不同的拷贝，因为在那个时候，大量的数据其实完全是一样的。写时复制是在推迟真正的数据拷贝。若后来确实发生了写入，那意味着parent和child的数据不一致了，于是产生复制动作，每个进程拿到属于自己的那一份，这样就可以降低系统调用的开销。所以有了写时复制后呢，vfork其实现意义就不大了。 

- vfork: parent and child share the same address space

- clone: a call with parameter (clone_flags). Can select partial resource of parent to share with child

##### Process Creation Details

进程的创建主要通过系统调用 `execl`, `execle`或`execv`，读取 ELF文件，把其中的代码和数据部分放到链接时指定的内存位置，然后从指定的开始地址执行代码。

1. 创建一个独立的虚拟地址空间
2. 读取可执行文件头，少部份 segment 直接加载进内存，其余 segment 建立虚拟空间与可执行文件的映射，后续通过缺页中断加载进内存。
3. 将CPU的指令寄存器设置成可执行文件的入口地址，启动运行

### Process Address Space Management

##### Virtual memory: 32-bit vs 64-bit

On 32-bit architecture, the highest 1GB of virtual memory is kernel space, and the lowest 3GB is user space. 内核虚拟内存的前 896M 区域被称为直接（线性）映射区，包含 DMA区、NORMAL区，会连续的映射到物理内存中最低 896M 区域的 ZONE_DMA 和 ZONE_NORMAL 中，偏移为 PAGE_OFFSET=3GB。当然内核还是会使用虚拟地址访问该内存。（X86 下 ISA 总线的 DMA 控制器，只能对内存的前16M 进行寻址）

这样，内核虚拟内存空间还剩128M，用于映射并访问物理内存中 3200M 大小的 ZONE_HIGHMEM 区域。由于剩余空间过小，需要采用动态映射：某段物理内存和这128MB的内核虚拟空间建立映射并完成所需操作后，需要断开与这部分虚拟空间的映射关系，以便映射其他物理内存。

64位地址空间达到了 16EB，大部份操作系统和应用程序完全不需要这么大的空间。目前 x64 仅使用 48 位用于address translation (page table lookup)，剩下的高 16 位留作为符号扩展。而在ARM 体系中，以ARMv8-A为例，它的页大小和页表层级有多种组合。采用 4KB 页，4级页表时，虚拟地址也为48位。

48位虚拟地址空间的范围为 256TB ，内核空间和用户空间各占128TB。此时用户虚拟空间和内核虚拟空间不再是挨着的，但用户空间仍在底部，内核空间占据顶部。kernel space的高17位都为1，user space的高17位都为0，此时用户空间和内核空间地址被称为 canonical address。

>  **canonical address**: the most significant 16 bits of any virtual address, bits 48 through 63, must be copies of bit 47. 也就是最高的17位必须相同，对于内核空间就全是1，用户空间全是0，而他们之间的偌大的区域为 non-canonical address，访问是非法的。

在64位系统中内核的虚拟地址空间足够大，可以直接映射所有物理内存，不再需要动态映射。

##### VMA (Virtual Memory Area)

OS 使用 VMA 来对进程的地址空间进行管理，包括被映射 ELF 中的各个Segment，管理运行时的堆和栈等。堆栈这种没有指定特定的映像文件的 VMA 通常被称为匿名虚拟内存区域 (Anonymous Virtual Memory Area, AVMA)

划分VMA的基本原则是将相同权限属性的，有相同映像文件的映射成一个VMA，一个进程基本上可以分为如下几种VMA区域：

- CODE VMA：可读、可执行： (`.text`, `.rodata`, ...)
- DATA VMA：可读写、可执行；有映像文件 (`.data`, ...)
- BSS VMA：可读写；无映像文件，匿名 (`.bss`, ...)
- HEAP VMA：可读写、可执行；无映像文件，匿名，可向上扩展
- STACK VMA：可读写、不可执行；无映像文件，匿名，可向下扩展

一个进程通常由加载一个elf文件启动，而elf文件是由若干segments组成的，而进程地址空间也由许多不同属性的segments组成。ELF 将读写执行权限相同的 Sections 合并为 Segment，然后将Segment作为一个整体映射到虚拟内存空间和实际物理空间，一个Segment对应一个 VMA。（如果依赖的动态库多，segments数量会很大）（这与硬件的 segmentation 机制不是同一个概念！）

一个 VMA 由许多的虚拟内存页组成，并按起始地址递增存放于双向链表（早期内核使用单向链表）。VMA 同时还通过红黑树组织起来，以加速通过虚拟地址对 VMA 的查找 (e.g. page fault)。

##### 段地址对齐

可执行文件被OS装载运行时，一般是通过虚拟内存的页映射机制，以页为单位装载指令和数据。也就是说，要映射的内存长度必须是整数个页，且这段空间在物理内存和进程虚拟地址空间的起始地址必须也是页大小的整数倍。

首先明确一点，不同Segment在虚拟地址空间中不能在一个页中有交集，因为不同 segment 权限不同，需要依靠虚拟地址做访问权限控制，因此产生了如下两种方案：

- 最简单的方案：段长度不足一页时补齐一页，然后分别映射。缺点：页对齐产生的内存碎片浪费磁盘空间。
- UNIX解决方案：段地址对齐 - 各个段在物理内存中紧凑排列，接壤部分的物理页面分别映射到两个（甚至多个）虚拟页面。这也意味着各个段的虚拟地址不再是系统页面长度的整数倍了。

### process status

- R：running，教科书一般将正在CPU上执行的进程定义为RUNNING状态、可执行但是尚未被调度执行的进程定义为READY状态，这两种状态在linux下统一为 TASK_RUNNING状态。
- Z：僵尸状态，进程在退出的过程中，处于TASK_DEAD状态。 在这个退出过程中，进程占有的所有资源将被回收，除了task_struct结构（以及少数资源）以外。于是进程就只剩下task_struct这么个空壳，故称为僵尸。
- S (TASK_INTERRUPTIBLE)，可中断的睡眠状态。
- D (TASK_UNINTERRUPTIBLE)，不可中断的睡眠状态。
- T (TASK_STOPPED or TASK_TRACED)，暂停状态或跟踪状态
- X：退出状态，进程即将被销毁

> ### Inverview question: Process vs Thread
>
> - Context Switch: Switching processes is super expensive:
>   - Process switching uses an interface in an operating system, while swithing/creating threads doesn't require a kernel call.
>   - Process switching involves:
>     - Loading the corresponding Process Control Block (PCB) from the PCB table (in kernel stack). 
>     - Loading CPU state info (registers) and memory management info (segmentation tables, page tables) from PCB.
>     - Flush Translation Lookaside Buffer (TLB), to ensure correct virtual-pyhsical address translation
>   - Thread switching involves much smaller context: 
>     - thread program counter, registers, stack pointers
> - Communication: Threads in the same process share the same virtual memory space, so thread communication can be as simple as sharing a variable or object.However, processes's memory are usually isolated, and requires Inter Process Communication (IPC), which is not very efficient. 
> - Isolation: Processes are usually isolated, while a crash in a thread takes down the whole process.
>
> 一次起500个线程 会怎样？崩溃？



# Paging

### 四种paging模式？？？

计算机发展到现在（x86体系），paging模式已经有了四种，分别是32-bit paging、PAE paging、4-level paging、5-level paging。我们可以将其做粗略划分，前两种模式是应用于32位平台，后两种模式是应用于64位平台。

### Memory Management

##### Segmentation

- **Segments**: A program is usually divided to logical segments (e.g. global data segment, main function, etc). Inside a segment, address starts from 0, and is stored continuously in physical memory.
- **Logical Address**: composed of segment id and offset within the segment
- **Segment Table**: map logical address to physical address. also segment limit
- **Segmentation selector**: identify segment in either **Global Descriptor Table** or  **Local Descriptor Table.** GDT is the global level (first level), LDT is the local level (second level)
- **Segmentation discriptor**:  describe the address, size and access control info of segment

##### **Paging**

- **Page Table**: **Page Table Entries (PTE)** hold the mapping between **Physical Frame Number (PFN)** and **Virtual Page Number (VPN)**, and other kinds of info.  
- Paged memory functionality is usually hardwired into **Memory Management Unit (MMU)**, which translates virtual addresses to physical address by looking up page table.

> Multi-level Page Table: single level page table can be very big

> Table Lookaside Buffer (TLB): a physical cache in CPU, storing the most frequent accessed  Page Table Entry (PTE)

##### Segmentation + Paging

- **Segment Table**: map segment number to page table address
- **Page Table**: map page number to Physical Frame Number

##### Linux Memory Management

>  In x86 architecture, segmentation is compulsory, while paging is optional. Paging and segmentation are redundant in some ways. So, Linux finds out a way to work around without engaging too much with segmentation.

linux中逻辑地址等于线性(虚拟)地址，因为Linux所有的段（用户代码段、用户数据段、内核代码段、内核数据段）的线性地址都是从 0x00000000 开始，这样 线性地址=逻辑地址+ 0x00000000，逻辑地址等于线性地址了。所以 Linux 操作系统，并没有使用到全部的分段功能。但分段可以做权限审核和访问越界检测。





# Service

- SSH: default to 22 port, *sshd* stand for SSH daemon and is the server-side process for SSH, configuration in `/etc/ssh/sshd_config`



# Users and Groups

### user

Linux user information is in `/etc/passwd`

### group

linux group information is in `/etc/group`

### permission

In Linux, file permissions are represented numerically with a three-digit octal value. Each digit corresponds to a different category of users:

1. **Owner**: The permissions for the file's owner (user).
2. **Group**: The permissions for the group that the owner belongs to.
3. **Others**: The permissions for all other users.

Each digit in the octal representation is an octal (base-8) number, which translates to a 3-bit binary number. This binary number represents the permissions as follows:

- **Read (r)**: The permission to read the file. (1 in the binary representation)
- **Write (w)**: The permission to modify the file. (2 in the binary representation)
- **Execute (x)**: The permission to execute the file. (4 in the binary representation)



# syscall

系统调用在 Linux 内核中实现，他们的定义可以在 Linux 内核源代码中找到。系统调用通常通过 `::syscall` 接口进行访问，而不是通过标准的 C 库或用户空间库头文件直接定义。可以在 `/usr/include/bits/syscall.h` 找到 syscall 的定义。

### io_uring

3 related syscall: 

- `io_uring_setup`: 初始化并分配内存
  - `u32 entries`: the minimum size of SQ and CQ
  - `struct io_uring_params *p`: contains options passed to kernel, other fields filld by kernel
  - return an `int` fd for subsequent operations on the io_uring instance
- `io_uring_enter`: 提交请求到内核执行
- `io_uring_register`:  注册文件描述符或缓冲区，使其可在 I/O 操作中被引用，避免重复传递，并且由于已提前 mmap，也能减少延迟

encapsulation & helper provided by liburing: 

- `io_uring_queue_init`: calls `io_uring_setup` and xxx ???
- `io_uring_submit`: calls `io_uring_enter`
- `io_uring_prep_*`: prepare SQE，包含所有异步 I/O 操作需要的信息。
- `io_uring_submit`: 一般仅更新 SQ，某些情况下 call `io_uring_enter` 来唤醒内核线程 `io_sq_thread` 
- `io_uring_wait_cqe`: block and loop to wait for a new CQE (no sys call)
- `io_uring_cqe_seen`: update CQ head, kernel will handle processed entries (no syscall)
- `io_uring_register_files`, `io_uring_register_eventfd`: calls `io_uring_register`
- `io_uring_queue_exit`: `munmap` 解除提交队列和完成队列的共享内存内存映射，`close` 关闭 ring 的文件描述符，（io_uring 实例本身似乎是内核负责释放？？？）



# POSIX

POSIX（Portable Operating System Interface）是一个标准化的操作系统接口规范，由 IEEE（Institute of Electrical and Electronics Engineers）制定。它旨在提高操作系统之间的兼容性，使得程序能够在不同的操作系统上移植和运行，而无需修改代码。POSIX 定义了操作系统的 API、工具和接口，使得在遵守该标准的系统上编写的程序可以更容易地迁移和运行。

> Ubuntu 和 CentOS：作为 Linux 系统的代表，广泛支持 POSIX 标准及其扩展。
> macOS：良好地支持 POSIX 标准，大多数基础功能和扩展都被实现，但其文件系统和系统调用有自己的实现和扩展。
> Windows：无原生 POSIX 标准支持，但可以通过 WSL、Cygwin、MinGW 等工具或子系统提供 POSIX 兼容的环境。

### Fork

- `fork`: create a new process. If success, returns 0 to the child process and returns the PID of the child process to the parent process.  Otherwise return -1 to the parent process, no child process is created, and the global variable errno is set to indicate the error.
- `_exit(0)`: terminate the calling process (`0`表示正常退出)
- `wait(nullptr)`: for parent process to wait a child process' termination(either one)
- `waitpid`: wait for a specific child process' termination

### Anonymous semaphore

Anonymous semaphore: used within a process to sync threads

Its lifecycle is controlled by the process. allocated within the process's memory, won't affect IPC namespaces.

- `sem_init`: initialize an unnamed semaphore
- `sem_destroy`: destroy an unnamed semaphore
- Delete: anonymous semaphore doesn't need explicit delete 

### Named semaphore: can be shared between processes

Named semaphore is given a name ，命名信号量在文件系统的 IPC 虚拟命名空间中注册，这使得不同进程可以通过名字访问同一个信号量。(类似于在文件系统中创建一个文件)

- `sem_open`: initialize and open a named semaphore, 一般选一个进程来open，确保其他进程仅打开
  - `name`: the identifier of the semaphore, usually start with `/`, e.g. `"/my_semaphore"`
  - `oflag`: specifies flags that control the operation of the call.  frequently used flags:
    - `O_CREAT`: created if doesn't exist 
    - `O_EXCL`: If both`O_CREAT` and `O_EXCL` are specified in oflag, then an `EEXIST` error is returned if a semaphore with the given name already exists.
  - `mode`: permissions settings for the semaphore, must supply if oflag includes `O_CREAT`
  - `value`: initial value for the semaphore, must supply if oflag includes `O_CREAT`
  
  returns the address of the new semaphore  `sem_t *` on success, otherwise `SEM_FAILED`
- `sem_close`:  关闭当前进程对该命名信号量的描述符。其他进程仍可通过其信号量描述符访问该信号量
- `sem_unlink`: 删除命名信号量，实质上的删除在所有打开该信号量的进程关闭它后触发。首次删除后，后续删除不会改变系统状态。
- `sem_wait`: decrements (locks) the semaphore.
- `sem_post`: increments (unlocks) the semaphore

### share memory

- `shm_open`: creates a new, or opens an existing shared memory object 实际的内存尚未分配

  - `name`: the identifier of the shared memory object, usually start with `/`, e.g. `"/my_shm"`
  - `oflag`: specifies flags that control the operation of the call.  frequently used flags:
    - `O_CREAT` & `O_EXCL`: similar to semaphore
    - `O_RDONLY` & `O_RDWR` Open the object for *read-only* or *read-write* access.
    - `O_TRUNC`: If the shared memory object already exists, truncate it to zero bytes.
  - `mode`: permissions settings for the shared memory

  returns **a file descriptor** referring to the shared memory object with `FD_CLOEXEC` flag set, and is normally used in subsequent calls to `ftruncate` (if newly created object) and `mmap`.  After a call to `mmap`, the file descriptor may be closed.

- `ftruncate`: truncate a file to a specified length

- `mmap` & `munmap`: map or unmap files or devices into memory

- `close`: close a file descriptor

- `shm_unlink`: removes a shared memory object name, and, once all processes have unmapped the object, deallocates and destroys the contents of the associated memory region.

### pipe

### pthread

### Others

- `sleep`: 影响整个进程，所有线程全部挂起。现代 c++ 中建议使用 ``std::this_thread::sleep_for` 以确保更好的跨平台兼容性和代码的可维护性。

> ### Interview question: IPC pros and cons
>
> Inter-process communication is a mechanism provided by the OS for communications between several processes.
>
> > Transmission Modes:
> >
> > - simplex: Only one of the two parties on a link can transmit, the other can only receive.
> > - half-duplex: each party can both transmit and receive, but not at the same time.
> > - full-duplex: both parties can transmit and receive simultaneously.
>
> - Pipe: half-duplex
>
>   A pipe is an important mechanism in Unix-based systems that allows us to communicate data from one process to another without storing anything on the disk. The following two kinds of pipes are very similar except for creating and deleting.
>
>   - Named Pipe (FIFO):
>
>     A named pipe can last as long as the system is up. Usually a named pipe appears as a file, and generally processes attach to it for IPC.
>
>     - Advantages: allows communication between unrelated processes
>     - Disadvantages: Long-term storage in the system, improper use is prone to errors. Limited buffer
>
>   - Anonymous pipe (pipe): `|`
>
>     Typically a parent program opens anonymous pipes, and creates a new process that inherits the other ends of the pipes, or creates several new processes and arranges them in a pipeline. When no process is holding a reference to a pipe, it will be closed automatically.
>
>     - Advantages: simple and convenient
>     - Disadvantages: Can only be created between its processes and their related processes. Limited buffer
>
> - Semaphore: a **counter** that can be used to control access to shared resources by multiple threads.
>
>   P and V are atomic operations, P(sv) will decrease sv by 1 and suspend if sv=0, while V(sv) will increase sv by 1 and invoke the suspended process.
>
>   - Advantages: can synchronize processes
>   - Disadvantage: just a counter, not very expressive
>
> - Signal: The only asyncronous communication in IPC.  It's complex and used to notify the process that an event has occurred. It's like the software-level interrupt. A process receiving a signal is like a processor receiving an interrupt.
>
> - Message Queue: a linked list of messages, stored in the kernel and identified by the message queue identifier
>
>   A data stream similar to a socket, but which usually preserves message boundaries. Typically implemented by the operating system, they allow multiple processes to read and write to the message queue without being directly connected to each other.
>
>   - Advantages: Easy to implement. Allow communication between any process. Send and Receive through system call, and no need to consider synchronization issues.
>   - Disadvantages: Copying information requires additional CPU time, which is not suitable for situations with large amounts of information or frequent operations
>
> - Shared Memory: A common memory created in RAM, and can be accessed by multiple processes.
>
>   - Advantages: fastest, bidirectional, large amount of information, can be used by multiple processes
>   - Disadvantages: Require concurrency control (e.g. semaphore, mutex)
>
> - Socket: can be used for process communication between different computers
>
>   Socket 是对 TCP/IP 协议族的一种封装，是应用层与TCP/IP协议族通信的中间软件抽象层。从设计模式的角度看来，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。
>
>   - Advantages:
>     1. The transmission data is byte level, the transmission data can be customized, the data volume is small and the efficiency is high
>     2. Short data transmission time and high performance
>     3. Suitable for real-time information exchange between client and server
>     4. Can be encrypted, strong data security
>   - Disadvantages: The transmitted data needs to be parsed and converted into application-level data.
