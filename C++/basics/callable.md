# function

### argument passing 

Pass by value - when you need a copy or accepting basic types

```c++
void Func(Data);
Func(Data());								// best, only one ctor called in-place!!!
Data d; Func(d);						// ctor + copy_ctor
Data d; Func(std::move(d));	// ctor + move_ctor
```

Pass by reference - when modify the param or pass output

```c++
void Func(Data &);
Func(Data());								// won't compile, lvalue ref cannot bind rvalue
Data d; Func(d);						// compiles
Data d; Func(std::move(d));	// won't compile, lvalue ref cannot bind rvalue
```

Pass by const reference - default choice for read-only params

```c++
void Func(const Data &);
Func(Data());								// compiles
Data d; Func(d);						// compiles
Data d; Func(std::move(d));	// compiles
```

Pass by rvalue reference - take ownership of the passed param

```c++
void Func(Data &&);
Func(Data());								// compiles
Data d; Func(d);						// won't compile, prevent lvalue accidently passed to it
Data d; Func(std::move(d));	// compiles
```

Smart Pointers - be careful - see memory chapter

Not recommended:

- `void Func(const Data)`: The variable wiil be destroied when out of scope, why can't I modify it?
- `void Func(const Data &&)`: So I took the ownership but still couldn't modify it?
- raw pointers

### return type

- for free functions, usually by value return is the only option, except for returning static/global objects
- for member methods, value, const and non-const reference are all possible
- rvalue reference？？？



# lambda

A convenient way of defining an anonymous function, full syntax:

`[capture list] (params list) mutable exception-> return type { function body }`

but usually:

- `[capture list] {function body}`

- `[capture list] (param list) {function body}`

- `[capture list] (param list) -> return type {function body}`

 ### Capture list

 Lambda expression can use the "outer" variables in it's scope, but must be included in the capture list `[]`

 pure lambda (non-capturing) expressions are free of side effects, and therefore cannot cause, e.g., race conditions 

捕获列表会形成一个闭包，实现原理呢就是靠语法糖生成一个匿名的结构体，捕获的都会作为这个匿名结构体中的变量

 ```c++
void abssort(float* x, unsigned n) {
  std::sort(x, x + n,
    // Lambda expression begins
    [](float a, float b) {
      return (std::abs(a) < std::abs(b));
    } // end of lambda expression
  );
}
 ```

  

 ```c++
shared_ptr<vector<Data>> data;
auto fun1 = [&]() {
	//do somthing with data 1 million times
};
fun1();
// 使用捕获列表时，会多一次内存寻址：需要先把被捕获的对象的地址存在栈上，再把该栈上存地址的单元的地址作为入参传入，同样的，在函数内访问该捕获对象时也需要取地址两次
 
auto fun2 = [](shared_ptr<vector<Data>> &d) {
	//do somthing with data 1 million times
};
// 通过参数传入引用时，和普通函数无区别
fun2(data);
 ```

> 在使用 gcc 编译时，捕获列表为空，似乎也还是会安排一个字节的空间到栈上占位置:
>
> The current object (*this) can be implicitly captured if either capture default is present. If implicitly captured, it is always captured by reference, even if the capture default is `=`. The implicit capture of *this when the capture default is `=` is deprecated.(since C++20)

### generic lambda (C++14)

generic lambda has `auto` in its parameter list, it's equivalent to:

```c++
auto lambda = [](auto x, auto y) {return x + y;};

struct unnamed_lambda
{
  template<typename T, typename U>
    auto operator()(T x, U y) const {return x + y;}
};
auto lambda = unnamed_lambda();
```

？？？泛型闭包：

```c++
auto f3 = [](auto a) {
  return [=]() mutable { return a = a + a; };
};
auto twice1 = f3(1);
cout << twice1() << endl; // 2
cout << twice1() << endl; // 4
auto twice2 = f3(string{"a"});
cout << twice2() << endl; // aa
cout << twice2() << endl; // aaaa
```





# STL Functionals

### function

函数包装器，可包装各种类型的调用实体如：普通函数，对象方法，实现了仿函数操作符的对象，lamda表达式等：`std::function<int(int)> callback;`

STL中大量使用function作为算法的入参，如`sort`, `for_each`, `visit` 等

