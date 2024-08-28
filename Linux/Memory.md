# Address

- logical address: composed of segment id and offset within the segment. 在机器语言指令中用来指定**操作数**或**指令**的地址，由段和偏移量组成。是程序实质上看到的地址

- virtual address/linear address：操作系统提供的一个抽象地址空间，(32-bit arch allow 4GB) 

  > 在一些远古操作系统中，逻辑地址由段地址和偏移量组成，而虚拟地址则需要经过内存控制单元 (MMU) 的分段单元 (segmentation unit) 转换得到。在 Linux 和大多数现代操作系统中，虚拟地址和逻辑地址是相同的。

- physical address：芯片上的内存单元的地址，经过 MMU 的分页单元 (paging unit) 由线性地址转换而来



# Segmentation

分段单元会将逻辑地址转换为虚拟地址。分段机制是个比较落后的技术，过去工程师们为了解决硬件限制而做出的妥协方案。在8086处理器诞生之前，内存寻址方式就是直接访问物理地址，也就是所谓的**实模式**。8086处理器为了寻址1M的内存空间，把地址总线扩展到了20位。但是，ALU的宽度只有16位，也就是说，ALU不能计算20位的地址。为了解决这个问题，从而引入了分段机制。

### Segments

A program is usually divided to logical segments (e.g. data seg, stack seg, etc). Inside a segment, address starts from 0, and is stored continuously in physical memory. 

### Segment selector

大小为 16 bits，存放在处理器的段寄存器中，内容包括：

- Index: GDT 或 LDT 中的索引值，占 13 位。
- TI (Table Indicator): 指示使用的是 GDT 还是 LDT，占 1 位。

- RPL (Request Privilege Level): 请求特权级别，表示当前任务的权限级别，占 2 位。？？？

### Segment discriptor

大小为 8 Bytes，存放在段描述符表中，内容包括段基地址、限长、特权级别等

> 为了加速逻辑地址到线性地址的转换，x86处理器提供一种非编程的寄存器 (不能被程序员所设置的寄存器)，供 6 个可编程的段寄存器使用。每当一个段选择符被装入段寄存器时，相应的段描述符中的基地址、限长等属性就由内存装入到对应的非编程寄存器，此后针对那个段的逻辑地址转换可不访问主存。

### Segment discriptor table 段描述符表

- GDT 全局描述符表：毎个 CPU 对应一个 GDT，在系统范围内有效，通常由操作系统定义，用于存储常见的段，如内核代码段、内核数据段、用户代码段、用户数据段等。GDT 对所有进程是共享的。
- LDT 局部描述符表： 由每个进程自己定义，允许每个进程创建和使用自己特定的段描述符。LDT 提供了更灵活和细粒度的内存管理选项。大多数用户态进程不使用 LDT，内核定义了一个缺省的 LDT 供大多数进程共享。

### Linux Memory Management

In x86 architecture, segmentation is compulsory, while paging is optional. Paging and segmentation are redundant in some ways. So, Linux finds out a way to work around without engaging too much with segmentation.

Linux 主要采用分页机制（X86叫保护模式，arm叫MMU机制）来对用户态与内核态进行隔离，也对进程与进程之间进行隔离。无奈在X86架构下，使用分页机制前，必须打开分段机制。所以Linux采用了讨巧的办法，“哄骗”硬件来绕过分段机制（以32位为例）：令每个段的线性地址都是从 0x00000000 开始，限长都为 4GB，使 虚拟地址=逻辑地址+0x00000000，即逻辑地址等于虚拟地址。剩下的管理全由分页机制来实现，仅使用分段做权限审核和访问越界检测。



# Paging

分页单元会将线性地址转换为物理地址。

### Page

