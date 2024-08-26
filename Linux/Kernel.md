# Kernel syncronization

### Per CPU variable

由操作系统内核在软件层面上实现，以优化多核处理器的性能表现。操作系统为每个 CPU 分配一块独立的内存，用来存储该 CPU 的 per-CPU variable。

1. 操作系统通过 `wrmsr`, `swapgs` 等汇编原语，直接修改段寄存器，使每个 CPU 使用相同的虚拟地址，但实际将 per-CPU 变量映射到不同的物理地址
2. 通过把每个元素存放在不同的 cache line，防止并发访问数组元素导致 cache line 失效。

此做法无须耗时同步比较高效，但限制条件较多：

1. 只适用于内核态，用户态程序无法执行直接控制 CPU 的汇编语句（需要最高权限级别 Ring 0），且进程切换后无法保证 CPU 独占物理内存
2. 对来自异步函数 (中断处理程序和可延迟函数) 的访问不提供保护，在这种情况下需要另外的同步原语。
3. 在单处理器和多处理器系统中，内核抢占都可能使每CPU变量产生竞争条件，所以内核控制路径应在禁用抢占的情况下访问每CPU变量。

### Atomic operations

内核提供了 atomic_t 类型 (一个原子访问计数器) 和一些专用的函数和宏，如 `atomic_read`, `atomic_inc `等。这些函数和宏基本上是对原子汇编指令（有 lock 字节前缀）的简单包装。这些操作是为内核设计的，且依赖于内核的基础设施，用户态需要使用标准库或其他工具提供的原子操作。

### Memory barriers

- `barrier()` 编译器优化屏障 (除此以外都为 CPU 屏障）
- `mb()` 读写内存屏障，用于SMP和UP
- `rmb()`读内存屏障，用于SMP和UP
- `wmb()`写内存屏障，用于SMP和UP
- `smp_mb()`用于SMP场合的内存屏障，对于 UP 不存在 memory order 的问题（对汇编指令），因此，在 UP 系统上仅被定义为一个优化屏障，确保汇编和 c 代码的 memory order 是一致的
- `smp_rmb()`用于SMP场合的读内存屏障
- `smp_wmb()`用于SMP场合的写内存屏障

### 互斥锁（Mutexes） `mutex`

前面说互斥锁加锁失败，线程会出让CPU，首先从用户态切换至内核态，内核会把线程的状态从「运行」状态设置为「睡眠」状态，然后把 CPU 切换给其他线程运行；当互斥锁可用时，之前「睡眠」状态的线程会变为「就绪」状态（要进入就绪队列了），之后内核会在合适的时间，把 CPU 切换给该线程运行。 然后返回用户态。

### 自旋锁（Spinlocks） `spinlock`

自旋锁是通过 CPU 提供的 CAS 函数（Compare And Swap），在「用户态」完成加锁和解锁操作，不会主动产生线程上下文切换。

### 读写(自旋)锁（Read-Write Locks） `rwlock`

### 顺序锁 (Sequential Lock) `seqlock`

Linux 2.6 引入 

### 信号量（Semaphores） `semaphore`

Linux 提供两种信号量，内核信号量，由内核控制路径使用，以及 System V IPC 信号量，由用户态进程使用

### 完成量（Completion Variables） `completion`

Linux 2.6 还引入了一种类似信号量的原语 : 完成量（Completion），主要为了解决同一信号量上并发执行的问题

### 条件变量（Condition Variables） `condition variable`

### 读-拷贝-更新 （Read-Copy-Update, RCU）

Linux 2.6 引入, 用于网络层和虚拟文件系统





# syscall

系统调用在 Linux 内核中实现，他们的定义可以在 Linux 内核源代码中找到。系统调用通常通过 `::syscall` 接口进行访问，而不是通过标准的 C 库或用户空间库头文件直接定义。可以在 `/usr/include/bits/syscall.h` 找到 syscall 的定义。

### mmap & brk - 详见 Memory 章节

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