> implementation in-deep: https://zhuanlan.zhihu.com/p/142175297

### bind

给函数绑定参数，使之变为另一个签名的函数

### std::ref

本质是一个wrapper，可以在使用bind的时候使之变为传引用（默认为传值）





# Exception

C++98 中，函数必须声明可能抛出的异常类型。如果函数实现有所改变，异常说明也可能需要修改，这可能会影响客户端代码，因为调用者可能依赖原版本的异常说明。C++11标准化过程中，委员会认为真正有用的信息是一个函数是否会抛出异常。

现代 C++ 使用`noexcept`保证函数不会抛出任何异常。`noexcept` becomes a part of function type since C++17. C++98's `throw` specifier become *deprecated* in C++17, and is removed in C++20.

### Try-Catch

```c++
try { /* */ } catch (const std::exception&) { /* */ }
try { /* */ } catch (...) { /* */ }
```

A function-try-block associates a sequence of catch clauses with the entire function body, and with the member initializer list (if used in a constructor) as well.

```c++
struct S {
    std::string m;
    S(const std::string& str, int idx)
    try : m(str, idx)
    { /* ... */ }
    catch(const std::exception& e)
    { /* ... */ }
};
```

### stack unwinding 运行时函数栈展开

栈展开指的是抛出异常时：

1. 先检查 `throw` 本身是否在 `try` 块内部（`try` 可能会嵌套）
2. 从最近的的 `catch` 块开始尝试匹配可处理该异常类型的 `catch` 块，如果不能处理，退出当前函数并释放当前函数栈上的局部对象，继续到上层的调用函数中查找，直到找到一个可以处理该异常的 `catch` 块
3. 若不在 `try` 内部，或检索到最外层 `try` 仍未匹配，调用 `std::terminate`，不析构任何对象终止程序
4. 若匹配，按照构造顺序的反序析构所有堆栈上的对象，执行该 `catch` 块后，继续执行程序。

> 析构函数不应抛出异常：在为某个异常进行栈展开的时候，析构函数如果又抛出自己的未经处理的另一个异常，将会导致调用 `std::terminate` ，再调用 `std::abort` ，导致程序非正常退出。

> 栈展开期间不会释放用 `new` 动态分配的内存对象，有内存泄露风险。

### noexcept (operator & specifier) (C++11)

- **specifier**: 声明为`noexcept` 的函数不处理异常信息，抛出异常时不生成异常类型信息，会调用 `std::terminate`，默认调用 `std::abort`, cause abnormal program termination (调用C库的 `abort()`，直接终止程序不返回任何值), unless SIGABRT is caught. 
  
  - declare a function as `noexcept` means it's declared not to throw exception
  - declare a function as `noexcept(expr)` means it's not allowed to throw exception if expr evaluates to true
  
  注意编译器不会为函数实现和异常规范提供一致性保障，`noexcept` 函数可以调用非 `noexcept`函数。
- **operator**: evaluate to true if an expression is declared to be `noexcept`

```c++
void funA() noexcept; // 函数funA不抛出异常
void (*fp)() noexcept(false); // fp指向的函数允许抛出异常
// 此处第一个noexcept是specifier，第二个是操作符
template <class T, class Alloc>
    void swap(list<T,Alloc>& x, list<T,Alloc>& y)
         noexcept(noexcept(x.swap(y)));
```

除了在接口给用户更多信息，添加 `noexcept` 还能使编译器做更多优化。优化器不需要保证 `noexcept` 函数的运行时栈可展开，也不保证函数中的对象按照构造的反序析构。Actually, **noexcept doesn't change the assembly**, but it provides semantic meaning that library code can rely on, which can eventually cause significant changes in the assembly, or no changes in the assembly.

除了显式声明为 `noexcept` 的函数外，C++11开始，编译器自动生成的一些函数，如 Dtor, Default Ctor, CCtor, MCtor，delete 都会尽量被隐式定义为 `noexcept`, 除非基类或成员的构造/析构/移动等是 potentially-throwing

### `noexcept` and *move*

将复制操作替换为移动操作有时会破坏函数的异常安全保证，这是个很严重的问题。

