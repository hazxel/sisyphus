this document contains C++ basic topics, or those hard to categorize.



# Operator & Expressions

- 逗号运算符：取最右边的表达式的值，其他值被丢弃 (`(a, b)`的值是`b`)，优先级最低
- ...
- 括号：Expressions in parenthesis `()` have the highest precedence. parenthesis preserve value, type, and value category.



# Pointer

### operations

`+` 和`-`对指针进行加减，步长是被指向的元素的大小。 `&` 为取地址(address of operator)，`*` 为解引用(dereference)。

### NULL vs nullptr

`nullptr` is a compile time constant, with type `nullptr_t`, and can be implicitly converted to any pointer type. `nullptr` resolves the ambiguity of `NULL`, because it's hard to tell if `NULL` is pointer or number. **Always** prefer `nullptr` in C++!

```c++
#define NULL ((void*)0) // definition in C
#define NULL 0			// definition in C++
typedef decltype(nullptr) nullptr_t;
```

### void pointer

Can point to any kinds of variable. Some compiler forbid arithmatic (+/-) operation to void pointers. C allows a `void*` pointer to be assigned to any pointer type without a cast, while **C++ does not allow implicit conversion** of `void*`.

### array

- must be init with a brace-enclosed initializer

 ```c++
 int arr[10] = {};
 int mat[][2] = {{1,2}, {3,4}}; // first level size can be omitted
 int arr2[10] = arr;	// won't compile, must use {}
 ```

- `[]` dereference the variable just as `*`

 ```c++
 arr[5] = 0;
 *(arr+5) = 0;		// equivalent
 mat[1][1] = 4; 	// a[n][m] 表示 *(*(a+n)+m)
 ```

- implicitly convert to pointer

 ```c++
 int *ptrInt = arr;		
 int (*ptrMat)[2] = mat;	// only consider the first layer
 ```

- `+` and `-`: step size is `sizeof()` element type 

- `*` and `&`: actual type of pointer and reference

 ```c++
 int (*parr)[10] = &arr;	
 int (&rarr)[10] = arr;
 ```

- 只能在数组定义所在的代码区中获得数组长度(in bytes)

 ```c++
 int size = sizeof(arr);	// 10 * sizeof(int)
 auto foo = [](int[] arr) { 	// won't get the size of the array
   std::cout << sizeof(arr) << std::endl; // sizeof(int*)
 } 	// size of array doesn't follow when passed to function
 ```

- will call ctor and dtor automatically for non-built-in types:

 ```c++
 struct Foo {
   int x;
   Foo() = delete;
   Foo(int) {}
   Foo(int,int) {}
 }
 Foo a[100];		// won't compile, need to call DC() but deleted
 Foo b[2]{1,2}; 	// call DC(int) twice
 Foo c[1]{{1,2}};	// call DC(int,int) once
 ```



# Value

Each C++ expression (an operator with its operands, a literal, a variable name, etc.) is characterized by two independent properties: a **type** and a **value category**. 

### Value categories：lvalue vs rvalue (C++11)

##### Primitive categories

- **lvalue** (non-expiring lvalue): 能够用&取地址的表达式，以及字符串字面值（特例）

  > 具名的参数，包括**具名右值引用**，一定永远是左值。
  >
  > 个人理解：右值只是为了告诉赋值给他的那一方，数据的所有权和生命周期被转交了，但对于接收的那一方，如函数的右值形参，在这个函数执行周期内这个值都是保证不会消亡的，所以其类型实际为 `T&`。

- *prvalue* (pure rvalue): 纯右值，即 C++11前的“右值”，包括但不限于：
  
  - 字符串以外的所有字面值
  - 不具名临时对象如`a+b`, `a++`
  - 返回非引用类型的函数调用，evaluate to 新建对象的表达式，如构造器等
  
- *xvalue* (expiring value): 将亡值，随着右值引用的引入而新引入。将亡值表达式的形式：
  - 返回右值引用的函数的调用表达式 (`move`, `forward`也算)

##### Mixed categories：

- *glvalue* (generalized left value): inlcudes *lvalue* and *xvalue*
- **rvalue** (right value): includes *prvalue* and *xvalue*

### lifetime extension of temporary objects:

