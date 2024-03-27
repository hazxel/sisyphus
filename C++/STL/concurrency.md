这一章中很多对象不能复制只能移动，如：`thread`, `future`, `promise`, `packaged_task` 

还有一些不能复制也不能移动，如：`mutex`, `atomic`

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



### atomic

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

