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



# Paging

### 四种paging模式？？？

计算机发展到现在（x86体系），paging模式已经有了四种，分别是32-bit paging、PAE paging、4-level paging、5-level paging。我们可以将其做粗略划分，前两种模式是应用于32位平台，后两种模式是应用于64位平台。



# Service

- SSH: default to 22 port, *sshd* stand for SSH daemon and is the server-side process for SSH, configuration in `/etc/ssh/sshd_config`