不准确但先这么理解：Temporary objects are rvalue. 就是不具名的临时对象

const lvalue reference`const T&`, rvalue reference`T&&`, and storing by named variable (可能指的是基本类型，不然岂不是会调用构造函数?) are 3 ways to extend the lifetime of a temporary object. **For any statement explicitly binding a reference to a temporary, the lifetime of the temporary would be extended to match the life time of reference.**

Why can‘t temporary bind to non-const reference? C++ doesn't want you to accidentally modify temporaries, because they will die soon. But calling a non-const member function on a modifiable (and non basic typed) rvalue is explicit, so it's allowed.

Historical reason: "lifetime extension of temporary objects" is proposed in 1993, before the existance of RVO. So binding of temporary to a reference will save one copy ctor in such circunstance: 

```c++
Foo bar();						// programming under old C++ standard in 1993
Foo f = bar();				// no (N)RVO back then, copy_ctor called
const Foo &f = bar();	// copy free
Foo f = Foo();				// copy_ctor called, no copy elision back then
const Foo &f = Foo();	// copy free
```



# Reference

A Reference is not a object, but an alias ot an existing object or function. Since references are not objects, there are **no arrays** of references, **no pointers** to references, and **no references** to references.

- Reference **cannot be modified to refer to other objects** after init.
- Reference cannot be null, and must be initialized, but 悬垂引用 (dangling reference) is still possible
- There is no const reference (reference is by definition immutable, but reference to a const object is ok)
- reference has type check (safer than pointer)

### reference & rvalue refrence (&&)

个人理解，引用符号和cv-qualifier类似，提供类型信息以外的信息：alias所绑定的对象已经存在。右值引用进一步附加了信息：被绑定的对象是右值。

引用绑定就是给被绑定的对象一个别名：被绑定的对象本身有type和value category；别名的type需要体现被绑定对象的type+value category，但它自己本身的value category一定是lvalue。

### reference collapsing

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

### Universal Reference

C++11 标准引入右值引用，规定右值引用形式的参数只能接收右值，不能接收左值。但对于函数模板和`auto`中使用`T&&`语法定义的参数来说，它既可以接收右值，也可以接收左值（此时的右值引用又被称为“万能引用”）。这一特性可在参数增加时，避免指数级增长的重复函数定义。如果只希望定义右值形参而禁用左值，可？？？

万能引用其实是**模版自动类型推导**以及**引用折叠**一起作用而自然产生的结果：对于使用一个万能引用的函数模版 `template<typename T> void foo(T&&)`，如果需要匹配左值形参 `void foo(Widget&)`，则`T`可被推导为`Widget&`，这样`T&&`即 `Widget& &&` 就会被折叠为 `Widget&`；而对于右值形参 `void foo(Widget&&)`，`T`直接被推导为 `Widget`，`T&&` 则被推导为`Widget&&`。

 ```c++
template<class T>
void wrapper(T&& arg); // this is univeral reference
template<typename T>
void foo(std::vector<T>&& param); // this is not!!!
 ```

### move & forward (C11)

- `std::move` (move semantic, 移动语义) is used to indicate that an object may be "moved from", allowing efficient transfer of resources. 可以不显式指明模版类型`_Tp`，而是编译器查看形参类型利自动推导，以达成 universal reference.

  implementation: remove reference and cast to `Widget&&`

  ```c++
  template<typename _Tp>																		// rvalue: _Tp deduced to T
  typename remove_reference<_Tp>::type&& move(_Tp&& __t) { 	// lvalue: _Tp deduced to T&
    typedef typename remove_reference<_Tp>::type _Up;
    return static_cast<_Up&&>(__t);													// all cast to T&&
  }
  ```

