# Template

模板能做的事情都是编译期完成的，C++ Template最初是为了提供一个类型安全、易于调试的宏

##### typename vs class

模板参数除了类型外，也可以是一个整型数。这里的整型数比较宽泛，包括布尔型，不同位数、有无符号的整型，甚至包括指针。在模板定义语法中关键字 `class` 与 `typename` 的作用完全一样。其他场景下，`typename` 用于提示后面的字符串为一个类型名称(有时编译器不能确定这种东西`Widget<T>::type` 是类型名还是变量名)

##### typedef vs using

Almost the same, except that `using` is compatible with templates, whereas the C style `typedef` is not:

```c++
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;
MyAllocList<Widget> lw;

template<typename T>        
struct MyAllocList {   
    typedef std::list<T, MyAlloc<T>> type;     
};
MyAllocList<Widget>::type lw;        
```



# Type Deduction

> trick to get type duction at compile time - pass a type to an unimplemented template struct: `template<typename T> struct TD;` and get deducted type from compiler error message.

### Template deduction

对函数模版`template<T> void f(ParamType param)`，编译器使用`f(expr);`中的`expr`进行针对`T`的和对`ParamType`的类型推导。这两个类型通常是不同的，因为`ParamType`包含一些修饰，比如`const`, `&`, `*`等。

- 不加限定符号时，也即`ParamType=T` 时，为传值类型推导，本质上是拷贝，所以`const`和`volatile`实参修饰会被忽略
- 加 `&`modifier时为传引用推导，加`&&` modifier 时为万能引用

> 特殊情况：**数组名**或者**函数名**实参：
>
> - 在 `ParamType=T`（无限定符号） 时 `T` 推导为 `Widget*`
> - 在 `ParamType=T&` （引用限定）时 `T` 推导为`Widget&`


### `auto` deduction

`auto` 不处理形参，但其推导规则和 template deduction 几乎一致。`auto` 前后也可以加修饰，`auto` 本身相当于`T`， 修饰后的整体类型 `ParamType`类似于函数的形参。(e.g. universal reference: `auto&& u = lvalue;`)类似的，不加修饰会被推导为传值，可能会产生拷贝等，需要注意。

对于花括号的处理是`auto`类型推导和 template deduction 唯一不同的地方：`auto` 类型推导时总是会把花括号理解为`std::initializer_list`，而 template deduction 接收花括号形参时会报错。

`auto`可出现在函数返回值或者 *lambda* 形参(C++14)中，但是它的工作机制遵循是模板类型推导规则，而不是`auto`类型推导。

### `decltype` deduction

`decltype` tells the type of a name or expression. 主要用于声明返回类型依赖于形参类型的模版函数。

- for entity: *entity* refer to id-expression and member access (unparenthesized). yield the **declared type** of the entity, regardless of **value category**.
- for expression: 
  - prvalue expression yield `T`
  - lvalue expression yield `T&`
  - xvalue expression yield `T&&`
- `decltype(auto)`  is equivalant to `decltype(expr)`, where `expr` is initializer or ones in return statement. `decltype(auto)` 不可再被其他modifier修饰

```c++
struct A { double x; };
const A* a;
decltype(a->x) y;    		// type: double (declared type), rule for entity
decltype((a->x)) z = y; // type: const double& (lvalue expr), rule for expr

template<typename Container, typename Idx>	// c++11
auto access(Container& c, Idx i)	// auto 标示返回值由类型自动推导而来
	-> decltype(c[i]) { 						// decltype 计算返回类型
    return c[i];				
}
template<typename Container, typename Idx>	// c++14  
decltype(auto) access(Container& c, Idx i) {                                           
    return c[i];
}
```

注意：`decltype` 推导非表达式时，如果需要返回非引用类型的对象的引用，可以在外层加上`()`, 使其成为一个表达式且 value category 不变，即 lvalue expr，可被推导为 `T&`。

### Class Template Argument Deduction (CTAD) (C++17)

C++17 前，实例化类模板时必须显式指定模板参数。

推导指引: `MyClass(const char*) -> MyClass<std::string>;`

CTAD 并不总是能自动推导所有情况，当模板参数不能从构造函数参数中明确推导时，仍需显式指定模板参数。

todo



# Function Template

### Function template specialization

```c++
template<typename T> void f(T s) { std::cout << s << '\n'; } // primary template
template void f<double>(double); // instantiates f<double>(double)
template void f<>(char);         // instantiates f<char>(char), template argument deduced
template void f(int);            // instantiates f<int>(int), template argument deduced
void f(bool);										 // overload 
```