> 标准库容器 `std::vector` 在扩容时，会通过 `std::vector::reserve()` 重新分配空间并转移已有元素。在C++98中这是通过复制来实现的，而如果想要优化为移动，会有这种问题：如果 *n* 个元素已移动到了新内存，但异常在移动第 *n+1* 个元素时抛出，那么对 vector 的移动就不能完成。因为原始的`vector`已经被修改，若想回滚到失败前的状态，从新内存移动到老内存本身又可能引发异常。

因此，很多函数，尤其是**STL中许多常用函数**，只有在移动元素的操作为 `noexcept` 时才能优化为移动版本。

>  这些函数通常会调用 `std::move_if_noexcept`（ `std::move` 的变体，检查类型的移动构造函数是否为 `noexcept`，视情况转换为右值或保持左值）。而 `std::move_if_noexcept` 会查阅 `std::is_nothrow_move_constructible`这个 *type trait*，编译器基于移动构造函数是否有 `noexcept`或`throw()` 来设置这个 *type trait* 的值。

### `noexcept` and *swap*

`swap`函数是STL算法实现的一个关键组件，它也常用于拷贝运算符重载中。它的广泛使用意味着对其施加不抛异常的优化是非常有价值的。交换高层次数据结构是否`noexcept`取决于它的构成部分的那些低层次数据结构是否`noexcept`，这激励你只要可以就提供`noexcept`的 `swap`函数。

```c++
template <class T1, class T2>
struct pair {
    void swap(pair& p) noexcept(noexcept(swap(first, p.first)) &&
                                noexcept(swap(second, p.second)));
};
```

### C++ Exception best practice

- 大多数函数不应该声明为`noexcept`，而应当是异常中立（*exception-neutral*）的。这些函数自己不抛异常，但是它们内部的调用可能抛出。这些函数内不处理异常，也不立即终止程序，而是让异常顺畅的到达调用者，把异常处理留给调用这个函数的函数。
- 何时使用 `noexcept`: 
  - 总是将确信不抛异常的函数声明为`noexcept`
  - 大量、频繁被使用，且有**自然的**`noexcept`实现法的函数，比如移动操作和`swap`。
  - `operator delete`, `operator delete[]`和 Dtor：它们在 C++98 时代可以抛出异常，但在这些函数中抛出异常是公认的需要避免的行为。C++11 中加强了约束：他们会被隐式声明为 `noexcept`，除非基类或非静态成员的析构显示声明为 `noexcept(false)`。
  - Ctor 如果抛出异常可能导致对象部分构造，要保证能够适当的撤销已构造成员。
- 为了`noexcept`而扭曲函数实现来达成目的是本末倒置。如果一个简单的函数实现可能引发异常，而你为了讨好调用者转而捕获所有异常，然后替换为状态码或者特殊返回值，这将会使函数实现和调用都变得复杂。调用者可能不得不检查状态码或返回值，开辟额外的分支，增大的函数也会给指令缓存带来压力。这些损耗可能超出`noexcept`带来的性能提升，且损害可读性和可维护性。
- 一些库接口设计者会区分有宽泛契约（wild contracts）和严格契约（narrow contracts）的函数。有宽泛契约的函数没有前置条件，不管程序状态如何永远不会表现出未定义行为。反之严格契约函数的前置条件若不满足将会出现未定义行为。设定 invariants 后将函数标记为 `noexcept` 是可能的，在我看来就是将实际参数的合法性检查责任甩交给了调用者，从而减少了运行时检查/异常处理的开销。





# const



# constexpr

和 `noexcept` 类似，`constexpr`是对象和函数接口的一部分。`constexpr`相当于宣称“我能被用在要求常量表达式的地方”。`constexpr` can be applied to functions，even **CTORs** and **DTORs** (C++20), indicating that the return value is computed at compile time wherever possible, which makes your program run faster and use less memory.

如果传给`constexpr`函数的实参在编译期可知，那么结果将在编译期计算。当 `constexpr`函数被一个或者多个非编译期值调用时，它就像普通函数一样，在运行时计算结果。这意味着你不需要两个函数，一个用于编译期计算，一个用于运行时计算。`constexpr`全做了。



# Terminologies

- 自由函数*free function*，指的是非成员函数，即一个函数，只要不是成员函数就可被称作*free function*
