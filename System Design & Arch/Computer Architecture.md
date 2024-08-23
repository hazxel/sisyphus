# CPU

### Architecture

##### AMD X86

x86 是强内存序架构，即使不显式使用内存屏障，编译器和硬件都会强制确保内存操作的顺序

##### ARM AArch64

AArch64 架构是弱内序的，这意味着内存操作可以被重排序

##### SMP（Symmetric Multiprocessing）vs UP（Uniprocessor）

SMP（Symmetric Multiprocessing）和 UP（Uniprocessor）是计算机体系结构中的两个概念，分别描述了处理器的配置和多处理能力。

- SMP 中有多个处理器（或核心）共享同一个内存空间和操作系统。这些处理器通常是对称的，即它们具有相同的能力和访问相同的硬件资源（如内存和I/O设备），操作系统和应用程序可以动态地将任务分配给任意处理器，从而提高并行处理能力和系统性能。
- UP 中系统只有一个处理器（或核心），所有计算任务都由这一个处理器来处理。



###  MMU (Memory Management Unit) 

即内存管理单元，是 cpu 可选的硬件模块，通常每个核心都配备一个，主要功能是**将虚拟地址转换为物理地址**。传统计算机系统中，内存控制器位于北桥芯片内部，CPU与内存的数据交换需要经过 “CPU-北桥-内存-北桥-CPU”5步，延迟较大，后逐步集成至 CPU 中。MMU 主要包括：

- **Table Lookaside Buffer (TLB)**: a physical cache in CPU, storing the most frequent accessed  Page Table Entry (PTE)，当 CPU 访问一个虚拟地址时，首先通过 TLB 查找虚拟地址的物理地址映射。

PTE 是操作系统软件层面的概念，但必须按照处理器定义的格式去填充，MMU 在地址翻译的过程中根据 PTE 的标志位来检测访问是否合法。

但处理器不能自动同步它们自己的TLB高速缓存，因为决定线性地址和物理地址之间映射何时不再有效的是内核，而不是硬件。处理器侧，以 Intel 为例，提供了两种 invalidate TLB 的方法，向 cr3 寄存器写入值，或 `invlpg` 汇编指令。而 Linux 内核则提供了比较丰富的 TLB 方法 `flush_tlb_xxxx`。



### Barrier

##### Compiler Scheduling Barriers：

作用在编译器层面，确保编译器不对内存操作进行重排序，通过语言级别的原子操作和内存序选项实现。

compiler barrier: `asm volatile ("" ::: "memory");` 

- `asm ("")` 是一个空的汇编指令
- `volatile` 防止编译器优化掉这个空指令。
- `:::"memory"` 表示这是一个内存屏障，告诉编译器此处涉及到对内存的访问，防止对该点前后的内存操作进行重排序。
- This compiler barrier is a possible implementation of `atomic_thread_fence(memory_order_release)` on x86, but not on AArch64. 因为 x86 是强内存序架构，而 AArch64 架构是弱内序的。为了实现 `memory_order_release` 的语义，必须使用汇编指令限定内存序， 如`__asm__ volatile ("dmb ish" ::: "memory");` （详见下）

##### Hardware Memory Barriers：

作用在硬件层面，确保处理器和内存操作的顺序性，通过低级汇编指令或特定的硬件同步机制实现。

- x86 processor 
  - write barrier: `asm volatile("sfence" ::: "memory");`
  - read barrier: `asm volatile("lfence" ::: "memory");`
  - read-write barrier: `asm volatile("mfence" ::: "memory");` 
  