线性地址被分成以固定长度为单位的组，称为 page，其内部连续的线性地址被映射到连续的物理地址。页内的所有地址的存取权限也相同。页大小的典型值是 4 KB，但也有称为大页 (Huge Page) 的 2 MB 的页，以及 1 GB 的超大页 (Gigantic Page)。一般 “页” 既指一组线性地址，又指包含在这组地址中的数据，是个比较抽象的概念。

### Page Frame 页框

分页单元把 RAM 分成固定长度的 page frame，其长度与页的长度一致，毎一个页框可以存放一个页。页框指的是一个储存区域，是个比较具体的概念。

System uses a `struct page` as a "descriptor" to keep track of a page frame. 大小为 32 字节，描述了页框是否空闲，所属内核还是用户等。所有描述符存放于 `mem_map` 数组中，可通过  `pfn_to_page(pfn)`  从物理页框号 PFN 得到页描述符地址 。`struct page` 中存在 lru 链表指针字段用于 PFRA 页框回收。

内核的**分区页框分配器** (zoned page frame allocator) 可负责处理对连续页框的分配和释放请求。

### Page Table

页表负责把线性地址映射到物理地址，存放在主存中，由内核进行管理。每个进程的用户态页表是独立的，但内核空间通常是统一的。

如果为每个页都保存一个 4B 的页表项，仅 4GB 的线性地址空间，每个页表就需要占用 4MB 内存。因此，操作系统一般会使用多级页表，一开始只维护上层的表，仅当进程实际需要一个页表时才给该后续层级的页表分配内存。（32位系统一般两级就足够，64位 linux 采用三级或四级）

- **Physical Frame Number (PFN)**: a unique index to identify a physical page frame

- **Virtual Frame Number (VFN)**: index for a page in virtual memory address space

- **Page Table Entries (PTE)** hold the mapping between **Physical Frame Number (PFN)** and **Virtual Page Number (VPN)**, and other kinds of info.  

  > **Memory Management Unit (MMU) & Table Lookaside Buffer (TLB)**: hardware that translates virtual addresses to physical address by looking up page table. 详见 Architecture 章节。

  PTE 是操作系统软件层面的概念，但必须按照处理器定义的格式去填充，因为 MMU 在地址翻译的过程中需要根据 PTE 的标志位来检测访问是否合法等。

  但处理器不能自动同步它们自己的TLB高速缓存，因为决定线性地址和物理地址之间映射何时不再有效的是内核，而不是硬件。处理器侧，以 Intel 为例，要触发 invalidate TLB，可以向 cr3 寄存器写入值，或调用汇编指令 `invlpg` 。另一方面， Linux 内核则提供了比较丰富的 TLB 方法 `flush_tlb_xxxx`。

- xxx

### paging models

目前主流的 paging 模式有 32-bit paging、PAE paging、4-level paging、5-level paging 等。前两种模式一般应用于32位平台，后两种模式应用于64位平台。

##### Physical Address Extension (PAE)

32位系统理论上可以安装 4GB 的 RAM，但为了支持更大的 RAM，Intel 将其地址管脚从 32 个增加到 36 个，这就需要操作系统引入一种新的分页机制，把 32 位线性地址转换为 36 位物理地址。为了支持该功能，Intel 处理器引入了页地址扩展 (PAE) 和页大小扩展 (PSE, Page Size Extension) 两种机制，而 linux 只采用了 PAE 机制。

##### 4-level paging

x86-64架构下的四级页表层级通常包括：页全局目录（PML4）、页上级目录（PDPT）、页目录（PD）、页表（PT），而每一级页表都包含指向下一层页表或物理页面的指针。



# Process Address Space Management

##### VMA (Virtual Memory Area) / memory region 线性区

OS 使用线性区（又叫 VMA）来对进程的线性地址空间进行管理，包括映射 ELF 中的各个Segment，管理运行时的堆栈等。VMA 由起始地址、长度和访问权限来描述，起始地址和线性区的长度都必须是4096的倍数， 以便完全利用分配给它的页框。

