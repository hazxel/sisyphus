# function

### *FunctionObject* type

A FunctionObject type is the type of an object that can be used on the left of the function call operator.

- Pointer to function: they are *FunctionObject* types

- *Function objects*: they are FunctionObject types, also called functor.

  > Definition: A *function object* is any object for which the function call operator is defined.

- Functions and references to functions: they are **not** function object types, but can be used where function object types are expected due to function-to-pointer implicit conversion. (see type chapter)

- Lambda: type of lambda expressions are unspecified, but are generally syntactic sugar for functors. 

  - A lambda with empty capture list is translated into a unnamed structure, which has a member conversion function to a **function pointer type**. (implicit conversion to function pointer type)
  - Normal lambda is also translated into a unnamed structure which implements `::operator()`.

### qualifier

`const`, `inline`, `noexcept` ???

### reference and pointer to function type

see pointer&array chapter

### terminologies

- 自由函数*free function*，指的是非成员函数，即一个函数，只要不是成员函数就可被称作*free function*



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
2. 若在 `try` 内部，查看 `catch` 块是否可处理该异常类型，如果不能处理，退出当前函数并释放当前函数栈上的局部对象，继续到上层的调用函数中查找，直到找到一个可以处理该异常的 `catch` 块
3. 若不在 `try` 内部，或检索到最外层 `try` 仍未匹配，调用 `std::terminate`，不析构任何对象终止程序
4. 若匹配，按照构造顺序的反序析构所有堆栈上的对象，执行该 `catch` 块后，继续执行程序。

> **析构函数不应抛出异常**：在为某个异常进行栈展开的时候，析构函数如果又抛出自己的未经处理的另一个异常，将会导致调用 `std::terminate` ，再调用 `std::abort` ，导致程序非正常退出。另一方面，构造函数抛出异常是合理的。

> 栈展开期间不会释放用 `new` 动态分配的内存对象，这是使用裸指针的内存泄露风险来源之一。

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



# constexpr

和 `noexcept` 类似，`constexpr`是对象和函数接口的一部分。`constexpr`相当于宣称“我能被用在要求常量表达式的地方”。`constexpr` can be applied to functions，even **CTORs** and **DTORs** (C++20), indicating that the return value is computed at compile time wherever possible, which makes your program run faster and use less memory.

如果传给`constexpr`函数的实参在编译期可知，那么结果将在编译期计算。当 `constexpr`函数被一个或者多个非编译期值调用时，它就像普通函数一样，在运行时计算结果。这意味着你不需要两个函数，一个用于编译期计算，一个用于运行时计算。`constexpr`全做了。



# Overload (重载)

C++ allows multiple definitions for the same function name in the same scope. The definition of the functions must have different types and/or the number of arguments in the argument list. You cannot overload function declarations that differ **only by return type**.

A function template and a non-template function may be overloaded, while 2 *non-equivalent* function templates may also be overloaded. 

### Overload Resolution

With overloaded functions, the compiler has to determine a best match from the candidates.

It prefer: (按重要性由高到低)

1. the one with perfect match of function arguments

2. the one with better implicit conversion of function arguments

3. (initializing non-class objects) the one with better standard conversion from the return type to the type being initialized

4. Non-template function against template function

5. (both are template function, and same target specialization) more specialized primary template according to the *partial ordering rules for template specializations* (refer to [template chapter](C++/template/template.md))

   > 仅模版原型参与重载决议，模版特化不参与（这也是为什么不推荐特化函数模版，详见 [template 相关章节](C++/template/template.md)）

6. ...





# lambda

Lambda expression constructs a closure: an unnamed function object capable of capturing variables in scope. It's a convenient way to define an anonymous function. (本质上是语法糖，对语言的功能无影响，仅方便使用）首先区分以下概念：

- lambda 表达式 (lambda expression): `[](){}`是一个 prvalue 表达式，不是一个类，也不是一个实例
- 闭包类 (closure class/closure type): 每个 lambda 都会使编译器生成一个闭包类，用以实例化闭包。
- 闭包 (enclosure): lambda 创建的，从闭包类实例化而来的的运行时对象

Syntax：`[capture-list] (param-list) specifier -> return-type { function-body }`

- *capture-list* : the captured outer variables
- *specifier* : (optional)  Lambdas are `const` by default (i.e. the cv-qualifier of `operator()` is const ) because function object should produce the same result every time it's called. To change the state of the function, use`mutable` specifier.
- *return-type* : (optional)  

### Implementation

The lambda expression is of a unique unnamed class type (*closure type*), declared in the smallest block, class, or namespace scope that contains the expression

> 有关这个生成的匿名 *closure type*，以在 `main` 函数中，返回值/参数为 `long(int)` 的 lambda 为例：
>
> - `decltype()` 看到的 type 是 `main()::<lambda(int)>`
>
> - `typeid().name()`  看到的名字为：`Z4mainEUliE_`, `Z4mainEUliE0_`, `Z4mainEUliE1_`, ... ( ) 