- aarch processor 内存屏障指令:

  - `dmb`: Data Memory Barrier，其后的存储器访问动作必须在之前的存储器访问都执行完毕后执行

    等价 x86 的屏障指令：`dmb ishld` - 读屏障 ｜ `dmb ishs t` - 写屏障 ｜ `dmb ish` - 读写屏障

  - `dsb`: Data Synchronous Barrier，更严格，其后的任何指令都在之前的存储器访问完成后执行，有些版本的 Kernel 转而用 `dsb` 实现上面的读写屏障

  - `isb`: Instruction Synchronous Barrier，刷新 CPU pipeline 和 prefetch buffer，这意味着 ISB 后的指令需要重新从 cache 或 memory 取值，以保证所有之前的指令都执行完毕

  而其后又可跟如下参数：

  - Non-shareable：core 独享区域

    `NSHLD`: Load -Load, Load - Store | `NSHST`: Store - Store | `NSH`: Any - Any

  - Inner shareable：可被多核共享，但不需要都可访问。一个系统中可能有多个Inner shareable区域

    `ISHLD`: Load-Load & Load-Store | `ISHST`: Store-Store | `ISH`: Any-Any

  - Outer shareable：Outer Shareable 操作会隐性影响其中所有 Inner Shareable 区域，反之不成立

    `OSHLD`: Load -Load, Load - Store | `OSHST`: Store - Store | `OSH`: Any - Any

  - Full System：影响系统中的所有observer

    `LD`: Load -Load, Load - Store | `ST`: Store - Store | `SY`: Any - Any

##### Linux interface

建议尽量用 linux 提供的 API，适配不同架构做了封装，（除第一个外都为 CPU 屏障）：

- `barrier()` 编译器优化屏障
- `mb()` 读写内存屏障，用于SMP和UP
- `rmb()`读内存屏障，用于SMP和UP
- `wmb()`写内存屏障，用于SMP和UP
- `smp_mb()`用于SMP场合的内存屏障，对于 UP 不存在 memory order 的问题（对汇编指令），因此，在 UP 系统上仅被定义为一个优化屏障，确保汇编和 c 代码的 memory order 是一致的
- `smp_rmb()`用于SMP场合的读内存屏障
- `smp_wmb()`用于SMP场合的写内存屏障



# Cache

每个 CPU 核心都有缓存单元，主要包括：

- 硬件缓存内存 (hardware cache memory)：一般为 SRAM，容量小速度快，存放内存中真正的行
- 缓存控制器 (cache controller)：存放一个表项数组，管理每个缓存行到物理地址的映射。不涉及虚拟地址

### Hierarchy

- L1 Cache: usually embedded in the processor chip as CPU cache.
- L2 Cache: may be embedded on the CPU, or on a separate chip or coprocessor and have a high-speed alternative system bus connecting the cache and CPU. (not slowed by traffic on the main system bus)
- L3 Cache: shared by all cpu

### Cache locality

- Temporal locality: Recently referenced items are likely to be referenced again in the near future.
- Spatial locality: Items with nearby addresses tend to be referenced close together in time.

### ?? Write Strategy

- Write-through:  on a write-hit,  write immediately to memory

  > Requires write propagation

- Write-back: on a write-hit, defer write to memory until replacement of line

  > Requires write serialization

- Write-allocate: on a write-miss, load into cache, update line in cache

- No-write-allocate: on a write-miss, writes immediately to memory

### Cache coherence

##### Snooping vs Directory based

- Snooping:

  Send all requests for data to all processors. Processors snoop to see if they have a copy and respond. Requires broadcast, since caching information is at processors. Works well with bus (natural broadcast medium).

  > Dominates for small scale machines (most of the market), but scale poorly due to the use of broadcasting.

  - Invalidation-based: Write to a shared line has to invalidate copies in other caches. 
  - Update-based, makes local write update copies in other caches.

