这一章中很多对象不能复制只能移动，如：`thread`, `future`, `promise`, `packaged_task` 

还有一些不能复制也不能移动，如：`mutex`, `atomic`

# Thread

##### Create/Join/Detach Thread

```c++
#include <thread>
void thread_fun(){};
void thread_fun_para(int x){};
std::thread thread1(thread_fun);
std::thread thread2(thread_fun_para(100));
thread1.join();
thread2.join();
std::thread (thread_fun_para, 100).detach();
```

Code after `join` won't be executed unless thread terminates. If using `detach`, main thread won't wait for child thread to terminate (child thread will be killed if main thread terminates). 

##### jthread (C++20)

 jthread 在析构时自动 join

##### reference parameter

The arguments to the thread function are moved or copied by value. If a reference needs to be passed, it has to be wrapped with `std::ref` or `std::cref`. (otherwise won't compile)



# Mutex & Lock

### mutex

```c++
#include <mutex>
std::mutex m_;
m_.try_lock(); 	// tries to lock the mutex, returns false if the mutex is not available
m_.lock(); 		// lock the mutex, blocks if mutex not available
// protected shared area
m_.unlock();
```

### shared_mutex

```c++
std::shared_mutex m_;
m_.lock_shared();
m_.unlock_shared();
m.lock();
m.unlock();
```

若一个线程已获取独占性锁（通过 lock 、 try_lock ），则无其他线程能获取该锁（包括共享的）。 仅当任何线程均未获取独占性锁时，共享锁能被多个线程获取（通过 lock_shared 、 try_lock_shared ）。

可解读成读写锁： 可以有任何数量的线程同时读数据，但一个线程只能在无其他线程读写时写数据。

### lock_guard

`lock_guard`call `mutex.lock()` in constructor, call `mutex.unlock()` in destructor. 

Doesn't support copy and move.

```c++
{
 std::lock_guard<std::mutex> l(mutex_); // lock
 // protected shared area
} // unlock
```

### unique_lock (write lock)

`unique_lock`call `mutex.lock()` in constructor, call `mutex.unlock()` in destructor. 

More flexible than `lock_guard`, can be moved or copied, provide more interfaces, but use more memory. 

```c++
std::unique_lock<std::mutex> get_lock() { // can be passed
 extern std::mutex m;
 std::unique_lock<std::mutex> l(m);
 // do sth
 return l; // 不需要 std::move，编译器负责调用移动构造函数
}

int main() {
 std::unique_lock<std::mutex> l(get_lock());
 // do sth
 return 0;
}
```

### shared_lock (read lock)

`shared_lock`call `mutex.lock_shared()` in constructor, call `mutex.unlock_shared()` in destructor. 

### scoped_lock (C17)

`std::scoped_lock` 相比上述的其他 RAII 锁，提供了更高级的功能，允许同时锁定多个互斥量，并提供了一种避免死锁的机制。



# conditional variable



# atomic

使用时需要十分小心。对于需要同步的是单个的变量或者内存位置，通常可以使用`std::atomic`。而当需要对两个以上的变量或内存位置作为一个单元来操作的话，通常应该使用互斥量。看如下例子，用户希望在一个类中，缓存一个开销昂贵的`int`，并使用一个 `bool` 类型指示其有效性，两个采用 `atomic` 的实现都有问题：

```c++
class Widget {
public:
    int getValueVer_1() const // does not always work
    { // if thread_A in expensiveFunc(), flag not updated, thread_B may compute again
        if (cacheValid) return cachedValue;
        else {
            cachedValue = expensiveFunc();  // first, set value atomically
            cacheValid = true;              // second, update flag atomically
            return cachedValid;
        }
    }
  
  	int getValueVer_2() const // even worse
    { // if thread_A in expensiveFunc(), value not updated, thread_B get invalid value
        if (cacheValid) return cachedValue;
        else {
            cacheValid = true;              // first, update flag atomically 
            cachedValue = expensiveFunc();  // second, set value atomically
           
            return cachedValid;
        }
    }
    
private:
    mutable std::atomic<bool> cacheValid{ false };
    mutable std::atomic<int> cachedValue;
};
```

### operations

- `store`: set

- `load`: get

- `exchange`: replaces the underlying value with a given value

- `compare_exchange(expected, desired)`: atomically compares the value of the atomic object with non-atomic argument `expected` and:
  
  -  if equal: performs atomic exchange with `desired` , return `true`
  - if not equal: atomic load value to `expected`, return `false`
  
  `compare_exchanges_weak` 和 `compare_exchanges_strong` 的区别：值与 `expected` 相等时，是否一定能成功并返回 `true`:
  
  - `compare_exchanges_weak`: has better performance，允许偶然的出乎意料的返回，(比如在字段值和期待值一样的时候却返回了`false`), 在一些**循环**算法中，这是可以接受的，可以认为循环一般都用 weak，因为反正循环中会经常重试，偶尔多失败一次换来每次调用的性能提升是十分划算的
  - `compare_exchange_strong` 失败时一直重试，保证一定会成功
  
- `wait`: blocks the thread until notified and the atomic value changes

- `notify_one`/`notify_all`: notifies one/all thread waiting on the atomic object

### memory order

- memory_order_relaxed: no synchronization or ordering constraints 最宽松的模型，仅仅保证原子性
- memory_order_consume
- memory_order_acquire
- memory_order_release
- memory_order_acq_rel
- memory_order_seq_cst: Sequentially-consistent ordering, 最严格的同步模型 
  - 所有以 seq_cst 为参数的原子操作(不限于同一个原子变量)，对所有线程来说有一个全局顺序
  - 两个相邻 seq_cst 原子操作之间的其他非原子变量操作也不能 reorder 到这两个相邻操作之外

> ### 原子操作 & Arm 弱内存序
>
> 在 ARM 平台上，从 ARMv8 开始，很多指令被添加了内存屏障语义。如 `ldaxr`/`stlxr` 指令与 `ldxr`/`stxr` 加上内存屏障指令等价，实测下来前者性能也更好。
>
> ARMv8.1 之前，如果想实现原子操作，必须要使用 LL/SC 操作，其本质是很多 CPU 核去抢某个内存变量的独占访问。以前 ARM 主要在低功耗设备上运行，核数不会太多，没有太大的问题。但现在出现了多达几十核的 ARM 处理器，LL/SC 抢占会存在严重的性能问题。因此 ARMv8.1 中加入了大量原生原子操作指令，如 `LDADD`, `LDSET` 等。
>
> 总的来说，`std::atomic` 的使用和直接插入汇编下的屏障语句在大多数情况下是等价的，但 `std::atomic` 提供了更好的可移植性和可维护性。如果你需要特定的性能调优或特定硬件平台的精确控制，使用汇编屏障可能是必要的，但在大多数情况下，`std::atomic` 的内存序选项已经足够满足需求。
>
> ```cpp
> void full_barrier() { __asm__ __volatile__ ("" : : : "memory"); }
> ```
>
> 在使用 `std::atomic` 时，具体是否会产生硬件内存屏障汇编代码取决于你使用的内存序（memory order）选项和目标硬件架构。`std::atomic` 的不同内存序选项会影响编译器如何生成汇编代码，并在必要时插入硬件内存屏障指令来确保操作的顺序性。有关内存序的深入探讨可查阅 Computer Architecture 章节。



# Task-based programming

应优先考虑基于任务的并发编程，而非基于线程的模式，以避免接触线程管理的各种琐碎细节。

### async

```c++
template< class Function, class... Args >
std::future<>, std::decay_t<>
async(Function&& f, Args&&... args );
// ------------- Usage ---------------
auto a1 = std::async(&X::foo, &x, 42, "Hello");
auto a2 = std::async(std::launch::deferred, &X::bar, x, "world!");
```

`std::async()` The function template async runs the function f asynchronously (potentially in a separate thread which might be a part of a thread pool) and returns a std::future that will eventually hold the result of that function call. 

Async启动一个异步任务，最终返回一个std::future对象，可通过future来获取想要得到的结果。在之前我们都是通过thread去创建一个子线程，但是如果我们要得到这个子线程所返回的结果，那么可能就需要用全局变量或者引用的方法来得到结果，这样或多或少都会不太方便。

>  std::launch policy:
>
>  是个可选参数，可指定 policy 的重载把 policy 作为第一个参数
>
>  - `std::launch::async`: executed on a different thread, potentially by creating and launching it first (asynchronous evaluation)
>  - `std::launch::deferred`: executed on the calling thread the first time its result is requested (lazy evaluation)

### promise

```c++
promise<int> prom;
auto fut = prom.get_future();
prom.set_value(42)
fut.get();
```

promise 和 future 成对出现，可以看作是一个一次性管道：一个线程兑现承诺，往 promise 里 set_value；一个线程去 future 里 get。我们可以把 prom 移动传给新线程，这样老线程就完全不需要管理它的生命周期了。

> promise 和 future 还有个用法是使用 void 类型模板参数。这种情况下，两个线程之间不是传递参数，而是进行同步：当一个线程在一个 future 上等待时（get 或 wait），另一个线程可以通过调用 promise 上的 set_value 让其结束等待



# latch & barrier (C++20)

`std::latch` 是一种一次性同步原语，主要用于在多个线程完成某个任务后，阻止某个线程继续执行，直到达到一个预定的计数值。计数器达到零后无法重置。

`std::barrier` 允许多个线程在一个特定的同步点上等待，直到所有线程都到达这个点，然后所有线程都可以继续执行。 这个特性在并发编程中非常重要，因为它可以确保所有线程在继续执行之前都已经完成了必要的工作。计数器达到零后，所有线程继续执行，计数器会自动回到初始值，可再次使用。

> 注意区分 Compiler scheduling barrier 编译器调度屏障 及 Hardware memory barrier 硬件内存屏障，详见 Architecture 章节
>



# packaged_task

packaged_task 把一个函数打包，它可以像正常函数一样被执行，也可以传递给 thread 在新线程中执行。你可以从它得到一个未来量：

``` c++
packaged_task<int()> task{work};
auto fut = task.get_future();
thread th{move(task)};
fut.get();
```