**磁盘文件系统的普通文件的某一部分**或者**块设备文件**可以被关联到一个线性区，系统把对区线性中页的访问转换成对文件的访问，这种技术称为内存映射。`mmap` 系统调用被用于创建一个内存映射，该函数需要传递以下参数:

- 文件描述符：传递 `-1` 则分配匿名内存，表示不与任何实际文件关联，内存会被初始化为零 (e.g. malloc)
- 文件内偏移量
- 映射长度
- 内存映射种类：shared 为读写，private 只读，anonymous 匿名，可组合使用
- 权限：read/write/execute
- 线性区地址：可选

相同权限属性的，相同映像文件的文件一般会被映射到同一个 VMA，而堆栈这种没有指定特定的映像文件的 VMA 通常被称为匿名虚拟内存区域 (Anonymous Virtual Memory Area, AVMA)。

> 一个进程通常由加载一个 ELF 文件启动。 ELF 文件是由若干 segments 组成的，进程地址空间也由许多不同属性的 segments 组成。ELF 将读写执行权限相同的 Sections 合并为 Segment，然后将 Segment 作为一个整体映射到虚拟内存空间和实际物理空间，一个 Segment 对应一个 VMA。（如果依赖的动态库多，segments数量会很大）（这里的 segments 与分段 segmentation 机制不是同一个概念！），一个进程一般会有如下几种 VMA 区域：
>
> - CODE VMA：可读、可执行： (`.text`, `.rodata`, ...)
> - DATA VMA：可读写、可执行；有映像文件 (`.data`, ...)
> - BSS VMA：可读写；无映像文件，匿名 (`.bss`, ...)
> - HEAP VMA：可读写、可执行；无映像文件，匿名，可向上扩展
> - STACK VMA：可读写、不可执行；无映像文件，匿名，可向下扩展

##### memory descriptor 内存描述符

内存描述符包含与进程地址空间有关的全部信息，类型为 `mm_struct` ，系统中所有的内存描述符统一存放在一个双向链表中，进程描述符的 `mm` 字段会指向该进程的内存描述符。内存描述符包含 `mm_count` 字段，用于计数使用者，当其递减时检查其是否为零并删除之。

（用户进程的）内存描述符的一个重要功能是记录线性区，每个 VMA 都对应一个 `vm_area_struct` 数据结构。线性区不会重叠，内核会尽量把新线性区紧邻的现有线性区，并根据访问权限进行合并。进程所拥有的所有线性区通过双向链表（早期内核使用单向链表）链接在一起，并按地址升序排列。为了应对线性区数量过多的场景，Linux 2.6 中新增了红黑树用于存放线性区，与链表结构同时存在，红黑树用来查找含有指定地址的线性区，链表用于扫描整个线性区集合。

内核对进程内存分配采用**推迟策略**，进程请求内存时，不直接获得页框，仅获得 VMA 的使用权。Linux 的缺页异常（Page Fault）处理程序会利用 `vm_area_struct` 来区分编程错误引起的非法访问，以及由于物理页框未分配引起的异常。主要检查线性地址是否属于该进程地址空间，以及访问权限是否匹配。在处理对磁盘文件进行映射的线性区时，会首先在 page cache 中查找所请求的页，如果没有找到则必须从磁盘上读取。

##### Virtual memory: 32-bit

On 32-bit architecture, the **highest 1GB** of virtual memory is **kernel space**, and the lowest 3GB is user space. 内核虚拟内存的前 896M 区域被称为直接（线性）映射区，包含 DMA区、NORMAL区，会连续的映射到物理内存中最低 896M 区域的 ZONE_DMA 和 ZONE_NORMAL 中，偏移为 PAGE_OFFSET=3GB。当然内核还是会使用虚拟地址访问该内存。

