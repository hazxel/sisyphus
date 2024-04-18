# Reference

A Reference is not a object, but an alias ot an existing object or function. Since references are not objects, there are **no arrays** of references, **no pointers** to references, and **no references** to references.

- Reference **cannot be modified to refer to other objects** after init.
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

> (Effective Modern C++, item 26/27) 完美转发的转换构造函数是糟糕的实现，因为他们会比默认拷贝构造函数更加匹配，劫持拷贝和移动构造的调用。(e.g. `template<T> Widget(T&&)` 会覆盖 `Widget(const T&)`)



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