- `std::forward<T>` (perfect forwarding, 完美转发) 虽然右值引用本身可能为左值或者右值，但通过 forward 我们可以强制保证左值引用转换为左值，右值引用转换为右值。forward 必须显式指明模版参数，不可以推导

  ```c++
  template<class T>
  void foo(T&& arg) { arg.DoSomething(); }
  template<class T>
  void bar(T&& arg) { foo(std::forward<T>(arg)); }
  ```
  
  注意理解实现：借助万能引用推导后显式传入的 `_Tp` 的类型来决定返回左值引用还是右值引用，和形参的 value category无关
  
  - 万能引用匹配左值形参时，传入的 `_Tp`是万能引用推导出的 `Widget&`。根据引用折叠规则，返回值也被折叠为 `Widget&`，返回的匿名左值引用一定是左值。
  - 万能引用匹配右值形参时，传入的 `_Tp `是万能引用推导出的 `Widget`。返回值为 `Widget&&`，返回的匿名右值引用一定是右值
  - 直接将具名引用传入`std::forward` 时一定是调用下列第一个左值形参的重载，因为具名左值引用和右值引用都是左值！（forward 函数内的`__t`有名字所以也是左值引用，若`_Tp`为右值引用类型则将其cast为右值引用且返回，相当于move）
  - 如果将具名引用 move 后再传给 forward，就会实例化第二个右值形参的重载。（这个做法在我看来本身就比较奇怪，因为万能引用本身就是为了方便同时接收左值和右值，但这里又forward一个固定为右值的东西）注意 `_Tp` 不能为左值引用，编译器禁止把 rvalue 转发为 lvalue，只能显示把右值复制给左值。
  
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
  

They both do type conversion, implemented using `static_cast`. *Efficient Modern C++* suggests that use `std::move` for rvalue reference conversion and `std:forward` for and **only for universal reference** scenarios.

 ### reference qualifier 引用限定 C11

 ```c++
template <typename T>
class optional {
 // version of value for non-const lvalues
 constexpr T& value() &；
 // version of value for const lvalues
 constexpr T const& value() const&；
 // version of value for non-const rvalues... are you bored yet?
 constexpr T&& value() &&；
 // you sure are by this point
 constexpr T const&& value() const&&；
};
 ```



# Final Exam for argument passing 

### Pass by value - when you need a copy or accepting basic types

```c++
void Func(Data);
Func(Data());								// best, only one ctor called in-place!!!
Data d; Func(d);						// ctor + copy_ctor
Data d; Func(std::move(d));	// ctor + move_ctor
```

### Pass by reference - when modify the param or pass output

```c++
void Func(Data &);
Func(Data());								// won't compile, lvalue ref cannot bind rvalue
Data d; Func(d);						// compiles
Data d; Func(std::move(d));	// won't compile, lvalue ref cannot bind rvalue
```

### Pass by const reference - default choice for read-only params

```c++
void Func(const Data &);
Func(Data());								// compiles
Data d; Func(d);						// compiles
Data d; Func(std::move(d));	// compiles
```

### Pass by rvalue reference - take ownership of the passed param

```c++
void Func(Data &&);
Func(Data());								// compiles
Data d; Func(d);						// won't compile, prevent lvalue accidently passed to it
Data d; Func(std::move(d));	// compiles
```

### Smart Pointers - be careful - see memory chapter

### Return type

- for free functions, usually by value return is the only option, except for returning static/global objects
- for member methods, value, const and non-const reference are all possible
- rvalue reference？？？

### Not recommended

- `void Func(const Data)`: The variable wiil be destroied when out of scope, why can't I modify it?
- `void Func(const Data &&)`: So I took the ownership but still couldn't modify it?
- raw pointers



# const (Keyword)

- const variable: read-only

 > const object cannot call non-const member functions. 
 >
 > member of const object cannot be modified.

- const pointer:

 ```c++
 char greeting[] = "Hello";
 char* p1 = greeting;       // 指针变量，指向字符数组变量
 const char* p2 = greeting;    // 指针变量，指向字符数组常量（const后面是char，说明char只读）
 char* const p3 = greeting;    // 指针常量，指向字符数组变量（const后面是p3，说明p3只读）
 const char* const p4 = greeting; // 指针常量，指向字符数组常量
 ```

- const reference: refrence itself cannot be modified after initialization, so there are only references of const objects, no "const references".

 ```c++
 char c = 'a';
 const int &cr = c; // compiles
 int &ncr = c;    // won't compile
 const int &n = 5;  // compiles
 int &n = 5     // won't compile
 ```

- const member function:
  - such member function cannot modify member variables. (指针指向的对象是例外)
  - const instance can only call const member functions.
  - const member function cannot be static (static functions are independent on instances)