> - **ZONE_DMA**：大小为 16MB，因为 X86 下 ISA 总线的 DMA 控制器，只能对物理内存的前 16MB 进行寻址
> - **ZONE_NORMAL**：紧接在DMA区之后，通常占据 16MB 到 896MB 之间的物理内存区域。提供给内核动态分配，如页表、进程控制块（PCB）、内存管理结构（如slab分配器）、中断处理程序所需的内存等

这样，内核虚拟内存空间还剩128M，用于映射并访问物理内存中多达 3200M 的高端内存 ZONE_HIGHMEM 区域。由于剩余空间过小，需要采用动态映射：某段物理内存和这128MB的内核虚拟空间建立映射并完成所需操作后，需要断开与这部分虚拟空间的映射关系，以便映射其他物理内存。

##### Virtual memory: 64-bit

64位地址空间达到了 16EB，大部份操作系统和应用程序完全不需要这么大的空间。目前 x64 仅使用 48 位用于address translation (page table lookup)，剩下的高 16 位留作为符号扩展。而在ARM 体系中，以ARMv8-A为例，它的页大小和页表层级有多种组合。采用 4KB 页，4级页表时，虚拟地址也为48位。

48位虚拟地址空间的范围为 256TB ，内核空间和用户空间各占128TB。此时用户虚拟空间和内核虚拟空间不再是挨着的，但用户空间仍在底部，内核空间占据顶部。kernel space的高17位都为1，user space的高17位都为0，此时用户空间和内核空间中间地址被称为 canonical address。

>  **canonical address**: the most significant 16 bits of any virtual address, bits 48 through 63, must be copies of bit 47. 也就是最高的17位必须相同，对于内核空间就全是1，用户空间全是0，而他们之间的偌大的区域为 non-canonical address，访问是非法的。

在64位系统中内核的虚拟地址空间足够大，可以直接映射所有物理内存，不再需要动态映射。

##### 段地址对齐

可执行文件被 OS 装载运行时，一般是通过虚拟内存的页映射机制，以页为单位装载指令和数据。也就是说，要映射的内存长度必须是整数个页，且这段空间在物理内存和进程虚拟地址空间的起始地址必须也是页大小的整数倍。

首先明确一点，不同Segment在虚拟地址空间中不能在一个页中有交集，因为不同 segment 权限不同，需要依靠虚拟地址做访问权限控制，因此产生了如下两种方案：

- 最简单的方案：段长度不足一页时补齐一页，然后分别映射。缺点：页对齐产生的内存碎片浪费磁盘空间。
- UNIX解决方案：段地址对齐 - 各个段在物理内存中紧凑排列，接壤部分的物理页面分别映射到两个（甚至多个）虚拟页面。这也意味着各个段的虚拟地址不再是系统页面长度的整数倍了。


##### heap 堆
每个 Unix 进程都拥 一个特殊的线性区，堆(heap)。内存描述符的 start_brk 与 brk 字段限定了堆的起止地址。

- start_brk: 确定的堆的起始地址，位于 bss 段结束地址后（开启 ASLR 保护时增加一个 random brk offset）
- brk (program break): 标志堆的结束地址，会动态改变，小于 brk 的地址空间都是已经分配的堆内存。程序启动时，brk 指针被设置在 start_brk，此时堆的大小为 0

常用的分配动态内存的 C 接口，如 malloc、calloc、realloc，都是通过系统调用 brk 和 mmap：

- brk 调用：传入一个地址，做检查后直接以此更新 program break location。brk 分配的内存需要等到高地址内存释放以后才能释放，容易产生内存碎片，glibc 中只有小于 128KB 的内存请求使用 brk 分配
- mmap 调用：直接从堆栈区寻找一块空闲的地址进行分配，mmap分配的内存可以单独释放

有关 malloc 等函数的具体实现，详见 C++/memory 章节



# Buddy System 伙伴系统

Linux 使用名为伙伴系统的内存分配算法管理物理内存的分配和回收。伙伴系统将内存分割成块，每个块的大小都是 2 的幂次方，最小时通常是一页（4 KB），最大时则取决于整个系统的内存规模。