- 依据 capture-list 为 closure type 产生对应的成员变量和构造函数 (一对一存在闭包对象里)

- 依据 param-list 为 closure type 产生对应的 Function call operator `operator()`

- 如果 capture-list 为空，还会产生一个成员转换函数 `operator FUNC_TYPE()`，将 closure type 隐式转换为对应的函数指针类型 (MSVC2010 doesn't support this, but this is part of the standard). 


### Capture

The *captures* is a comma-separated list of zero or more *captures*, e.g. `[=, a, &b](){}`

- optionally, *capture-list* could begin with *capture-default*, can be `=` or `&` ，默认捕获模式有误导性，可读性差，可能导致一系列问题，应尽量避免使用 (Effective modern C++ item 31)
- 按引用捕获可能导致悬垂引用
- pure lambda (non-capturing) expressions are free of side effects (e.g., won't cause race conditions)
- 具有静态生命周期的对象，如全局变量，无法被捕获，但可以在 lambda 里使用，他们不受 *capture-default* 的影响（有误导性，比如你以为`[=]`默认按值捕获了，但事实上全局变量仍通过地址访问）

##### init capture 初始化捕获（也称 generalized lambda capture，通用捕获） (C++14)

在闭包内创建一个数据成员，并用作用域中的一个对象初始化它。出发点是为了解决缺少**移动捕获**的问题：

```c++
auto ptr = std::make_unique<Widget>();
auto func [ptr = std::move(ptr)] { /*...*/ };
```

##### capture vs pass-by-parameter: 捕获是有 overhead 的

```c++
shared_ptr<vector<Data>> data;
auto fun1 = [&]() {
	//do somthing with data 1 million times
}; // call by fun1();
// 使用捕获列表时，会多一次内存寻址：需要先把被捕获的对象的地址/值存在栈上（闭包类实例中），该栈上对象的地址相当于函数的入参。因此在函数内访问被捕获对象时，需要取地址两次
 
auto fun2 = [](shared_ptr<vector<Data>> &d) {
	//do somthing with data 1 million times
}; // call by fun2(data);
// 通过参数传入引用时，和普通函数无区别
```

##### capture `*this` (C++17)

Before C++17, current object `*this` is implicitly captured if either capture-default is present.  `[=]`-default will capture `this` by value (copy the value of the pointer); and `[&]`-default will captures `*this` by reference. That is to say, the effects of `[=]` and `[&]` for capturing `this` are the same - no copy!!

这非常有迷惑性，因为直觉上 `[=]` 会捕获值，但其默认捕获 `this` 指针时，相当于使用实际地址捕获了所有成员变量（所以使用时和引用捕获类似，指针可能失效！）为了避免多线程场景下 `this` 失效的情况，可以拷贝 `this`: `[=, *this]` or `[&, *this]`. 

> **(deprecated since C++20)** The implicit capture of `*this` when the capture default is `[=]` is deprecated. 终于！



### generic lambda (C++14)

generic lambda has `auto` in its parameter list:

```c++
auto lambda = [](auto x, auto y) {return x + y;};
// above is equivalent to below:
struct unnamed_lambda {
  template<typename T, typename U>
    auto operator()(T x, U y) const {return x + y;}
};
auto lambda = unnamed_lambda();
```

应用完美转发和可变形参：

```c++
auto f = [](auto&&... params) {
    return some_templ_func(std::forward<decltype(params)>(params)...);
};
```

> 欣赏：结合 `mutable` 的一个神奇的用法：
>
> ```c++
> auto f3 = [](auto a) {
>   return [=]() mutable { return a = a + a; };
> };
> auto twice1 = f3(1);
> cout << twice1() << endl; // 2
> cout << twice1() << endl; // 4
> auto twice2 = f3(string{"a"});
> cout << twice2() << endl; // aa
> cout << twice2() << endl; // aaaa
> ```

### vs `std::bind`





# STL Functionals

### std::function

函数包装器，可包装各种类型的调用实体如：普通函数，对象方法，实现了仿函数操作符的对象，lamda表达式等：`std::function<int(int)> callback;`

STL中大量使用function作为算法的入参，如`sort`, `for_each`, `visit` 等

> implementation in-deep: https://zhuanlan.zhihu.com/p/142175297

### std::bind

给函数绑定参数，使之变为另一个签名的函数

### std::ref

本质是一个wrapper，可以在使用bind的时候使之变为传引用（默认为传值），详见 reference 章节

### std::invoke (C++17)

调用不同类型的可调用对象（函数、函数指针、成员函数指针、仿函数等）的语法各不相同，例如使用 `()` 来调用函数或函数指针，使用 ` ->` 或 `.` 来调用成员函数等。这样的语法差异导致了代码的冗余和不一致，给编写和维护代码带来了困扰。`std::invoke` 提供了一种统一的调用语法，无论是调用函数、函数指针、成员函数指针还是仿函数，都可以使用相同的方式进行调用。



# Best Practice

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
