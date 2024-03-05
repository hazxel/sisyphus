这一章中很多对象不能复制只能移动，如：thread、future、promise、packaged_task 

### Thread

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

### Mutex

##### mutex

```c++
#include <mutex>
std::mutex m_;
m_.try_lock(); 	// tries to lock the mutex, returns false if the mutex is not available
m_.lock(); 		// lock the mutex, blocks if mutex not available
// protected shared area
m_.unlock();
```

##### shared_mutex

```c++
std::shared_mutex m_;
m_.lock_shared();
m_.unlock_shared();
m.lock();
m.unlock();
```

### Lock

##### lock_guard

`lock_guard`call `mutex.lock()` in constructor, call `mutex.unlock()` in destructor. 

Doesn't support copy and move.

```c++
{
 std::lock_guard<std::mutex> l(mutex_); // lock
 // protected shared area
} // unlock
```

##### unique_lock (write lock)

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

##### shared_lock (read lock)

##### scoped_lock (C17)

### conditional variable





### async

```c++
template< class Function, class... Args >
std::future<>, std::decay_t<>
async(std::launch policy, Function&& f, Args&&... args );
```

`std::async()` The function template async runs the function f asynchronously (potentially in a separate thread which might be a part of a thread pool) and returns a std::future that will eventually hold the result of that function call. 

Async启动一个异步任务，最终返回一个std::future对象，可通过future来获取想要得到的结果。在之前我们都是通过thread去创建一个子线程，但是如果我们要得到这个子线程所返回的结果，那么可能就需要用全局变量或者引用的方法来得到结果，这样或多或少都会不太方便。

>  std::launch policy:
>
> - `std::launch::async`: enable asynchronous evaluation
> - `std::launch::deferred`: enable lazy evaluation

### promise

```c++
promise<int> prom;
auto fut = prom.get_future();
prom.set_value(42)
fut.get();
```

promise 和 future 成对出现，可以看作是一个一次性管道：一个线程兑现承诺，往 promise 里 set_value；一个线程去 future 里 get。我们可以把 prom 移动传给新线程，这样老线程就完全不需要管理它的生命周期了。

> promise 和 future 还有个用法是使用 void 类型模板参数。这种情况下，两个线程之间不是传递参数，而是进行同步：当一个线程在一个 future 上等待时（get 或 wait），另一个线程可以通过调用 promise 上的 set_value 让其结束等待

### packaged_task

packaged_task 则把一个函数打包，它可以像正常函数一样被执行，也可以传递给 thread 在新线程中执行。你可以从它得到一个未来量：

``` c++
packaged_task<int()> task{work};
auto fut = task.get_future();
thread th{move(task)};
fut.get();
```

