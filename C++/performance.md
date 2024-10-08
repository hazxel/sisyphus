# profiling 

cpu-profiling工具：

- linux perf
- brpc-view(CPU-profiler)，可用于分析性能瓶颈、热点函数、热点开销、耗时分布等。
- RDTSCP (Read Time-Stamp Counter and Processor ID): Intel cpu 指令, read the CPU's cycle counter

mem-profiling工具：

- brpc-view（heap-profiler），可用于分析常驻内存消耗、内存申请热点、内存泄漏等问题。
- gcc address sanitizer (ASAN)

系统调用: strace

valgrind: 参见 Vargrind.md

core dump?? Abort (core dumped)??



# Avoid unnecessary virtual function

缺点：

- 虚函数表机制导致调用成员函数时多一次跳转，有性能损耗
- vtable和object在内存中分离存储，大概率导致cache miss，导致性能退化
- 编译时无法确定具体调用的函数，只能在运行时动态决定，无法享受编译优化

优化方法：

- 使用函数指针替代虚函数
- 使用模板编程替代虚函数
- 对不被继承的子类用final关键字修饰（final修饰后，编译器可进行优化，线下demo测试耗时可优化约1.5%）



# 循环展开

循环展开是指通过增加每次循环迭代中处理的数据数量，来减少循环次数。循环展开对提升性能的帮助：

- 减少分支预测失败的次数
- 增加循环体内指令并行的可能性（需要依赖指令间不存在数据相关）



# 内存对齐

cpu一般按照双字节、四字节、八字节等为单位来存取内存。如果未做内存对齐，cpu在取数据时可能需要做多次内存访问，且可能需要做额外的位移、合并操作。做内存对齐后只需要一次内存访问，访存效率得到提升。

使用 `#pragma pack (n)`，编译器将按照n个字节对齐



# Cache-aware

指在程序设计时应该充分利用cache系统的特征，尽可能利用低延迟cache部件，以提升程序性能。常见方法有：

- 保证数据局部性，尽可能让数据在空间和访问时间上连续，从而提升cache命中率
- 程序中需要频繁使用的数据应尽可能load到三级cache中，例如需要查表的程序中可以考虑对表进行拆分或压缩，尽可能将表常驻高速缓存中
- 使用prefetch指令，提前将数据加载到缓存。prefetch是异步操作，可以指定将一个cache line（一般是64字节）的数据加载到指定cache level。prefetch的时机对性能影响非常大，prefetch最佳时机跟程序行为紧密相关，建议通过perf工具来分析和调整。
- 避免伪共享

gcc `__builtin_prefetch()` 预取给定地址的数据到缓存，减少 cache-miss 延迟



# 分支预测

现代 CPU 在预取指令时，会根据执行上下文猜测代码控制流分支的执行概率，猜测失败时需要丢弃先前工作重新取指令，会浪费约 20-40 个 cycle。可以通过先验知识编提示编译器和 cpu，提高分支预测准确率，如 gcc 提供的 `__builtin_expect` 指令：

```c
#define likely(x)			__builtin_expect(!!(x), 1)
#define unlikely(x)		__builtin_expect(!!(x), 0)
```

C++20 新增关键字  `[[likely]]` / `[[unlikely]]`：

```c++
if (n > 0) [[likely]]
    return n;
else [[unlikely]]
    return 1;
```