- const return type: useful when returning a reference of class' internal member



# static (keyword)

- static variables
  - normal variables are stored on stack
  - static variables are stored on static segment (scope remains the same), and invisible to other files

- static funcitons: only visible in current file, can be used to avoid conflicts
- static member variables: all instances share one copy, can be accessed without instances 
- static member functions: can be accessed without instances, but cannot use non-static members



# noexcept (operator & specifier)

- as specifier: 声明为`noexcept` 的函数不用处理异常信息，可做编译优化。但仍可能抛出异常，不会生成异常类型信息，程序直接终止
  - declare a function as `noexcept` means it's not allowed to throw exception
  - declare a function as `noexcept(expr)` means it's not allowed to throw exception if expr evaluates to true
- as operator: check if a function is declared to be `noexcept`

```c++
void funA() noexcept; // 函数funA不抛出异常
void (*fp)() noexcept(false); // fp指向的函数允许抛出异常
// 此处第一个noexcept是specifier，第二个是操作符
template <class T, class Alloc>
    void swap(list<T,Alloc>& x, list<T,Alloc>& y)
         noexcept(noexcept(x.swap(y)));
```

> make move ctor noexcept to facilitate STL useages, STL have optimizations for `nothrow_move_constructible` types and compilers can optimize `noexcept` functions
>



# sizeof (compile time operator)

- size of an object of an **empty class** is 1, in order to "ensure that the addresses of two different objects will be different." And the size can be 1 because alignment doesn't matter here, as there is nothing to actually look at.
- Existence of **virtual function(s)** will add 4 bytes of a virtual table pointer in the class. In this case, if the base class of the class already has virtual function(s) either directly or through its base class, then this additional virtual function won't add anything to the size of the class. Virtual table pointer will be common across the class hierarchy.
- Using of **virtual inheritance** will add 4 bytes of a virtual base table pointer in the class. A class will only maintain one virtual base table pointer.
- For classes, only non-static data members will be counted.
- alignment: alignment with the "widest" **basic** type (maybe inside a compound type), and compound types are treated as a whole ({double, char}, char -> 24)
- ordering matters here because of byte padding (char, short, int -> 8 ; char, int, short -> 12)
- char: 1, short: 2, int: 4, double: 8 (depends on GCC version, platform, etc.)



# constexpr (C++11)

Unlike `const`, `constexpr` can also be applied to functions and class constructors. `constexpr` indicates that the value, or return value, is constant and, where possible, is computed at compile time. When a value is computed at compile time instead of run time, it helps your program run faster and use less memory. (low latency!)

when a constexpr function is called with only compile-time arguments, the result of the function will be computed at compile-time. If, however, any argument is notknown at compile-time, the computation will be executed at runtime, like a regular function.

##### constexpr if (C++11?)

???

##### constexpr new(C++20)

???



# using

- using-directives for namespaces: `using namespace namespace-name;`

  Allowed only in namespace scope and in block scope. Every name from `namespace-name` is visible until the end of the scope in which it appears. 

  > **Don‘t do this in header files!** Including your header will introduces namespace to global namespace, may lead to conflicts! Use *using-declarations* to introduce single member instead.

- using-declarations: `using ns::member;`

  - In namespace and block scope: introduce a member of another namespace into the region
  - In class definition: introduces a member of a base class into the derived class definition
    - introduce base class constructors: `using BaseClassName::BaseClassName;`: For every constructor of BaseClass, compiler generates an identical constructor for derived class.
    - Introduce other functions: `using BaseClassName::FuncName;` for every overload of the function name, generates an identical function for derived class;
  - for enumerators (C++20) `using enum enum-class-name;`
  
- type alias: refer to types chapter




# Namespace

Namespaces provide a method for preventing name conflicts in large projects.

#### operator ::

- `::name` global namespace

- `class-name::member-name` scope is some class

  > can be used to call super class' virtual functions!

- `namespace-name::member-name` scope is some namespace

- Default: nearest scope



# Range-based for loop (since C++11)

```c++
for (auto declaration : expression) {
	// loop-statement
}
// is equivalant to:
auto && __range = range-expression;
auto __begin = begin-expr;
auto __end = end-expr;
for ( ; __begin != __end; ++__begin)
{
  range-declaration = *__begin;
  //loop-statement
}
```

