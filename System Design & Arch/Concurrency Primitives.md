# Parallelism

- 指令级并行（Instruction Level Parallelism, ILP）：主要指指令之间的并行性，当指令之间不存在相关时，这些指令可以在处理器流水线上重叠起来并行执行。现代处理器有多种较为成熟的技术挖掘指令级并行性，包括指令流水线、多发射、动态调度、寄存器重命名、转移猜测等。

  > 详见 Architecture/CPU 章节

- 数据级并行（Data Level Parallelism, DLP）是指对集合或者数组中的元素同时执行相同的操作。数据级并行性比较易于处理，典型应用有 SIMD，SPMD ，Map-Reduce 等。

- 数据并行（Data Parallelism）是一种并行计算模式，它强调同一个操作应用于大量数据的多个元素上。与数据级并行（Data Level Parallelism, DLP）概念相似，但更多指的是如何将计算任务分解为对大规模数据的并行处理，从而利用多个计算单元同时工作，以提高计算效率。

- 任务级并行（Task Level Parallelism）是将不同的任务（如进程或线程）分布到不同的处理单元上执行。



# Thread

### pthread vs std::thread

If you want to run code on many platforms, go for Posix Threads. They are available almost everywhere and are quite mature. On the other hand if you only use Linux/gcc `std::thread` is perfectly fine - it has a higher abstraction level, a really good interface and plays nicely with other C++11 classes.

The C++11 `std::thread` class unfortunately doesn't work reliably (yet) on every platform, even if C++11 seems available. For instance in native Android `std::thread` or Win64 it just does not work or has severe performance bottlenecks (as of 2012).



# Locks

### Semaphore

Semaphore is actually a signaling mechanism, not a lock. It is a counter that can be modified by P(wait /decrease), V(signal, increase) operations.

一个互斥量只能用于一个资源的互斥访问，信号量可以实现多个同类资源的多线程互斥和同步（或者生产者消费者模式）

### Mutex

Mutex is a synchronization primitive that grants exclusive access to the shared resource to only one thread. If a thread acquires a mutex, the second thread that wants to acquire that mutex is **suspended** until the first thread releases the mutex.

互斥锁加锁失败后，线程会释放 CPU ，给其他线程。此时会从用户态陷入到内核态来切换线程，虽然简化了使用锁的难度，但是存在一定的性能开销成本（两次线程上下文切换的成本：加锁失败时，「运行」状态设置为「睡眠」状态；锁被释放时，「睡眠」状态的变为「就绪」状态，内核在合适的时间把 CPU 切换给该线程；）

### Spinlock

a spinlock is a lock that causes a thread trying to acquire it to simply wait in a loop ("spin") while repeatedly checking whether the lock is available.

自旋锁不应该被持有时间过长。如果需要长时间锁定的话, 最好使用信号量。持有自旋锁的进程是不可抢占的（即使内核是可以抢占的），自旋锁的加锁函数spin_lock第一步便是禁止抢占；等待自旋锁的函数是可以被抢占的。

### Read-Write-Lock



### “乐观锁”与“悲观锁”

悲观锁：每次取数据的时候，都会认为该数据会被修改，所以必须加一把锁才安心。

乐观锁：认为同一个数据不会发生并发操作的行为，所以取的时候不会加锁，只有在更新的时候，会通过例如版本号之类的来判断是否数据被修改了。