- Directory based:

  Keep track of what is being shared in one centralized place. Send point-to-point requests to processors. Scales better than snooping.

  - MESI Protocol
    - Four States for cache line：Modified, Exclusive, Shared, Invalid
    - 在 “独占” 和 “共享” 状态下，Cache 块的数据是 “清” 的，任何读取操作可以直接使用 Cache 数据；
    - 在 “已失效” 和 “已修改” 状态下，Cache 块的数据是 “脏” 的，它们和内存的数据都可能不一致。**延迟回写**机制意味着只在需要的时候才将数据写回内存，即当一个 CPU 核心要求访问已失效状态的 Cache 块时，会先要求其它核心先将数据写回内存，再从内存读取。在读取或写入 “已失效” 数据时，需要先将其它核心 “已修改” 的数据写回内存，再从内存读取；
    - 在 “共享” 和 “已失效” 状态，核心没有获得 Cache 块的独占权（锁）。在修改数据时不能直接修改，而是要先向所有核心广播 **RFO（Request For Ownership）请求** ，将其它核心的 Cache 置为 “已失效”，等到获得回应 ACK 后才算获得 Cache 块的独占权。这个独占权这有点类似于开发语言层面的锁概念，在修改资源之前，需要先获取资源的锁；
    - 在 “已修改” 和 “独占” 状态下，核心已经获得了 Cache 块的独占权（锁）。在修改数据时不需要向总线发送广播，能够减轻总线的通信压力。

### Cache line

缓存系统中是以缓存行为单位存储的，缓存行通常是 64 字节，并且它有效地引用主内存中的一块地址。Java 的 long 类型是 8 字节，因此在一个缓存行中可以存 8 个 long 类型的变量。所以，如果你访问一个 long 数组，当数组中的一个值被加载到缓存中，它会额外加载另外 7 个，以致你能非常快地遍历这个数组。事实上，你可以非常快速的遍历在连续的内存块中分配的任意数据结构。而如果你在数据结构中的项在内存中不是彼此相邻的（如链表），你将得不到免费缓存加载所带来的优势，并且在这些数据结构中的每一个项都可能会出现缓存未命中。

### Cache friendly programming

### False Sharing

有多个线程操作不同的变量，但都在相同的缓存行。线程之间轮番夺取拥有权不但带来大量的 RFO 消息，而且如果某个线程需要读此行数据时，L1 和 L2 缓存上都是失效数据，只有 L3 缓存上是同步好的数据，读L3非常影响性能。

##### Padding solution

填充无用的数据，让不同的对象处于不同的缓存行

##### alignment solution

### Cache vs Buffer

cache is between CPU and memory, buffer is between memory and disk. cache accelerate the CPU's read/write, buffer reduces the frequency of I/O read/write.



# 存储介质

### SRAM (Static Random Access Memory)

属于易失性存储器（断电后数据丢失）。存储密度不如DRAM高（SRAM存储一位数据需要4-8个晶体管，且工艺上堆叠困难，存储密度很难做大），不用周期性刷新，但访问速度比DRAM快，可以达到纳秒级，小容量时能够和处理器核工作在相同的时钟频率。一般用来作为 cache。

### DRAM (Dynamic Random Access Memory)

属于易失性存储器（断电后数据丢失）。特点是存储密度较高（存储一位数据只需一个晶体管，且可以堆叠，可以做到 SRAM 的 100 倍容量），需要周期性刷新，访问速度较快。其访问速度一般在几十纳秒级。

> **DRAM 接口**:
>
> - DDRx：目前常用 DDR5
> - GDDR：图形 DDR，用于显存
> - LPDDR：Low Power DDR，低电压低功耗的移动节能版本

### 磁性存储介质

如硬盘、磁带等。特点是存储密度高、成本低、具有非易失性（断电后数据可长期保存），缺点是访问速度慢。磁带的访问速度在秒级，磁盘的访问速度一般在毫秒级。

### ROM (Read-Only Memory)

- ROM: 存储的内容任何情况下都不会改变或丢失
- PROM (Programmable ROM): 可一次性写入数据，后续无法再更改（内部排列许多可用电流熔断的熔丝）
- EPROM (Erasable Programmable ROM): 可利用高电压写入数据，但抹除时需人工将线路曝光于紫外线下
- EEPROM (Electrically Erasable Programmable ROM): 带电可擦可编程只读存储器，掉电后数据不丢失。运作原理类似EPROM，但使用高电场抹除数据。

### FLASH (Flash EEPROM Memory)