当需要分配一块内存时，伙伴系统会找到最小的足够容纳请求大小的块。如果找到的块太大，它会被拆分成两个伙伴块，并继续尝试在较小的块中找到合适的内存。当块被释放时，系统会尝试将该块与其伙伴合并（如果伙伴也空闲），以减少内存碎片。

系统使用多个链表来管理大小相同的块，这些链表存放于 free_area 数组中。当内存块被释放时，系统会检查它的伙伴块是否也空闲。如果是，系统会将这两个块合并为一个更大的块，并将其返回到更高一级别的链表中。



# 回收页框

当系统负载较低时，大部分内存由 page cache 即磁盘缓存占用。当系统负载增加时，需要缩小 page cache 从而给进程页让出空间。这需要内核从用户态进程和内核高速缓存“窃取” 页框，以补充伙伴系统的空闲块列表。

### 页框回收算法 (page frame reclaiming algorithm, PFRA)

PFRA 的目标是获得非空闲的页框并释放，分类大致为：

- 不可回收页：内核页/保留页/空闲页 等不允许回收或无需回收的页
- 可交换页：匿名页或 tmpfs 的映射页 (如 IPC 共享内存的页)，可交换至磁盘
- 可同步页：file backed pages，可会写到磁盘文件后释放页框
- 可丢弃页：一些已经分配但暂未使用的页，如 SLAB cache，文件系统目录项高速缓存 dentry cache 等

PFRA 回收的基本原则：

- 首先释放“无害”页，即没有被任何进程使用的页。这不需要修改任何页表
- 优先回收非脏页，因为不必回写磁盘
- 用户态的所有页都可回收，这样睡眠较久的进程会逐渐失去所有页框
- 使用简化的 LRU 算法，将很长时间未访问的页视为 unused 优先回收

### LRU 链表

用户态进程地址空间或 page cache 中的页根据最近是否访问分类后，存放于两个 LRU 链表 active_list 和 inactive_list 中。链表中的元素为 `struct page`，注意该数据结构统一存储于内核的 mem_map 数组，而其中的一部分通过 lru 链表指针字段连接成两条 LRU 链表。kswapd 内核线程在内存紧张时周期性扫描 inactive_list 获取要回收的页框。

### 反向映射

由于释放页框后需要更新页表，需要从页框描述符快速定位引用它的所有页表项。然而在每个页框描述符中引入附加字段 (一个链表存放所有引用它的页表) 会增大系统开销，所以 Linux 转而在数量更少的线性区描述符中做文章。线性区描述符中存放指向该进程内存描述符的指针，而内存描述符又包含页全局目录 PML4 的指针。(...todo)

### swap 交换

前面我们提到过，堆栈这种没有指定特定的映像文件内存区通常被称为匿名内存。在内存不足时将匿名页保存到磁盘的技术被称为交换。

- 页槽 (page slot) 是磁盘上的一个 4096 字节的块，每个页槽可以存放一个换出的页。
- 交换区 (swap area) 由页槽组成，交换区的第一个页槽会存放交换区的 metadata。交换区可以使用磁盘上的一整个分区 (如 `/dev/sda3`)，也可以使用某个分区中的文件 (如 `/swapfile`)。
- 交换子区 (swap extent) ：每个交换区又由交换子区组成，每个子区对应一组在磁盘上连续相邻的页槽。当使用磁盘分区为交换区时，每个交换区仅有一个子区；而使用文件时则可能有多个，因为文件系统不一定会把该文件分配在磁盘的连续块中。

### 内存不足

PFRA 会尽量保留一定的空闲页框，但仍可能出现内存耗尽的情况，此时内核会调用 out_of_memory() 函数从现有进程中选择一个拥有大量页框，运行时间相对较短，优先级较低的进程删除。