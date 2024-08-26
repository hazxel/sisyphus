# X86

### 原子指令

- 进行零次或一次**对齐内存访问**的汇编指令是原子的
- 如果在读操作之后 ，写操作之前没有其他处理器占用内存总线，那么这些 “读-修改-写” 汇编指令 (如 inc 或 dec) 是原子的。（单处理器系统中，永远不会发生内存总线窃用）
- 操作码前缀是 lock 字节 (0xf0) 的 “读-修改-写”汇编语言指令即使在 SMP 中也是原子的。当控制单元检测到这个前缀时，就“锁定” 内存总线 ，直到这条指令执行完成为止。
- 操作码前缀是 rep 字节 (0xt2，0xt3) 的汇编语言指令不是原子的

### 内存屏障

- write barrier: `asm volatile("sfence" ::: "memory");`
- read barrier: `asm volatile("lfence" ::: "memory");`
- read-write barrier: `asm volatile("mfence" ::: "memory");` 



# AArch64

### 内存屏障

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
