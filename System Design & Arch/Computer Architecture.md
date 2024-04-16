# CPU

### Architecture

AMD



###  MMU (Memory Management Unit) 

即内存管理单元，是 cpu 可选的硬件模块，主要功能是将虚拟地址转换为物理地址。

硬件MMU就可以在地址翻译的过程中根据PTE的标志位来检测访问是否合法，这也是为什么PTE是一个软件实现的东西，但又必须按照处理器定义的格式去填充，这可以理解为软硬件之间的一种约定。那可以用软件去检测PTE么？当然可以，但肯定没有用专门的硬件单元来处理更快嘛。



# Cache

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

缓存系统中是以缓存行为单位存储的，缓存行通常是 64 字节，并且它有效地引用主内存中的一块地址。一个 Java 的 long 类型是 8 字节，因此在一个缓存行中可以存 8 个 long 类型的变量。所以，如果你访问一个 long 数组，当数组中的一个值被加载到缓存中，它会额外加载另外 7 个，以致你能非常快地遍历这个数组。事实上，你可以非常快速的遍历在连续的内存块中分配的任意数据结构。而如果你在数据结构中的项在内存中不是彼此相邻的（如链表），你将得不到免费缓存加载所带来的优势，并且在这些数据结构中的每一个项都可能会出现缓存未命中。

### Cache friendly programming

### False Sharing

有多个线程操作不同的变量，但都在相同的缓存行。线程之间轮番夺取拥有权不但带来大量的 RFO 消息，而且如果某个线程需要读此行数据时，L1 和 L2 缓存上都是失效数据，只有 L3 缓存上是同步好的数据，读L3非常影响性能。

##### Padding solution

填充无用的数据，让不同的对象处于不同的缓存行

##### alignment solution

### Cache vs Buffer

cache is between CPU and memory, buffer is between memory and disk. cache accelerate the CPU's read/write, buffer reduces the frequency of I/O read/write.









# Numa

在传统的统一内存访问架构（UMA，Uniform Memory Access）下，所有处理器共享同一块内存，导致内存带宽成为瓶颈。在现代多处理器系统中，大多采用非统一内存访问架构（NUMA， Non-Uniform Memory Access），在NUMA架构下，多个CPU被封装在一起，这种封装被称为CPU Socket（插槽），每个CPU Socket拥有自己的本地内存，访问速度比访问其他Socket的远端内存更快。



# Data Representation

### ILP model

在32位和64位平台上，`float`都是 4 字节，`double`都是 8 字节。而标准C中并没有明确规定`long`类型的长度，只规定了`long long`至少占用32位，所以在操作系统的ABI中，要明确`long`类型的宽度。

LP64，ILP64，LLP64 是 64 位平台上的字长模型，ILP32 和 LP32 是 32 位平台上的字长模型。其中I指`int`，L指`long`，LL指`long long`，P指`pointer`。（e.g. LP64 代表 long 和指针为64位，int没出现所以是32位，long long一般也最多64位 ）现今所有64位的类Unix平台均使用 LP64 ，而64位Windows使用 LLP64 。



# Single Instruction/Multiple Data (SIMD)

SIMD operations refers to a computing method that enables processing of multiple data with a single instruction. Also called **vector** instructions. In contrast, the conventional sequential approach using one instruction to process each individual data is called **scalar** operations.