> **Don’t specialize function templates. (C++ Core Guidelines: [T.144](https://link.zhihu.com/?target=http%3A//isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines%23Rt-specialize-function): )**
>
> You can fully specialize a function template but you almost certainly want to overload instead – because function template specializations don’t **participate in overloading,** they don’t act as you probably wanted. 

> **You CAN'T partially specialize function templates.**
>
> A major concern over *partial specialization of function templates* is that function templates can be **overloaded**, thus the function template name is not adequate to identify the template being specialized.

### function template overloading 

A function template and a non-template function may be overloaded, while 2 *non-equivalent* function templates may also be overloaded. 

##### partial ordering of overloaded function templates 

> 首先明白，模版特化需要出现在模版原型的声明之后，否则编译器不会认为这是该原型的特化

When a function template specialization matches more than one overloaded function templates, *partial ordering of overloaded function templates* is performed to select the more **specialized** template.

Informally "A is more **specialized** than B" means "A accepts fewer types than B". Examples:

1. A template function expecting a specific type is more specialized than one taking a generic type.
2. A template taking `T*` is more specialized than one taking `T`, because  `T*` is a valid argument for a `T` template argument, but `T` isn't a valid argument for a `T*` template argument.
3. `const T`(or `volatile`) is more specialized than `T`, because `const T` is a valid argument for a `T` template argument, but `T` isn't a valid argument for a `const T` template argument.



# Class Template

### Class template specialization

???

##### Partial Specialization

`template <parame-list> class class-name <arg-list> declaration` the `<arg-list>` indicates it's a partial specialization, one without which is the primary template.





# Argument-dependent lookup (ADL)
???



# variadic templates (C++11)

```c++
template <class... T>
void func(T... args) // args is a template parameter pack
{ Foo(args...); }
```

### pack expansion

??? https://zhuanlan.zhihu.com/p/670867561

### Fold Expressions (C++17)

可以简化对参数包的处理：???似乎是直接就地展开 ??? 和上一个有啥区别？？？

```c++
template<typename T>
string format(T t) {
    std::stringstream ss;
    ss << "[" << t << "]";
    return ss.str();
}

template<typename... Args>
void FormatPrint(Args... args)
{
    (std::cout << ... << format(args)) << std::endl;
}
```

### 继承 variadic template

如下面这个例子，多继承所有模版参数类，并显式启用所有functor。这种场景一般继承的是 lambda（variant 的 visit），如果出现基本类型会无法通过编译。

```c++
template<class... Ts>
struct overloads : Ts... { using Ts::operator()...; };
```





# Variable template (C++14)

A variable template defines a family of variables or static data members.

```c++
template<class T>
constexpr T pi = T(3.1415926535897932385L); // variable template

template<class T>
T circular_area(T r) { // function template
    return pi<T> * r * r; // pi<T> is a variable template instantiation
}
```

In class scope, only declares **STATIC** data members. Initialization rules same as static data members.

```c++
struct limits {
    template<typename T>
    static const T min;
}; // static 成员需要类外初始化
template<typename T> const T limits::min = { };

// 这个不是 variable template
template<class T>
class X {
    static T s; 
}; // static 成员需要类外初始化
template<class T> T X<T>::s = 0;
```





# (C++20) Concept & Requires

specify what's expected of template arguments, and get a nice clear compiler error message if misused

### Usage with integer template parameter

```c++
template <unsigned int i>
requires (i <= 20)         
int sum(int j) { return i + j; }
```

### Define a Concept

```c++
template<typename T>
concept Addable = requires (T a, T b) { a + b; }; // 检查 a 和 b 可加

template<typename T>
concept Printable = requires(T a) { 
    // 检查 T 是否有一个名为print的成员函数，该函数接受一个类型为 std::ostream& 的参数，且返回类型为void
    { a.print(std::cout) } -> std::same_as<void>;
  	// 可以有多个条件，可以如下这样做编译期检查
  	requires(std::is_same<T::type, std::int32_t>::value);
};
```

### Require a concept (use concept to validate template param)

```c++
template<typename T>  
requires Addable<T>
auto add(T a, T b) { return a + b; }
// or
template<Printable T> 
void print(const T& t) { t.print(std::cout); }

template<typename T>                                        
requires std::integral<T> // use std concepts whenever possible                
auto gcd(T a, T b) { //... }
```

### Requires anonymous concept

You can define an anonymous concept and directly use it but it should be avoided. Anonymous concepts are less readable and not reuseable.

```c++
template<typename T>
    requires requires (T x) { x + x; } 
auto add(T a, T b) { return a + b; }
```

### logic behind `concept`

```c++
template<typename T> requires std::integral<T>  some_func(const T &a, const T &b) {};
template<class T> concept integral = std::is_integral_v<T>;
template<class _Tp> struct is_integral : _BoolConstant<__is_integral(_Tp)> {};
template<bool _Val> using _BoolConstant _LIBCPP_NODEBUG = integral_constant<bool, _Val>;
```

### STL built-in concepts

- `same_as`, `convertible_to`: type requirements , evaluate to bool.
- 



# consteval (C++20)

函数可以用 `consteval` 限定只在编译期调用





# Substitution failure is not an error (SFINAE)

This rule applies during overload resolution of function templates: When substituting the explicitly specified or deduced type for the template parameter fails, the specialization is discarded from the overload set instead of causing a compile error.

当创建一个重载函数的候选集时，某些（或全部）候选函数是用模板实参替换模板形参的模板实例化结果。如果失败，编译器将在候选集中删除该模板，而不是当作一个编译错误从而中断编译过程。如果一个或多个候选保留下来，那么函数重载的解析就是成功的。

