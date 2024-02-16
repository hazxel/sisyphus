### Single Instruction/Multiple Data (SIMD)

SIMD operations refers to a computing method that enables processing of multiple data with a single instruction. Also called **vector** instructions. In contrast, the conventional sequential approach using one instruction to process each individual data is called **scalar** operations.





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