闪存的耗电量小，但速度比 DRAM 慢，且有写入擦除寿命问题（几千次 P/E循环）。闪存正在逐步取代磁盘作为计算机尤其是终端的辅助存储器。细分类型有或非门闪存 NOR Flash，与非门闪存 NAND Flash （用来生产固态硬盘）等。

> ### Latency Numbers everyone should know
>
> |            Access             |   Time    | Cycles  |
> | :---------------------------: | :-------: | :-----: |
> |    execute cpu instruction    |   1 ns    |   1+    |
> |         read L1 cache         |  0.5 ns   |         |
> | read L2 cache (L1 cache miss) |   7 ns    |  10~20  |
> |       atomic operations       |   10 ns   |  10~20  |
> |       mutex lock/unlock       | 25-100 ns |  100+   |
> |  read memory (L2 cache miss)  |  100 ns   | 50~100+ |
> |             SRAM              |   10 ns   |         |
> |             DRAM              |   60 ns   |         |
> |             Flash             |           |         |
>



# Interconnect

### 南桥北桥架构

- 北桥：离CPU最近的芯片，主要负责 CPU、内存、南桥、PCIe设备间的数据交换。
- 南桥：主要连接低速网卡、USB设备、音频、硬盘等设备，通过 PCI 总线和 LPC 总线等连接外设。
- 系统总线：连接 CPU 和北桥，也称处理器总线。现代 CPU 中集成了北桥后，其成为了 CPU 片内总线。
- 内存总线：连接北桥和主存储器。现代 CPU 中集成了北桥后，也可认为其直连 CPU 和主存。
- PCIe：PCIe 设备通过 PCIe 总线连接至北桥芯片，目前主要用于连接 GPU

现代计算机架构中，CPU 通常集成了 MMU 和 PCIe 控制器，因此不再需要北桥芯片。同时，南桥的功能也逐渐被整合到主板上的其他芯片中，或由处理器直接管理。

### PCI

### PCIe

PCIe 总线目前主要用于连接 GPU 与 CPU

### NVLink

为了在单个计算节点上使用多个 GPU，先前一般通过 PCIe Switch 将多块 GPU 与 CPU 相连。PCIe 3.0*16有接近 32GB/s 的双向带宽，但当训练数据不停增长的时候，PCIe 的带宽还是逐渐成为瓶颈。

NVIDIA 开发的互联架构 NVLink 便是为了解决该问题。单条 NVLink 具备双路双工共40GB/s的带宽。如 P100 上集成了4条NVLink 后可具备 160GB/s 的带宽。除了实现 GPU 间的互联，部分 CPU 也开始支持 NVLink 标准（如 IBM 的 Power 处理器）

### CXL




# NUMA (Non-Uniform Memory Access)

在传统的统一内存访问架构（UMA，Uniform Memory Access）下，所有处理器共享同一块内存，导致内存带宽成为瓶颈。在现代多处理器系统中，大多采用非统一内存访问架构 NUMA，在NUMA架构下，多个CPU被封装在一起，这种封装被称为CPU Socket（插槽），每个CPU Socket拥有自己的本地内存，访问速度比访问其他Socket的远端内存更快。



# RDMA



# Data Representation

### ILP model

在32位和64位平台上，`float`都是 4 字节，`double`都是 8 字节。而标准C中并没有明确规定`long`类型的长度，只规定了`long long`至少占用32位，所以在操作系统的ABI中，要明确`long`类型的宽度。

LP64，ILP64，LLP64 是 64 位平台上的字长模型，ILP32 和 LP32 是 32 位平台上的字长模型。其中I指`int`，L指`long`，LL指`long long`，P指`pointer`。（e.g. LP64 代表 long 和指针为64位，int没出现所以是32位，long long一般也最多64位 ）现今所有64位的类Unix平台均使用 LP64 ，而64位Windows使用 LLP64 。



# Single Instruction/Multiple Data (SIMD)

SIMD operations refers to a computing method that enables processing of multiple data with a single instruction. Also called **vector** instructions. In contrast, the conventional sequential approach using one instruction to process each individual data is called **scalar** operations.
