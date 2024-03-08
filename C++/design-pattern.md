# RAII (Resource acquisition is initialization)

RAII 机制是一种对资源申请、释放这种成对的操作的封装，通过这种方式实现在局部作用域内申请资源然后便可以自动销毁资源



# Type Erasure & Pattern Matching

把类型信息擦除。这种代码可以处理来自多个类或模板的对象，而不需要知道即将对其操作的对象的类型是什么。类型擦除主要用于泛型编程，可以编写出能接受任意类型参数的函数和类。

### 基于`void*`的实现

一种相当早期的类型擦除实践，将类型转换到`void*`便可擦除类型信息。现代C++编程不建议使用`void*`：

- 类型安全：编译时无法执行静态类型检查。运行时出错远比编译时出错难以诊断。
- 可读性：void*往往会使代码更难以理解，因为它隐藏了类型信息。
- 裸指针：直接操作裸指针十分危险

### 基于虚函数的实现

使用虚函数可以提供**运行时**多态。虚函数的实现和后面的实现方式相比来说更加简单易懂。

但虚函数实现运行效率相对较低，因为虚表的跳转会损耗部分性能，且使用虚函数会阻碍某些编译器优化。

### 基于`std::variant`的实现

`std::variant`提供一种类型安全并且在**编译期**的多态方案。`std::variant`可存储多种类型的对象，类型安全并且在编译期完成类型判断。但如果需要增加或者删除某些类型则需要修改大量代码，并且std::variant可能会造成内存浪费。节省了虚表跳转的开销，但variant自己似乎也需要存储并判断类型后寻找对应的函数代码？？？

```c++
#include <variant>
#include <iostream>
#include <cassert>

template <typename... Fs>
struct overload : Fs...
{
    overload(Fs&&... fs) : Fs{std::move(fs)}... { }
    using Fs::operator()...;
};

template <typename... Branches>
auto match(Branches&&... branches)
{
    return [o = overload{std::forward<Branches>(branches)...}](
        auto&&... variants) -> decltype(auto)
    {
        return std::visit(o, std::forward<decltype(variants)>(variants)...);
    };
}

int main()
{
    using VariantType = std::variant<int, float, char>;

    auto vis = match(
        [](int   x){ std::cout << x << "i\n"; },
        [](float x){ std::cout << x << "f\n"; },
        [](char  x){ std::cout << x << "c\n"; }
    );

    VariantType v0{10};
    VariantType v1{10.f};
    VariantType v2{'a'};

    vis(v0);
    vis(v1);
    vis(v2);

    auto value = match(
        [](int){ return 100; }, 
        [](float){ return 50; })(std::variant<int, float>(0));

    assert(value == 100);

    auto value2 = match(
        [](int){ return 100; }, 
        [](float){ return 50; })(std::variant<int, float>(0.f));

    assert(value2 == 50);
}
```

### 基于CRTP的实现

通过CRTP可以在基类中指定一个未被实现的接口，在派生类中再提供具体的实现，这就是编译期的多态。CRTP真正节省了虚表的开销。

### 基于Concept的实现(C++20)

Concept是一种类型约束，表示一个模板参数必须满足的条件。使用的类型满足Concept的约束才可以正常使用，否则会在编译时报错，但具体类型我们并不关心。和CRTP的实现一样都是通过模板来实现并且都在编译期完成。



# Singleton

https://zhuanlan.zhihu.com/p/37469260

C++11规定了local static在多线程条件下的初始化行为，要求编译器保证了内部静态变量的线程安全性。在C++11标准下，《Effective C++》提出了一种更优雅的单例模式实现，使用函数内的 local static 对象。这样，只有当第一次访问`getInstance()`方法时才创建实例。这种方法也被称为Meyers' Singleton。C++0x之后该实现是线程安全的，C++0x之前仍需加锁。（C++0x是C++11标准成为正式标准之前的草案临时名字）

```c++
class Singleton
{
public:
	static Singleton& getInstance() {
		static Singleton instance;
		return instance;
	}
};
```



# Type Punning

- `reinterpret_cast`: avoid it, unless to access the `char[]` of a type `T`, and after profiling the program you realize the compiler was not able to optimize the `std::bit_cast` (or polyfill) operation, you can think about `reinterpret_cast`.
- `bit_cast`
- `memcpy`



# 极致性能优化

### profiling 

cpu-profiling工具：valgrind、linux perf、brpc-view（CPU-profiler），可用于分析性能瓶颈、热点函数、热点开销、耗时分布等。
mem-profiling工具：brpc-view（heap-profiler），可用于分析常驻内存消耗、内存申请热点、内存泄漏等问题。

cache工具: cachegrind, 分析程序的cache miss情况，可以看到函数级的cache miss。

### Avoid unnecessary virtual function

缺点：

- 虚函数表机制导致调用成员函数时多一次跳转，有性能损耗
- vtable和object在内存中分离存储，大概率导致cache miss，导致性能退化
- 编译时无法确定具体调用的函数，只能在运行时动态决定，无法享受编译优化

优化方法：

- 使用函数指针替代虚函数
- 使用模板编程替代虚函数
- 对不被继承的子类用final关键字修饰（final修饰后，编译器可进行优化，线下demo测试耗时可优化约1.5%）

### 循环展开

循环展开是指通过增加每次循环迭代中处理的数据数量，来减少循环次数。循环展开对提升性能的帮助：

- 减少分支预测失败的次数
- 增加循环体内指令并行的可能性（需要依赖指令间不存在数据相关）

### 内存对齐
cpu一般按照双字节、四字节、八字节等为单位来存取内存。如果未做内存对齐，cpu在取数据时可能需要做多次内存访问，且可能需要做额外的位移、合并操作。做内存对齐后只需要一次内存访问，访存效率得到提升。

使用 `#pragma pack (n)`，编译器将按照n个字节对齐

### Cache-aware

指在程序设计时应该充分利用cache系统的特征，尽可能利用低延迟cache部件，以提升程序性能。常见方法有：

- 保证数据局部性，尽可能让数据在空间和访问时间上连续，从而提升cache命中率
- 程序中需要频繁使用的数据应尽可能load到三级cache中，例如需要查表的程序中可以考虑对表进行拆分或压缩，尽可能将表常驻高速缓存中
- 使用prefetch指令，提前将数据加载到缓存。prefetch是异步操作，可以指定将一个cache line（一般是64字节）的数据加载到指定cache level。prefetch的时机对性能影响非常大，prefetch最佳时机跟程序行为紧密相关，建议通过perf工具来分析和调整。
- 避免伪共享
