# Reference

A Reference is not a object, but an alias ot an existing object or function. Since references are not objects, there are **no arrays** of references, **no pointers** to references, and **no references** to references.

- Reference **cannot be modified to refer to other objects** after init.

  > 这意味着，将引用作为类成员会导致类不可赋值 (assignment operator will default to `delete`) 解决方案一般为使用指针，自定义所有特殊成员函数，或是使用 `reference_wrapper` （用法参见下文相关章节）

- Reference cannot be null, and must be initialized, but 悬垂引用 (dangling reference) is still possible

- There is no const reference (reference is by definition immutable, but reference to a const object is ok)

- reference has type check (safer than pointer)

个人理解，引用符号和cv-qualifier类似，提供类型信息以外的信息：alias所绑定的对象已经存在。右值引用进一步附加了信息：被绑定的对象是右值。

引用绑定就是给被绑定的对象一个别名：被绑定的对象本身有type和value category；别名的type需要体现被绑定对象的type+value category，但它自己本身的value category一定是lvalue。



# reference collapsing

由于 C++11 引入了右值引用，引用类型的引用被允许出现，但只允许出现在模版相关的编译器的内部推导（template, auto, decltype, ...）或者 typedef。以下代码 `int& && a;` 试图创建引用的引用，无法通过编译。

refernece to reference follows the reference collapsing rule:

```c++
typedef int& lref;
typedef int&& rref;
int n;
lref& r1 = n;		// type of r1 is int&
lref&& r2 = n;	// type of r2 is int&
rref& r3 = n;		// type of r3 is int&
rref&& r4 = 1;	// type of r4 is int&& !!!
```

Reference collapsing and template arguemnt deduction rules works together, so that universal reference and `std::forward` are possible.



# Universal Reference

C++11 标准引入右值引用，规定右值引用形式的参数只能接收右值，不能接收左值。但对于函数模板和`auto`中使用`T&&`语法定义的参数来说，它既可以接收右值，也可以接收左值（此时的右值引用又被称为“万能引用”）。这一特性可在参数增加时，避免指数级增长的重复函数定义。

万能引用其实是**模版自动类型推导**以及**引用折叠**一起作用而自然产生的结果：对于使用一个万能引用的函数模版 `template<typename T> void foo(T&&)`，如果需要匹配左值形参 `void foo(Widget&)`，则`T`可被推导为`Widget&`，这样`T&&`即 `Widget& &&` 就会被折叠为 `Widget&`；而对于右值形参 `void foo(Widget&&)`，`T`直接被推导为 `Widget`，`T&&` 则被推导为`Widget&&`。

 ```c++
template<class T>
void wrapper(T&& arg); // this is univeral reference
template<typename T>
void foo(std::vector<T>&& param); // this is not!!!
 ```

如果只希望定义右值形参而禁用左值，可使用 `std::enable_if` 实现：

```c++
template<typename T, 
	typename = std::enable_if<
		!std::is_same<T, 
			typename std::decay<T>::type
    >::value
	>::type
>
void (T &&foo) {};
```

> **避免在通用引用上重载** (Effective Modern C++, item 26/27) 
>
> - 通用引用函数非常容易覆盖别的重载（甚至非模版重载），这非常反直觉，即“重载时编译器会优先选择非模版函数”
>
> - 完美转发的转换构造函数是糟糕的实现，因为他们有时会比默认拷贝构造函数更加匹配，劫持拷贝和移动构造的调用。(e.g. 当传入的参数非 const 时，`template<T> Widget(T&&)` 会覆盖 `Widget(const T&)`)

> 其他通用引用失效的情况：
>
> - 花括号初始化器：编译器无法在通用引用中推导大括号表达式`{ 1, 2, 3 }`的类型为 `initializer_list`
> - 重载函数的名称和模板名称：在通用引用场景下，编译器没有类型信息来帮助选择合适的重载
> - 位域



# move - move semantic, 移动语义

`std::move` indicates that an object may be "moved from", allowing efficient transfer of resources. 。

**implementation**: remove reference and cast to `Widget&&` （使用万能引用, 不用显式指明模版参数`_Tp`）

```c++
template<typename _Tp>																		// rvalue: _Tp deduced to T
typename remove_reference<_Tp>::type&& move(_Tp&& __t) { 	// lvalue: _Tp deduced to T&
  typedef typename remove_reference<_Tp>::type _Up;
  return static_cast<_Up&&>(__t);													// all cast to T&&
}
```

- 对`const`对象的移动会转化为拷贝操作，因为`const T&&`不匹配移动操作的入参`T&&`，重载决议为拷贝
- 对不可移动类型的移动也能通过编译和运行，但会调用拷贝操作，因为根本没有接受 `T&&`的重载版本



# forward - perfect forwarding, 完美转发 (C++11)

引用本身可能为左值或者右值。通过 `forward` 可以强制保证左值引用转换为左值，右值引用转换为右值。

```c++
template<class T> // 万能引用将值完美转发给另一个万能引用
void foo(T&& arg) { arg.DoSomething(); }
template<class T>
void bar(T&& arg) { foo(std::forward<T>(arg)); }
```

