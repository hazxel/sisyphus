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



# Curiously Recurring Template Pattern (CRTP)

The main usage of the CRTP includes:

- add a generic functionality to a particular class.

 ```c++
template <typename T>
struct GeneralFunctions {
  void multiply(double multiplicator) {
    T& underlying = static_Cast<T&>(*this);
    underlying.setValue(underlying.getValue() * multiplicator);
  }
  void abs(); // and more ...
}
 
struct Data : public GeneralFunctions<Data> {
  double getValue() const;
  void setValue(double value);
};
 ```

 why not non-member template functions? **CRTP shows the interface**.

- create static interface

 ```c++
template <typename T>
class Base {
public:
	void interface() {
		static_cast<T*>(this)->impl();
	}
private:
  Base(){}; // no one can access private constructor, except the friend classes - T
  friend T; // won't compile unless derived class is T (avoid typo)
};
 
class Derived : public Base<Derived> {
  void impl();
};
 
template <typename T>
void exampleUsage(const Base<T> &foo) { foo.interface(); }
 ```

 why not virtual method? Avoid the virtual calls here because the information of which class to use was **available at compile-time**. 



# Mixin

 Mixin is complementary to the CRTP. A Mixin template defines a generic behavior, and inherit from the type you wish to plug functionality onto. 

```c++
class Name {
  void print() const;
};

template<typename Printable>
struct RepeatPrint : Printable {
  explicit RepeatPrint(Printable const& printable) : Printable(printable) {}
  void repeat(unsigned int n) const {
    while (n-- > 0)
      this->print(); // won't compile w/o "this->", names in template base classes ignored in C++
  }
};

// for auto type deduction
template<typename Printable>
RepeatPrint<Printable> repeatPrint(const Printable &printable) {
  return RepeatPrint<Printable>(printable);
}

Name name("Eddard", "Stark");
repeatPrint(name).repeat(10);
```