- Choose `auto x` when you want to work with copies.
- Choose `auto &x` when you want to work with original items and may modify them.
- Choose `auto const &x` when you want to work with original items and will not modify them.



# Initialization

### Basic Syntax

```c++
T declarator();								// direct init
T declarator{};								// list init
T declarator = <expression>;	// copy init
T declarator = {};						// possible
```

- `{}` doesn't allow narrowing conversion (good thing), also called uniform initialization
- `()` can't use no-param-ctor (*most vexing parse* : `Widget w();` initialization or function declaration?)
- Non-copyable ojbects cannot use `=` to initialize

### List Initialization (C++11)

##### Aggregate Initialization

A special case of List Initialization. 使用花括号初始值列表初始化聚合类(Aggregate)的数据成员，初始值的顺序必须与声明的顺序一致，若初始值列表的元素个数少于类的成员数量，则靠后的成员被值初始化。聚合类型定义为：

- 普通数组，如int[5]，char[]

- 满足下列所有条件的 `class` , `struct` 或 `union`：

 - 没有用户提供的构造函数 (允许 `default` 或 `delete`)

 - 没有私有或保护的非静态数据成员

 - 没有虚函数
 - 如果有继承关系，必须是公开、非虚继承 (C++17)

注意基类是否是聚合类型与派生类是否为聚合类型没有关系。

标准库 `<type_traits>` 中提供了 `is_aggregate`，可帮助判断目标类型是否为聚合类型。

> GCC 支持小括号初始化聚合类型，允许窄化转换，其他编译器不支持

##### For non-aggregate types

大括号初始化大部分时候是优于小括号初始化的。它能被用于各种不同的上下文，防止隐式窄化。但当对象拥有接收`std::initializer_list`的构造函数时，大括号初始化的语义比较混乱：

Example: Changing braces to parentheses in initialization may change semantics：

```c++
std::vector<int> v1{5, 6}; // 2 elems: {5, 6}
std::vector<int> v2(5, 6); // 5 elems: {6, 6, 6, 6, 6}
```

若没有包含`std::initializer_list`形参的构造函数，花括号初始化和圆括号初始化效果完全相同。编译器看到 `{t1, t2, …, tn}`便会构造一个 `initializer_list`，它关联到一个 `array<T, n>`，该array内的元素会被编译器分解逐一传给构造函数。

若有一个或多个构造函数包含一个如`std::initializer_list<T>`的形参时：

- 花括号初始化优先尝试将实际参数转换为一个`std::initializer_list<P>`:
  - 如果`T`不能转换为 `P` ，编译器才会考虑其他构造函数
  - 如果`T`是基本类型且可以转换为 `P` ，即使存在类型匹配更准确的非`std::initializer_list`形参构造函数，也会被忽略
    - 如果`T`需要窄化才可转换为`P`，编译器就会报错
    - 如果`T`不需窄化就可转换为`P`， 编译器就会选择该函数
- 想传入一个空的 `std::initializer_list`, 需要这么写：`Widget w({}); `
- 在模板中使用花括号构造对象时情况会更麻烦，因为模版类通常不知道关于模版参数类是否有包含一个`std::initializer_list`形参的构造器。(e.g. `std::make_unique`, STL 的解决方案是使用圆括号，并记录至接口文档中)

##### STL initializer_list

`std::initializer_list` is an array of `const T` objects. It will be automatically constructed when:

- a *brace-init-list* is the right operand of `=` 
- Pass a *brace-init-list* to a function, and the function accepts `std::initializer_list` parameter
- Use a *brace-init-list* to construct an object and exists a ctor accepts `std::initializer_list` （窄化转换会编译失败）
- a *brace-init-list* is bound to auto

所有的标准容器的构造函数都有以initializer_list为参数的构造函数。initizlizer_list的最广泛的使用就是不定长度同类型参数的情况。

##### Designated Initializers(C++20)

？？？



# static_assert

`static_assert` requires a compile-time predicate, and a message is displayed when the compile-time predicate fails. With C++17, the message is optional. With C++20, this compile-time predicates can be a requires expression.