forward 的本质：借助万能引用推导后显式传入的 `_Tp` 的类型来决定返回左值引用还是右值引用，和形参的 value category无关。

- 万能引用匹配左值形参时，显式传入的模版参数 `_Tp`是万能引用推导出的 `Widget&`。根据引用折叠规则，返回值 `_Tp&&` 实际类型为 `Widget&`，所以返回匿名左值引用（一定是左值）
- 万能引用匹配右值形参时，显式传入的模版参数 `_Tp `是万能引用推导出的 `Widget`。根据引用折叠规则，返回值 `_Tp&&` 实际类型为 `Widget&&`，返回匿名右值引用 (一定是右值)

**implementation**: （并非万能引用，必须显式指明模版参数，不可推导）

```c++
template <class _Tp>
_Tp&& forward(typename remove_reference<_Tp>::type& __t) {
  return static_cast<_Tp&&>(__t);
}

template <class _Tp>
_Tp&& forward(typename remove_reference<_Tp>::type&& __t) {
  static_assert(!is_lvalue_reference<_Tp>::value, "cannot forward rvalue as lvalue");
  return static_cast<_Tp&&>(__t);
}
```

- 重载一：直接将具名引用传入`std::forward` 时（大部份情况）一定是调用第一个左值形参的重载，因为具名左值引用和右值引用都是左值！（forward 函数内的`__t`有名字所以也是左值引用，若`_Tp`为右值引用类型则将其cast为右值引用且返回，相当于move）
- 重载二：如果将具名引用 move 后再传给 forward，就会实例化第二个右值形参的重载。（这个做法在我看来本身就比较奇怪，因为万能引用本身就是为了方便同时接收左值和右值，但这里又forward一个固定为右值的东西）注意 `_Tp` 不能为左值引用，编译器禁止把 rvalue 转发为 lvalue，只能显示把右值复制给左值。

>  `move` and `forward` both do type conversion, implemented using `static_cast`. *Efficient Modern C++* suggests that use `std::move` for rvalue reference conversion and `std:forward` for and **only for universal reference** scenarios.



# STL reference helper

### std::reference_wrapper

Wraps a reference in a copyable, assignable object. 

优点：

- 让引用具象为一个可复制、可赋值的对象
- 模拟引用，保证引用不可为 `null` 或无效，但也有悬空引用问题
- 无默认构造，也不能用临时对象 (rvalue) 初始化
- 有类似指针的重新绑定的灵活性

缺点：

- 要访问对象 `T`的成员变量或方法，必须使用 `std::reference_wrapper<T>::get` 
- 要为被引用的对象重新赋值（而不是绑定到一个新对象），也必须使用 `get()`：

Possible implementation：

- 构造时使用 `std::addressof` 获取指针 `_ptr` ，存放在实例内部。
- 通过成员函数 `constexpr T& get() const noexcept { return *ptr; }` 返回实例
- 实现转换函数 `constexpr operator T& () const noexcept { return *_ptr; }` 以便隐式转换为对应类型的指针 can be used as arguments with the functions that take `T&`.
- 实现 function call operator (functor) `auto operator()(Args&&... args) const { return get()(std::forward<Args>(args)...); }`，以便包装函数对象时可以直接使用函数调用运算符

### std::ref

`std::ref` creates a `std::reference_wrapper` with a type deduced from its argument. 因为可以自动推断，所以通常使用 `std::ref` 来创建 `std::reference_wrapper`。

### Use cases：

个人理解，虽然表现上和引用相似，本质上还是传指针，但更优雅和安全

- 在 `std::bind` 时使一个类型被推断为引用，按引用传递绑定参数，而不是传值:

  正常使用 `std::bind` 时，绑定的参数需要被复制或移动。使用 `reference_wrapper` 后，便可传递引用

- 可实现引用的容器: `vector<Widget&>` 是非法的，`vector<Widget*>` 又不够安全

- 使用 `std::thread` 时，按引用传递参数给函数: 

  `std::thread(func, arg);` 一般都是按值传递参数，即使函数接收引用形参 `void func(const T&);`；引入 `reference_wrapper` 后，便可传递引用

- 代替裸引用作为类成员时，不再导致类不可赋值 (can use default trivial assignment operator)

- 按引用传递 callable 对象 (function, lambda, functor, ...)， 可避免复制大型或有状态的函数对象

- `make_pair` 和 `make_tuple` 时更方便的传引用：

  ```c++
  int a=10, b=20, c=30;
  std::pair<int&, int> p1(a b);
  std::tuple<int&, int, int&> t1(a, b, c);
  // 与以下方式对比 (好像写的字还变多了哈哈)
  auto p2 = std::make_pair(std::ref(a), b);
  auto t2 = std::make_tuple(std::ref(a), b, std::ref(c));
  // since C++17, with CTAD:
  std::pair p3(std::ref(m), n); 
  std::tuple t3(std::ref(m), n, std::ref(s));
  ```

  细微差别：`make_pair `和 `make_tuple` 会通过 type trait 识别到 `reference_wrapper<T>` ，并将其 unwrap 为引用`T&`，而 CTAD 构造时则直接操作 `reference_wrapper<T>`。大部份情况无区别。

